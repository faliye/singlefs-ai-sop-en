<!-- generated-from: rules/command-safety.md sha256:dbfa4f6060a6fb598183bda793dc8b1c74183a0284fa2bcdfd8047bdb51c33b3 -->
<!-- doc-lint:rule-definition -->
# Process and command discipline

What is written in scripts is judged by `scripts/shell-lint.sh` (S1 to S10); the hooks and gate stages each section names judge the rest; which forms each one recognises is whatever the script says.
Hooks are registered in `.claude/settings.json`; how is in each hook's file header.

## Before anything you cannot take back, think once more

The criterion: **after this step, can you get back?**

| Kind | Examples | What to do |
|---|---|---|
| **Throws away uncommitted work** | `git checkout <file>`, `git restore`, `git reset --hard`, `git clean` | `git stash` or `cp` a copy out first; use them only when you mean to discard *all* uncommitted changes to that file |
| **Deletes data outright** | `rm -rf`, `> file`, `sed -i`, `rsync --delete`, `mkfs`, `dd` | Look at the target first (`ls` / `git status` / `lsblk`); guard variable paths with `${VAR:?}` |

- **Run throwaway experiments on a copy**, not in the working tree.
- **Scripts must not contain git's undo commands.** To get to a clean state, `git stash` first, or copy the whole repository and work on the copy.
- **`rm -rf` on a variable path needs an empty-value guard**: write `rm -rf "${build_directory:?}/cache"`. `rm -rf "$build_directory"` with nothing after it needs no guard.
- For a path joined from several variables, every piece carries a guard on the line that joins it (`dir="${root:?}/${name:?}/x"`); a fixed path is written as the literal absolute path. S5 in `scripts/shell-lint.sh` judges a joining line that has a piece without a guard.

## Do not create files or directories at the project root

Do not create a file or directory at the project root (the top level of the repository): drafts and logs go in `$TMPDIR` or the session's scratch directory; scripts, records and outputs go in the directories the project has set for them.
Recreating something after deleting it counts as creating it, and so do entries ignored by `.gitignore`. `.git`, `.claude`, `CLAUDE.md` and `.singlefs-ai-sop-version` need no registration.

When something really has to be created at the root:

1. Ask the user first: the name, what goes in it, and why it cannot go into an existing directory.
2. Once the user agrees, register one line `<name>  # reason` in `.claude/root-entries` at the project root: the name is the single top-level component, with no other `/`; the reason says what goes in it and why it sits at the root, at least 4 characters; one line per name.
3. Then create it. Commit the registry and the new entry in the same commit.
4. When the entry is deleted, delete its line from the registry in the same change.

Something a tool rebuilds every time (a build directory and the like) is registered once.

The hook `scripts/claude-hooks/root-entry-guard.sh` refuses before the write, reports the new entries after a Bash command has run, and has the user confirm a change to the registry; the gate stage "New project-root entries" (`项目根的新条目`, `scripts/root-entries.py`) judges the entries added relative to the diff base and how the registry is written.
Whether the user agreed and whether the reason holds is left to review.

## No symbolic links in the repository

The repository (from the project root down, `.git` excepted) holds no symbolic links: tracked, untracked and ignored ones all count. To use a file from elsewhere, copy it in, or write its real path where it is used.
This package's repositories and every project that installs the package follow this, each registering under its own project root.

One that really has to stay is registered as one line `<path relative to the project root>  # reason` in `.claude/soft-link-exempt` at the project root: no leading `/`, no `.` or `..`; the reason says what it points to and why it cannot be replaced by a copy, at least 4 characters; one line per path.
When the link is deleted or replaced by a regular file, delete its line in the same change.

The gate stage "Soft links" (`软链接`, `scripts/soft-links.py`) judges this.

## `pkill -f` / `killall` are forbidden outright

- **Stopping a process**: `ps` first, look at it, then kill by literal pid in a separate command; or `scripts/proc.py stop <pid>` (TERM first, then KILL if the process is still there after `--grace` seconds, default 10; it refuses when the target is the process issuing the command or one of its ancestors).
- **Waiting for a process to end**: do not use `pgrep -f`. For a background job you started yourself, use `wait "$pid"`; for anything else, use the literal pid: `until ! kill -0 "$pid" 2>/dev/null; do sleep 5; done`; for one you did not start, have whoever starts it write its pid to a file.
- **The pid was not recorded**: find it with `scripts/proc.py find <executable-name> [--argument <argument>]`, then `scripts/proc.py wait <pid> --timeout <seconds>` (exit code 3 if it has not exited when time is up).
- **Counting**: use structured criteria from `/proc` and exclude your own process tree.

In scripts this is judged by S2 and S3 in `scripts/shell-lint.sh`; commands typed by hand are refused before they run by the hook `scripts/claude-hooks/pattern-process-guard.sh`; the two use the same criteria.

## Parallelism must not swallow the failures

- There are only two ways to collect: record `pids+=($!)` when starting and take the exit code with `wait "$pid"` one at a time; or have each item write its own exit-code file, and judge them when collecting in the order the work was handed out.
- Do not use a `wait` with no arguments, and do not write `|| bad=1` inside the background body for the parent to read `$bad`. Where the exit code really is collected elsewhere, write `# shell-lint:exit-collected <how it is collected>` on that line.
- Each item writes its output to its own file, not straight to stdout; collection reads them back in the order the work was handed out.
- However many items were handed out, that many have to come back: count them when collecting, and turn the whole thing red when the count does not match.
- After making something parallel, feed it once an input that must go red, and see whether it still goes red.

