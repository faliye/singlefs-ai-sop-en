<!-- generated-from: rules/command-safety.md sha256:5afa10eedc88d2f4ba0ac404c1c7c83d17a0d63295205c0aec710e172e169db0 -->
<!-- doc-lint:rule-definition -->
# Process and command discipline

Eight of these are failing checks, judged by `scripts/shell-lint.sh`: **killing processes
by pattern match** (`pkill -f`, `killall`), **`pgrep -f`**, **carrying a value out of a
subshell through a variable**, **git's undo commands inside a script**, **`rm -rf` on
an unguarded variable path**, **a `wait` with no arguments**, **a pipeline ending in `grep -q` in a script that sets `pipefail`**, and **judging whether a file under `/proc`, `/sys` or cgroupfs is empty by its size**. Whether a test harness deleted the images it created when it finished is judged by `gate.sh`'s "No temporary files left behind" (`跑完没留下临时文件`). Creating an unregistered file or directory at the project root is judged by the hook `scripts/claude-hooks/root-entry-guard.sh` and by `gate.sh`'s "New project-root entries" (`项目根的新条目`). An unregistered symbolic link in the repository is judged by `gate.sh`'s "Soft links" (`软链接`). The rest are still prose, because the criterion for
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

**`rm -rf` on a variable path needs an empty-value guard**: write `rm -rf "${build_directory:?}/cache"`, not
`rm -rf "$build_directory/cache"`. `rm -rf "$build_directory"` with nothing after it needs no guard.

## Do not create files or directories at the project root

Do not create a file or directory at the project root (the top level of the repository). Put new things into a directory that already exists: drafts and logs go in `$TMPDIR` or the session's scratch directory; scripts, records and outputs go in the directories the project has set for them.
Recreating something after deleting it counts as creating it. Entries ignored by `.gitignore` count too. The entries this package itself needs at the root — `.git`, `.claude`, `CLAUDE.md`, `.singlefs-ai-sop-version` — need no registration.

When something really has to be created at the root:

1. Ask the user first: say the name, what goes in it, and why it cannot go into an existing directory.
2. Once the user agrees, register one line `<name>  # reason` in `.claude/root-entries` at the project root: the name is the single top-level component, with no other `/`; the reason says what goes in it and why it sits at the root, at least 4 characters; one line per name.
3. Then create it. Commit the registry and the new entry in the same commit.
4. When the entry is deleted, delete its line from the registry in the same change.

Something a tool rebuilds every time (a build directory and the like) is registered once.

| Who judges | What |
|---|---|
| Hook `scripts/claude-hooks/root-entry-guard.sh` | Write, Edit or NotebookEdit writing into a root entry that does not exist yet and is not registered: refused before it runs. Changing the registry (with those tools, or a Bash command that writes to it): the user has to confirm in the UI. After a Bash command, new unregistered entries at the root: reported to the agent, which must move them or ask |
| Gate stage "New project-root entries" (`项目根的新条目`, `scripts/root-entries.py`) | Every root entry added relative to the diff base (tracked, and untracked but not ignored) must be registered. A registry line with no reason, too short a reason, a name with more than one level, a duplicate, or a registered entry that does not exist and is not covered by `.gitignore`: red |

Register the hook in `.claude/settings.json`; how is in its file header.

What it cannot judge: whether the user agreed and whether the reason holds — review. Ignored entries are not judged by the gate; only the hook sees them. Something a Bash command creates is reported only after the command has run. A Bash write to the registry whose path is built in a variable or written from an interpreter is not recognised and asks for no confirmation.

## No symbolic links in the repository

The repository (from the project root down, `.git` excepted) holds no symbolic links: tracked, untracked and `.gitignore`-ignored ones all count. To use a file from elsewhere, copy it in, or write its real path where it is used.
This package's repositories and every project that installs the package follow this, each registering under its own project root.

A symbolic link that really has to stay:

1. Register one line `<path relative to the project root>  # reason` in `.claude/soft-link-exempt` at the project root: no leading `/`, no `.` or `..`; the reason says what it points to and why it cannot be replaced by a copy or a written-out path, at least 4 characters; one line per path.
2. When the link is deleted or replaced by a regular file, delete its line in the same change.

The gate stage "Soft links" (`软链接`, `scripts/soft-links.py`) judges: symbolic links in the working tree, and index entries with mode `120000`, that are not registered are listed path by path and judged red; under `gate.sh --staged` the source repository's working tree is judged too.
An exempt-table line with no reason, too short a reason, a path not relative to the project root, a duplicate, or a registered path that is not a symbolic link: red.

