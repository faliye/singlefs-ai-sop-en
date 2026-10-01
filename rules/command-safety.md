<!-- generated-from: rules/command-safety.md sha256:395ed99b83ed5ebcf18f954cbf516d60b9beb1f598d96b2e7cd6357996941b45 -->
<!-- doc-lint:rule-definition -->
# Process and command discipline

Seven of these are failing checks, judged by `scripts/shell-lint.sh`: **killing processes
by pattern match** (`pkill -f`, `killall`), **`pgrep -f`**, **carrying a value out of a
subshell through a variable**, **git's undo commands inside a script**, **`rm -rf` on
an unguarded variable path**, **a `wait` with no arguments**, and **a pipeline ending in `grep -q` in a script that sets `pipefail`**. Whether a test harness deleted the images it created when it finished is judged by `gate.sh`'s "No temporary files left behind" (`跑完没留下临时文件`). The rest are still prose, because the criterion for
checking them mechanically is not worked out yet (`show-me-test.md`, "what the gate
can and cannot prove").

## Before anything you cannot take back, think once more

The criterion: **after this step, can you get back?**

| Kind | Examples | What to do |
|---|---|---|
| **Throws away uncommitted work** | `git checkout <file>`, `git restore`, `git reset --hard`, `git clean` | `git stash` or `cp` a copy out first; use them only when you mean to discard *all* uncommitted changes to that file |
| **Deletes data outright** | `rm -rf`, `> file`, `sed -i`, `rsync --delete`, `mkfs`, `dd` | Look at the target first (`ls` / `git status` / `lsblk`); guard variable paths with `${VAR:?}` |

**Run throwaway experiments on a copy**, not in the working tree.

**Scripts must never contain git's undo commands.** To get to a clean state inside
a script, `git stash` first, or copy the whole repository and work on the copy.

**`rm -rf` on a variable path needs an empty-value guard**: write `rm -rf "${d:?}/x"`, not
`rm -rf "$d/x"`. `rm -rf "$d"` with nothing after it needs no guard.

## `pkill -f` / `killall` are forbidden outright

To stop a process: `ps` first, look at it, then kill by **literal pid** in a separate
second command; or hand it to `scripts/proc.py stop <pid>`: it sends TERM, sends KILL if the process is still there
after `--grace` seconds (default 10), and refuses when the target is the process issuing the command or one of its ancestors.
For counting, use structured criteria from `/proc` and exclude your
own process tree.

**Do not use `pgrep -f` in a wait loop either** (a wait like `until ! pgrep -f "X"; do sleep 5; done`).
To wait for a process to end, use its **literal pid**: `until ! kill -0 "$pid" 2>/dev/null; do sleep 5; done`;
for a background job you started yourself, use `wait`; for one you did not start (a daemon another script launched, say), have whoever starts it write its pid to a file and wait on that.
If the pid was not recorded, find it with `scripts/proc.py find <executable-name> [--argument <argument>]`: it compares the executable name exactly instead of matching a pattern against the whole command line, and it does not list the process tree that issues the command;
with the pid in hand, `scripts/proc.py wait <pid> --timeout <seconds>` waits for it to exit, and if it has not exited when time is up, lists the ones still alive and exits with code 3.
The S3 check in `scripts/shell-lint.sh` turns every `pgrep -f` in command position in a script red (after `if` /
`while` / `until` / `!` also counts as command position), without looking at whether the same line also has a `kill`.
Commands typed by hand are refused before they run by `scripts/claude-hooks/pattern-process-guard.sh`, which uses the same criteria as shell-lint's S2 and S3 (`PATTERN_KILL_RE` and `PATTERN_PGREP_RE` in `scripts/lib.sh`),
and when it refuses it gives the pid-based ways to do the same thing. It is a Claude Code PreToolUse hook; a project registers it once under `hooks.PreToolUse` in `.claude/settings.json`:

```json
{"matcher": "Bash", "hooks": [{"type": "command", "command": "bash \"$CLAUDE_PROJECT_DIR\"/.claude/singlefs-ai-sop/scripts/claude-hooks/pattern-process-guard.sh"}]}
```

In a session where it is not registered, commands typed by hand still rely on this text alone.

## Once you start a background task or a subagent, re-check it on a schedule, and do not force it to end

Long work may run long, but someone has to be watching it:

- **Anything started gets re-checked.** For a task started in the background or a subagent sent off, whoever dispatched it checks on a schedule: how long since its last action,
  whether it is sitting in a wait loop, whether the same command keeps running with byte-identical output, whether the files it writes are still growing.
- **Keep detection separate from handling.** Tools that detect stuck or spinning work (hooks, watchdog scripts) only record and report; they do not block commands, kill processes, or stop subagents.
  Whether it ends and how it is handled is decided by whoever dispatched it, after looking.
  A hook that refuses one specific dangerous form is not covered here (`session-wrapup.md` item 4: refuse, before writing, to overwrite an untracked file).
- **Before waiting for a line in a log, make sure that line is really written to that file.**
- In sessions such as Claude Code, put long work in the background and come back to re-check it periodically, instead of idling in the foreground.

## Within one script, run the checks in parallel when they can be

**Criterion: this batch of checks do not depend on each other, and every one of them has to start a
subprocess to do its work (a build, a test run, a VM, a python process). Then run them in parallel,
instead of waiting for them one at a time.**

Once it is fast, look again at where the remaining time goes: a case that idles on purpose (one that verifies, say,
that `wait` does not return once the timeout logic is broken) is not saved by parallelism.

**These three do not go parallel**:

- a later item reads an earlier item's product;
- they share one writable state (the same temp directory, the same target directory, the same device). To go parallel, give each item its own;
- each item is fast by itself, running only a few milliseconds.

Take the degree of parallelism (processes, and threads within a process) from an argument or an environment variable, falling back to `nproc`; do not hard-code it; heavy work such as VMs and builds gets
its own ceiling from memory and devices.
The cargo commands the shared scripts start (`scripts/check.sh`) run through the prefix registered in `.claude/cargo-command-prefix` at the project root:
one line `<command and arguments>  # reason`, split on whitespace with no quoting, relative paths taken from the project root, each cargo command going through it once.
If the file exists but yields no usable prefix, that is red; it never falls back to running without the wrapper:
words that look like paths (containing `/`, or ending in `.sh` or `.py`; options starting with `-` and assignments containing `=` do not count) must exist, and a dry run of the prefix (`<prefix> true`) must succeed; under `gate.sh --staged`, a registration present in the source repository but missing from HEAD plus the index is red as well.

## Parallelism must not swallow the failures

There are only two ways to collect: record `pids+=($!)` when starting and take the exit code with `wait "$pid"`
one at a time; or have each parallel item write its own exit-code file, and judge them when collecting in the
order of the table the work was dealt from.
A `wait` with no arguments, and `|| bad=1` inside the background body with the parent reading `$bad`, collect no exit code.

S6 in `scripts/shell-lint.sh` judges this one: a `wait` with no arguments in command position turns red.
Where the exit code really is collected elsewhere, write `# shell-lint:exit-collected <how it is collected>`
on that line, and the reason may not be left out.

**No parallel item writes its output straight to stdout.** Each item writes its own file, and collection reads
them back in the order the work was handed out.

**However many items were handed out, that many have to come back.** Count them when collecting, and turn the
whole thing red when the count does not match what was handed out.

**After making something parallel, prove again that it can go red.** Do what `show-me-test.md` says: feed it an
input that must go red, and see whether the parallel version still goes red.

## Do not use echo to fake success

Do not report success the way `cmd 2>/dev/null; echo "done"` does.
Any command that changes state must be verified by **reading the state back**: after
`systemctl stop X`, confirm with `is-active`; the exit code of `stop` is not enough.

## Result collection needs a completeness gate

**Do this**: have the program under test report, on its final line, how many results it
emitted; the collector compares the count and discards the round on a mismatch.
That gate must itself be proven to go red first — feed it a fake program that claims
N results and emits N−1, and it must fail.

## A script with a gate hands over its output only after judging it

When one script both produces a result and decides whether that result is usable, **the output goes to the caller only
after the verdict**.

**What to do**: write the result to a temporary file first and output it only once the gate passes; on red, stdout stays
empty. This gate must be shown to go red as well: change the script back to "output first, judge later", and the
self-test must go red.

## Assignments inside a subshell do not travel back to the parent

In `rc="$(run_one ...)"`, any variable `run_one` assigns is **empty in the parent**.
**Pass values through a file or an argument, not through a variable.** It is a check that goes red in `scripts/shell-lint.sh`.

## After a script edits a file, read it back — a compiler warning is a free signal

When a script does a string replacement on code or docs, two things to do:

1. **The replacement must assert it hit something**: zero replacements is an error to
   raise, not "ran the script, so it's done".
2. **Read back after editing**: grep for the new content, or just run it and see
   whether the behavior changed.

**Never wave off a compiler or linter warning.**

## Three silent failures at a process boundary

| Form | How to write it |
|---|---|
| `inner; rc=$?` under `set -e` | Take the exit code inside an `if`: `if inner; then rc=0; else rc=$?; fi` |
| `export X="$(cmd)"` / `local x="$(cmd)"` | Assign first, `export` second — write it as two lines |
| An environment variable used for a handshake leaking into a child process | `unset` it as soon as it is read; clear it with `env -u` when starting a child process |

**The criterion is "how many process levels does this value cross"**: cross one and you
have to ask whether it is still there at the next level, and whether it should be.

## The exit code of a pipeline is not the one you want

`$?` after `cmd | head`, and `cmd | grep x && do_something`, both judge the last stage of the pipeline, not `cmd`.

**What to do**: to judge whether the earlier stage succeeded, use `${PIPESTATUS[0]}`,
or skip the pipe altogether — capture the output into a variable or file first, then
check the exit code before doing anything with it.

S7 in `scripts/shell-lint.sh` judges one form of this: in a script that sets `pipefail`, a pipeline ending in `grep -q` (`… | grep -q pattern`) turns red; a match gets read as no match.
The fix: capture the earlier stage's output into a variable first, then `grep -q pattern <<<"$variable"`; that form, with no earlier stage, is not judged.

## Test images always go in a temporary directory

Never inside the repository. Image paths come from an environment variable, defaulting
to `${TMPDIR:-/tmp}`.

**The harness that creates an image deletes it itself when the test ends**: on success, on failure and on panic alike. Rust uses a Drop guard; shell uses `trap … EXIT`.
Keeping it to look at the scene is switched on by an explicit switch (an environment variable), and the path is printed when it is kept; without the switch it is deleted.
A gate stage that goes red may leave its scene in `$TMPDIR` and name the path in its remedy.
A cache meant to be reused across runs (build output, a verdict store and the like) is not an image: put it under `${GATE_CROSS_RUN_TMPDIR:-${TMPDIR:-/tmp}}` or a place the project sets, not in `$TMPDIR`.

`gate.sh` checks this: each run gives the stages a `TMPDIR` of that run's own, and at the end looks at what is left in it (the stage "No temporary files left behind", `跑完没留下临时文件`):
if every other stage is green, whatever is left goes red, listed by name and size, and is deleted on exit; if some stage went red, this item is recorded as not judged this run, and the run's temporary directory is kept whole with its path printed — delete it yourself once you have looked.
Of the directories kept this way on a red run, only the latest 3 are kept; older ones are deleted by a later run.
What it cannot see: harnesses that hard-code `/tmp` instead of going through `TMPDIR`, tests run by hand outside the gate, and anything created by child processes still running after the gate was interrupted.

## Look before running anything destructive

For `mkfs` / `dd` / `dmsetup remove`, **the target device must come from a variable, and `lsblk` must print it for
confirmation first**; never hard-code a literal `/dev/sdX`.
