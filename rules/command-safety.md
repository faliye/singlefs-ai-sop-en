<!-- generated-from: rules/command-safety.md sha256:9c713ebeacf3fb1a14d1a2c1fba18edc45671b27e3d3d13e080bfc7b079690ca -->
<!-- doc-lint:rule-definition -->
# Process and command discipline

Ten of these are failing checks, judged by `scripts/shell-lint.sh`: **killing processes
by pattern match** (`pkill -f`, `killall`), **`pgrep -f`**, **carrying a value out of a
subshell through a variable**, **git's undo commands inside a script**, **`rm -rf` on
an unguarded variable path**, **a `wait` with no arguments**, **a pipeline ending in `grep -q` in a script that sets `pipefail`**, **judging whether a file under `/proc`, `/sys` or cgroupfs is empty by its size**, **a space inside a prefix-removal pattern on `$*` or `$@`**, and **cutting text shown to people by bytes**. Whether a test harness deleted the images it created when it finished is judged by `gate.sh`'s "No temporary files left behind" (`跑完没留下临时文件`). Creating an unregistered file or directory at the project root is judged by the hook `scripts/claude-hooks/root-entry-guard.sh` and by `gate.sh`'s "New project-root entries" (`项目根的新条目`). An unregistered symbolic link in the repository is judged by `gate.sh`'s "Soft links" (`软链接`). A command that discards output and also judges a count or emptiness is recorded and reminded about by the hook `scripts/claude-hooks/discarded-output-count-check.sh`. Build directories and repository copies a subagent has not deleted before handing back are judged by the hook `scripts/claude-hooks/handback-scratch-check.sh`.

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

A path joined from several variables (`"$a/$b"`, `"${root}/${name}"`) can land elsewhere when one piece is empty, and `${joined:?}` only guards against the whole string being empty: on the line that joins it, every piece carries a guard (`dir="${root:?}/${name:?}/x"`); a fixed path is written as the literal absolute path.
S5 in `scripts/shell-lint.sh` judges this: when the argument of `rm -rf` is a variable joined in this file, and the line that joins it has a piece without a guard, it goes red.

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

**After making something parallel, prove again that it can go red**: feed it an
input that must go red, and see whether the parallel version still goes red.

## Files under `/proc`, `/sys` and cgroupfs: read the contents to judge emptiness

File sizes in these places do not reflect the contents: procfs and cgroupfs report 0, sysfs reports 4096.
To judge whether such a file is empty, or whether there are processes, read the contents and judge those: `[[ -n "$(< "$cgroup_directory/cgroup.procs")" ]]`; do not use `[ -s <file> ]`, `stat`'s `%s`, or `find`'s `-size` and `-empty`.