S6 in `scripts/shell-lint.sh` judges a `wait` with no arguments.

## Files under `/proc`, `/sys` and cgroupfs: read the contents to judge emptiness

File sizes in these places do not reflect the contents. To judge whether such a file is empty, or whether there are processes, read the contents and judge those: `[[ -n "$(< "$cgroup_directory/cgroup.procs")" ]]`;
do not use `[ -s <file> ]`, `stat`'s `%s`, or `find`'s `-size` and `-empty`.

S8 in `scripts/shell-lint.sh` judges these three forms in shell when applied to a path that literally starts with `/proc/` or `/sys/`, or whose file name is `cgroup.<name>`; other languages are left to review.

## Assignments inside a subshell do not travel back to the parent

In `exit_code="$(run_one ...)"`, any variable `run_one` assigns is empty in the parent. **Pass values through a file or an argument.**
Judged by `scripts/shell-lint.sh`.

## The exit code of a pipeline is not the one you want

`$?` after `<command> | head`, and `<command> | grep <pattern> && <next action>`, both judge the last stage of the pipeline.
To judge the earlier stage, use `${PIPESTATUS[0]}`, or capture the output into a variable or file first, then check the exit code before doing anything with it.

S7 in `scripts/shell-lint.sh` judges: in a script that sets `pipefail`, a pipeline ending in `grep -q`. The fix: capture into a variable first, then `grep -q pattern <<<"$variable"`.

## Test images always go in a temporary directory

- Images never go inside the repository; their paths come from an environment variable, defaulting to `${TMPDIR:-/tmp}`.
- **The harness that creates an image deletes it itself when the test ends**: on success, on failure and on panic alike. Rust uses a Drop guard; shell uses `trap … EXIT`.
- Keeping it to look at the scene is switched on by an explicit environment variable, and the path is printed when it is kept.
- A gate stage that goes red may leave its scene in `$TMPDIR` and name the path in its remedy.
- A cache meant to be reused across runs (build output and the like) goes under `${GATE_CROSS_RUN_TMPDIR:-${TMPDIR:-/tmp}}` or a place the project sets, not in `$TMPDIR`.

`gate.sh`'s stage "No temporary files left behind" (`跑完没留下临时文件`) judges this: each run gives the stages a `TMPDIR` of their own, and anything left in it at the end goes red. When another stage went red, the run's temporary directory is kept whole with its path printed — delete it yourself once you have looked.
Harnesses that hard-code `/tmp`, and tests run by hand outside the gate, it cannot see.

## Prefix and suffix removal on `$*` and `$@` happens once per parameter

`${*#pattern}` and `${@#pattern}` (and `##`, `%`, `%%`) remove from each parameter separately and then join.
To remove from the joined string, join first: `joined="$*"`, then write `"${joined#* -- }"`.

S9 in `scripts/shell-lint.sh` judges: any of these forms whose pattern contains a space.

## Cut text shown to people by characters, not bytes

`cut -c`, `cut -b`, `head -c` and `tail -c` all count bytes. Cut text meant for people (failure lines, explanations, log excerpts) with `clip_text <bytes>` from `scripts/lib.sh`.
Cutting that only ever sees ASCII (hex digests and the like) keeps cutting by bytes and writes `# shell-lint:bytes-ok <reason>` on that line.

S10 in `scripts/shell-lint.sh` judges a line that cuts and also prints outward; a cut stored in a variable and printed on another line is left to review.

## A command you draw a conclusion from keeps its output

- A command you will cite to say "zero matches", "no errors" or "the exit code shows …" does not discard stderr and does not discard stdout (`2>/dev/null`, `&>/dev/null` and `>/dev/null 2>&1` all count as discarding).
- A 0 printed by a form like `grep -c pattern file 2>/dev/null || echo 0` is not taken as zero matches.
- For empty output or a 0, first confirm the command really ran: append `; echo rc=$?` to the same command and look at its exit code and its error output.

The optional hook `scripts/claude-hooks/discarded-output-count-check.sh` records and reminds after the command has run; it does not refuse. A command that carries `# discarded-output: <why discarding is fine>` is recorded without a reminder.

## Before a subagent hands back, it deletes the build directories and repository copies it created

- **Delete**: the build directories this subagent created this time (`target`, the directory `CARGO_TARGET_DIR` points at, any other cache carrying `CACHEDIR.TAG`) and its repository copies (clones, copied repositories, git worktrees).
  Remove a worktree from its source repository with `git worktree remove --force <path>`; `rm -rf` the rest.
- **Do not delete**: reports, artifacts that are to be committed, replay material the main agent still has to check; anything a predecessor, the main agent or another agent created, even in the same scratch directory; anything inside the project root (the project's own `target`, worktrees Claude Code created inside the project) — those belong to the main agent.
- Before deleting, run `du -sh`; the handback report carries one line saying what was deleted and how big each was.
- For anything not deleted, the handback report carries one line per path: "Not deleted <full path>: <why>". Merely citing the path as a source ("ran on the copy at …") does not count.

The hook `scripts/claude-hooks/handback-scratch-check.sh` blocks before the handback and on SubagentStop. What it cannot recognise still has to be deleted: paths that live only in a variable, directories a script created internally that no call mentions, anything more than one level below a directory it created, source copies with no `.git` that were never built in, and anything created by a detached process (`nohup`, `setsid`, `&`).