What it cannot judge: whether the reason holds — review. Anything inside `.git` is not judged.

## `pkill -f` / `killall` are forbidden outright

To stop a process: `ps` first, look at it, then kill by **literal pid** in a separate
second command; or hand it to `scripts/proc.py stop <pid>`: it sends TERM, sends KILL if the process is still there
after `--grace` seconds (default 10), and refuses when the target is the process issuing the command or one of its ancestors.
For counting, use structured criteria from `/proc` and exclude your
own process tree.

**Do not use `pgrep -f` in a wait loop either** (a wait like `until ! pgrep -f "<process name>"; do sleep 5; done`).
To wait for a process to end, use its **literal pid**: `until ! kill -0 "$pid" 2>/dev/null; do sleep 5; done`;
for a background job you started yourself, use `wait`; for one you did not start (a daemon another script launched, say), have whoever starts it write its pid to a file and wait on that.
If the pid was not recorded, find it with `scripts/proc.py find <executable-name> [--argument <argument>]`: it compares the executable name exactly instead of matching a pattern against the whole command line, and it does not list the process tree that issues the command;
with the pid in hand, `scripts/proc.py wait <pid> --timeout <seconds>` waits for it to exit, and if it has not exited when time is up, lists the ones still alive and exits with code 3.
The S3 check in `scripts/shell-lint.sh` turns every `pgrep -f` in command position in a script red (after `if` /
`while` / `until` / `!` / `coproc` also counts as command position), without looking at whether the same line also has a `kill`.
Commands typed by hand are refused before they run by `scripts/claude-hooks/pattern-process-guard.sh`, which uses the same criteria as shell-lint's S2 (killing processes by pattern match) and S3 (`pgrep -f`) (`PATTERN_KILL_RE` and `PATTERN_PGREP_RE` in `scripts/lib.sh`),
and when it refuses it gives the pid-based ways to do the same thing. It is a Claude Code PreToolUse hook; a project registers it once under `hooks.PreToolUse` in `.claude/settings.json`:

```json
{"matcher": "Bash", "hooks": [{"type": "command", "command": "bash \"$CLAUDE_PROJECT_DIR\"/.claude/singlefs-ai-sop/scripts/claude-hooks/pattern-process-guard.sh"}]}
```

In a session where it is not registered, commands typed by hand still rely on this text alone.

## Sample processes a harness starts go in a process group of their own

When a harness, a judging script or a replay script starts sample processes (a real shell, the program under test, a sample command), the criterion is: **when each sample finishes, not one of the processes it started is left.**

- Put each sample in a process group of its own: in shell start it with `setsid`; in Python pass `start_new_session=True` to `subprocess`.
- On timeout and at the end, signal the whole group, TERM first and KILL after a grace period: in Python `os.killpg(<process-group-id>, <signal>)`, in shell `kill -TERM -- -<process-group-id>`. Do not stop only the process you started directly.
- After the run, check `/proc` by process group id that nothing is left in the group; stop what is left and write it into that sample's failure message.
- Stopping only the direct child leaves the grandchildren behind: a `cat` started by a process substitution holds the write end of the pipe, and whoever reads that pipe never gets EOF.

This is not a check yet: whether what a script starts is a sample, and whether it stopped the whole group on timeout and at the end, cannot be told reliably from the script; it rests on review.

## Once you start a background task or a subagent, re-check it on a schedule, and end it only when whoever dispatched it does not respond

Long work may run long, but someone has to be watching it:

- **Anything started gets re-checked.** For a task started in the background or a subagent sent off, whoever dispatched it checks on a schedule: how long since its last action,
  whether it is sitting in a wait loop, whether the same command keeps running with byte-identical output, whether the files it writes are still growing.
- **Keep detection separate from handling.** Tools that detect stuck or spinning work (hooks, watchdog scripts) only record and report and do not block commands; they do not kill processes or stop subagents, except when whoever dispatched the work does not respond.
  Whether it ends and how it is handled is decided by whoever dispatched it, after looking; if whoever dispatched it does not respond after the report (the same thing reported several times in a row, neither handled nor acknowledged), whoever reported (a person, an agent, or the detecting tool) may end it: a process with `scripts/proc.py stop <process id>`, a subagent with the tool that stops it; an ending is reported.
  A hook that refuses one specific dangerous form is not covered here (`session-wrapup.md` item 4 "Is another session in flight in the same repository?": refuse, before writing, to overwrite an untracked file).