S8 in `scripts/shell-lint.sh` judges this: any of these three forms applied to a path that literally starts with `/proc/` or `/sys/`, or whose file name is `cgroup.<name>`, turns red.
Paths held entirely in variables with an unrecognizable file name are not judged; languages other than shell (python's `os.path.getsize`, `st_size` and the like) are not judged and are left to review.

## Assignments inside a subshell do not travel back to the parent

In `exit_code="$(run_one ...)"`, any variable `run_one` assigns is **empty in the parent**.
**Pass values through a file or an argument, not through a variable.** It is a check that goes red in `scripts/shell-lint.sh`.

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

## Prefix and suffix removal on `$*` and `$@` happens once per parameter

`${*#pattern}` and `${@#pattern}` (and `##`, `%`, `%%`) remove from each parameter separately and then join; when the pattern contains a space, the part it means to remove exists only in the joined string, so nothing is removed.
To remove from the joined string, join first: `joined="$*"`, then write `"${joined#* -- }"`.

S9 in `scripts/shell-lint.sh` judges this: any of these forms whose pattern contains a space goes red.

## Cut text shown to people by characters, not bytes

GNU `cut` with `-c` or `-b`, and `head` or `tail` with `-c`, all count bytes; cutting inside a multibyte character leaves half of it, and whoever reads the output as UTF-8 reports bad bytes.
Cut text meant for people (failure lines, explanations, log excerpts) with `clip_text <bytes>` from `scripts/lib.sh`: cut by bytes, then drop the trailing partial character.
Cutting that only ever sees ASCII (hex digests and the like) keeps cutting by bytes and writes `# shell-lint:bytes-ok <reason>` on that line.

S10 in `scripts/shell-lint.sh` judges this: a line that has one of these byte cuts, prints outward (`say`, `bad`, `howto`, `echo`, `printf` or `✗`), and lacks that marker goes red.
A cut stored in a variable and printed on another line is not recognised; that is left to review.

## A command you draw a conclusion from keeps its output

A command you will cite to say "zero matches", "no errors" or "the exit code shows …" does not send stderr to `/dev/null` and does not discard stdout (`2>/dev/null`, `&>/dev/null` and `>/dev/null 2>&1` all count as discarding).
A 0 printed by a form like `grep -c pattern file 2>/dev/null || echo 0` is not taken as zero matches.

For empty output or a 0, first confirm the command really ran: look at its exit code (append `; echo rc=$?` to the same command) and its error output, and only once it ran take the result as zero matches.

The hook `scripts/claude-hooks/discarded-output-count-check.sh` judges one half of this: it judges after a Bash command has run, and when the command discards output and the same command also judges a count or emptiness (`grep -c`, `wc -l`, `test -z`, `[ -z`, `|| echo 0` and the like), it writes an entry to its detection record and adds a one-line reminder; it does not refuse.
The forms it recognises are those in that hook's `DISCARD_FORMS` and `COUNT_FORMS`; a command that carries `# discarded-output: <why discarding is fine>` is recorded without a reminder. Whether a conclusion was drawn from that command, it does not judge.
A project may leave it unregistered (its file header says `# hook-registration: optional`); how to register it is in the hook's file header.

## Before a subagent hands back, it deletes the build directories and repository copies it created

The repository copies a subagent made in its scratch directory, and the build directories it compiled into with its own `CARGO_TARGET_DIR`, are deleted by the subagent itself before it hands back:

- **Delete**: the build directories this subagent created this time (`target`, the directory `CARGO_TARGET_DIR` points at, and any other cache carrying `CACHEDIR.TAG`) and its repository copies (clones, copied repositories, git worktrees).
  Remove a worktree from its source repository with `git worktree remove --force <path>`, so the registration there goes too; `rm -rf` the rest.
- **Do not delete**: reports, artifacts that are to be committed, replay material the main agent still has to check; anything a predecessor, the main agent or another agent created, even in the same scratch directory;
  anything inside the project root (the project's own `target`, worktrees Claude Code created inside the project) — those belong to the main agent.
- Before deleting, record the size with `du -sh`; the handback report carries one line saying what was deleted and how big each was.
- For anything not deleted, the handback report carries one line per path: "Not deleted <full path>: <why>". Merely citing the path as a source ("ran on the copy at …") does not count.

The hook `scripts/claude-hooks/handback-scratch-check.sh` checks this, attached in two places:

- PreToolUse of the handback tool (`SubagentHandback`): stops the handback before it happens. If a build directory, worktree or repository copy that sits in a temporary directory, was created while one of this subagent's own tool calls had not yet finished,
  and was mentioned in its calls, is still there, and the report being handed back has no line for it, the handback is refused and the list is given to the subagent.
- SubagentStop: the fallback when the subagent stops without going through the handback tool. A delivered handback report that carries the line lets it through (a refused handback does not count);
  after a stop is blocked, writing the line in the reply or in a written report and then stopping lets it through (merely running du, ls or Read on it does not count).

The project registers both in `.claude/settings.json`; how to write them is in that hook's header.
What it cannot recognise still has to be deleted: paths that live only in a variable, directories a script created internally that no call mentions, anything more than one level below a directory it created,
source copies with no `.git` that were never built in, and anything created by a detached process (`nohup`, `setsid`, `&`) after the call returned. Whether the report carries the line saying what was deleted and how big, it does not check.