- **After stopping a sub-agent, settle its changes and its unfinished work on the spot.** Decide file by file for whatever it left half-changed: keep it, revert it, or hand it to someone to continue; decide who takes over the work it did not finish.
  Write the decision into the record and tell whoever takes over; leave no half-made change without an owner.
- **Name background tasks and script tasks in the language used with the user, saying what each is and which job it is waiting for.** Tasks running at the same time may not share a name; one whose name alone does not say what it is doing is renamed before it is started.
- **Before waiting for a line in a log, make sure that line is really written to that file.**
- In sessions such as Claude Code, put long work in the background and come back to re-check it periodically, instead of idling in the foreground.

## When you hit a problem, first find out whether another agent owns that part

This covers a problem met mid-work that your own task cannot solve or that is not your task's to solve: the gate is red on a file you did not change, a file you need to change is being changed by another session, something you need is produced by another task, or you cannot fix it or are unsure how.

Before fixing it, working around it, or stopping to wait, do these in order:

1. **First find out who owns it.** List the agents and sessions that are running (in Claude Code, `ListAgents`), and from each one's task name, the files it is changing and its recent actions, judge whether any of them owns the part where the problem is. When unsure whether it is theirs, send a message and ask.
2. **If one owns it, send it a message** (in Claude Code, `SendMessage`), and do not change the part it owns yourself. The message says: which file, which line, what you observe, how to reproduce it, whether your own change brought it in, and what you want it to do or what you are asking it.
3. **If you cannot fix it yourself, or you lack information or permissions someone else holds, message for help on your own initiative**; do not quietly work around it, and do not idle.
4. **If nobody owns it**: if it belongs to your own task, handle it yourself; if not, report it to the user or to whoever dispatched you, saying where the problem is and whom you suggest handing it to.
5. **Once the message is sent, carry on with the work in hand that does not depend on it**, and take that part up again when the reply comes. If the other side keeps not responding, report it to the user or to whoever dispatched you; do not take over the part it owns on your own.

How to judge a red on a file you did not change is in `session-wrapup.md` item 4 "Is another session in flight in the same repository?"; relaying another place's convention in a message follows `verify-before-claiming.md` "Relaying another place's convention carries its source or its original words".

This is not yet a check: whether you looked for an owner and whether you sent what should have been sent is left to review.

## Within one script, run the checks in parallel when they can be

**Criterion: this batch of checks do not depend on each other, and every one of them has to start a
subprocess to do its work (a build, a test run, a virtual machine, a python process). Then run them in parallel,
instead of waiting for them one at a time.**

Once it is fast, look again at where the remaining time goes: a case that idles on purpose (one that verifies, say,
that `wait` does not return once the timeout logic is broken) is not saved by parallelism.

**These three do not go parallel**:

- a later item reads an earlier item's product;
- they share one writable state (the same temp directory, the same target directory, the same device). To go parallel, give each item its own;
- each item is fast by itself, running only a few milliseconds.

Take the degree of parallelism (processes, and threads within a process) from an argument or an environment variable, falling back to `nproc`; do not hard-code it.

## Parallelism must not swallow the failures

There are only two ways to collect: record `pids+=($!)` when starting and take the exit code with `wait "$pid"`
one at a time; or have each parallel item write its own exit-code file, and judge them when collecting in the
order of the table the work was dealt from.
Do not use a `wait` with no arguments, and do not write `|| bad=1` inside the background body for the parent to read `$bad`.

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

Do not report success the way `<command> 2>/dev/null; echo "done"` does.
Any command that changes state must be verified by **reading the state back**: what is judged is the content read back, not the exit code of that command.

## Files under `/proc`, `/sys` and cgroupfs: read the contents to judge emptiness

File sizes in these places do not reflect the contents: procfs and cgroupfs report 0, sysfs reports 4096.
To judge whether such a file is empty, or whether there are processes, read the contents and judge those: `[[ -n "$(< "$cgroup_directory/cgroup.procs")" ]]`; do not use `[ -s <file> ]`, `stat`'s `%s`, or `find`'s `-size` and `-empty`.

S8 in `scripts/shell-lint.sh` judges this: any of these three forms applied to a path that literally starts with `/proc/` or `/sys/`, or whose file name is `cgroup.<name>`, turns red.
Paths held entirely in variables with an unrecognizable file name are not judged; languages other than shell (python's `os.path.getsize`, `st_size` and the like) are not judged and are left to review.

## Result collection needs a completeness gate

**Do this**: have the program under test report, on its final line, how many results it
emitted; the collector compares the count and discards the round on a mismatch.
That gate must itself be proven to go red first — feed it a fake program that claims
one more result than it actually emits, and it must fail.

## A script with a gate hands over its output only after judging it

When one script both produces a result and decides whether that result is usable, **the output goes to the caller only
after the verdict**.

**What to do**: write the result to a temporary file first and output it only once the gate passes; on red, stdout stays
empty. This gate must be shown to go red as well: change the script back to "output first, judge later", and the
self-test must go red.

## Assignments inside a subshell do not travel back to the parent

In `exit_code="$(run_one ...)"`, any variable `run_one` assigns is **empty in the parent**.
**Pass values through a file or an argument, not through a variable.** It is a check that goes red in `scripts/shell-lint.sh`.

## After a script edits a file, read it back — a compiler warning is a free signal

When a script does a string replacement on code or documents, two things to do:

1. **The replacement must assert it hit something**: zero replacements is an error to
   raise, not "ran the script, so it's done".
2. **Read back after editing**: grep for the new content, or just run it and see
   whether the behavior changed.

**Never wave off a compiler or linter warning.**

## Three silent failures at a process boundary

| Form | How to write it |
|---|---|
| `inner; exit_code=$?` under `set -e` | Take the exit code inside an `if`: `if inner; then exit_code=0; else exit_code=$?; fi` |
| `export <variable name>="$(<command>)"` / `local <variable name>="$(<command>)"` | Assign first, `export` second — write it as two lines |
| An environment variable used for a handshake leaking into a child process | `unset` it as soon as it is read; clear it with `env -u` when starting a child process |

**The criterion is "how many process levels does this value cross"**: cross one and you
have to ask whether it is still there at the next level, and whether it should be.

## The exit code of a pipeline is not the one you want

`$?` after `<command> | head`, and `<command> | grep <pattern> && <next action>`, both judge the last stage of the pipeline, not `<command>`.

**What to do**: to judge whether the earlier stage succeeded, use `${PIPESTATUS[0]}`,
or skip the pipe altogether — capture the output into a variable or file first, then
check the exit code before doing anything with it.

S7 in `scripts/shell-lint.sh` judges one form of this: in a script that sets `pipefail`, a pipeline ending in `grep -q` (`… | grep -q pattern`) turns red.
The fix: capture the earlier stage's output into a variable first, then `grep -q pattern <<<"$variable"`; that form, with no earlier stage, is not judged.

## Test images always go in a temporary directory

Never inside the repository. Image paths come from an environment variable, defaulting
to `${TMPDIR:-/tmp}`.

**The harness that creates an image deletes it itself when the test ends**: on success, on failure and on panic alike. Rust uses a Drop guard; shell uses `trap … EXIT`.
Keeping it to look at the scene is switched on by an explicit switch (an environment variable), and the path is printed when it is kept; without the switch it is deleted.
A gate stage that goes red may leave its scene in `$TMPDIR` and name the path in its remedy.
A cache meant to be reused across runs (build output and the like) is not an image: put it under `${GATE_CROSS_RUN_TMPDIR:-${TMPDIR:-/tmp}}` or a place the project sets, not in `$TMPDIR`.

`gate.sh` checks this: each run gives the stages a `TMPDIR` of that run's own, and at the end looks at what is left in it (the stage "No temporary files left behind", `跑完没留下临时文件`):
if every other stage is green, whatever is left goes red, listed by name and size, and is deleted on exit; if some stage went red, this item is recorded as not judged this run, and the run's temporary directory is kept whole with its path printed — delete it yourself once you have looked.
Of the directories kept this way on a red run, only the latest 3 are kept; older ones are deleted by a later run.
What it cannot see: harnesses that hard-code `/tmp` instead of going through `TMPDIR`, tests run by hand outside the gate, and anything created by child processes still running after the gate was interrupted.

## Look before running anything destructive

For `mkfs` / `dd` / `dmsetup remove`, **the target device must come from a variable, and `lsblk` must print it for
confirmation first**; never hard-code a literal `/dev/sdX`.
