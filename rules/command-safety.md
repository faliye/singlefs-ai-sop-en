<!-- generated-from: rules/command-safety.md sha256:a4fbf62c43e97df8c011a3730818929267671b22b53d9fc3395ef3d67a2f6847 -->
<!-- doc-lint:rule-definition -->
# Process and command discipline

Four of these are failing checks now, not reminders — `scripts/shell-lint.sh` judges
them: **killing processes by pattern match**, **carrying a value out of a subshell
through a variable**, **git's undo commands inside a script**, and **`rm -rf` on an
unguarded variable path**. The rest are still prose, because the criterion for
checking them mechanically is not worked out yet (`show-me-test.md`, "what the gate
can and cannot prove" — what is not done has to be said, not glossed over).

## Before anything you cannot take back, think once more

One question decides it: **after this step, can you get back?**

Two kinds cannot, and they are handled differently:

| Kind | Examples | What to do |
|---|---|---|
| **Throws away uncommitted work** | `git checkout <file>`, `git restore`, `git reset --hard`, `git clean` | `git stash` or `cp` a copy out first; use them only when you mean to discard *all* uncommitted changes to that file |
| **Deletes data outright** | `rm -rf`, `> file`, `sed -i`, `rsync --delete`, `mkfs`, `dd` | Look at the target first (`ls` / `git status` / `lsblk`); guard variable paths with `${VAR:?}` |

**Measured the hard way**: an attempt to undo one throwaway `sed` used
`git checkout <file>` and took every uncommitted change to that file with it.
That round was recovered from a just-amended commit still in the reflog — had that
commit not existed, the work was gone.

**So: run throwaway experiments on a copy**, not in the working tree.

**Scripts must never contain git's undo commands.** While a script runs, nobody is
watching `git status`, and these commands have no undo. To get to a clean state inside
a script, `git stash` first, or copy the whole repository and work on the copy.

**`rm -rf` on a variable path needs an empty-value guard.**
`rm -rf "$d/x"` with `$d` empty becomes `rm -rf /x`; written `rm -rf "${d:?}/x"`, the
shell errors out before `rm` runs. (`rm -rf "$d"` with nothing after it is fine: empty
gives `rm -rf ""`, which `rm` refuses.)

## `pkill -f` / `killall` are forbidden outright

The pattern string appears in the wrapper's own command line, so it **kills your own
shell.**

To stop a process: `ps` first, look at it, then kill by **literal pid** in a separate
second command. For counting, use structured criteria from `/proc` and exclude your
own process tree.

## QEMU virtual machines must write their pid to a file

Tear them down by the literal pid from that file; never by name matching.
A runaway VM will eat all memory, and name matching will take other tests down with it.

## Do not use echo to fake success

In `cmd 2>/dev/null; echo "done"`, that echo runs **unconditionally** — it prints
"done" even when the sudo failed.

Any command that changes state must be verified by **reading the state back**: after
`systemctl stop X`, confirm with `is-active`; the exit code of `stop` is not enough.

## Result collection needs a completeness gate

The program under test prints results and a script outside collects them — **losing a
line on that path is silent**. Consoles get polluted by BIOS escape sequences, kernel
logs and serial noise, and anchoring at start-of-line (`grep '^ANCHOR'`) quietly drops
the line whose start got overwritten. What you see outside is "one item fewer",
not "something failed".

**Do this**: have the program under test report, on its final line, how many results it
emitted; the collector compares the count and discards the round on a mismatch.
That gate must itself be proven to go red first — feed it a fake program that claims
N results and emits N−1, and it must fail.

## Assignments inside a subshell do not travel back to the parent

In `rc="$(run_one ...)"`, any variable `run_one` assigns is **empty in the parent**.
If result collection depends on that variable (a log path, say) it will collect zero
results forever while the exit code stays 0 — green light, wrong answer.
**Pass values through a file or an argument, not through a variable.**

Under `set -u` it is worse: the reference is not an empty value, it is an immediate
"unbound variable" that **takes the script down before it prints its diagnosis**.
Measured in this repository's own QEMU harness — five failure branches never printed
a single `howto`, and those branches are exactly where you need one when the harness
is lying. A comment cannot stop this, so it is now a failing item in
`scripts/shell-lint.sh`.

## After a script edits a file, read it back — a compiler warning is a free signal

When a script does a string replacement on code or docs, **a match that fails to hit
does not error — it just does nothing.** An indentation off by one space, a changed
quote style, a trailing space on the line — the replacement silently fails, and the
exit code is 0.

**Two things to do**:
1. **The replacement must assert it hit something**: zero replacements is an error to
   raise, not "ran the script, so it's done".
2. **Read back after editing**: grep for the new content, or just run it and see
   whether the behavior changed.

⚠️ **A compiler or linter warning is the cheapest signal for this failure mode — never
wave it off.** Observed: a replacement failed to take because of an indentation
mismatch, leaving a method as dead code. Every build reported it `never used`, and
that warning was ignored for a whole round — while the conclusion drawn from that
code **pointed in the wrong direction.**

## The exit code of a pipeline is not the one you want

`cmd | head` followed by reading `$?` gets `head`'s exit code, not `cmd`'s.
`cmd | grep x && do_something` has the same problem — it is judging whether `grep`
succeeded.

The result is a **swallowed failure**: `cmd` already died, and the script keeps
going, with every later step built on a result that does not exist.

**What to do**: to judge whether the earlier stage succeeded, use `${PIPESTATUS[0]}`,
or skip the pipe altogether — capture the output into a variable or file first, then
check the exit code before doing anything with it.

## Test images always go in a temporary directory

Never inside the repository. Accidentally committing a multi-gigabyte image into git
is an irreversible nuisance. Image paths come from an environment variable, defaulting
to `${TMPDIR:-/tmp}`.

## Look before running anything destructive

A mistyped device name in `mkfs` / `dd` / `dmsetup remove` destroys real data.
**The target device must come from a variable, and `lsblk` must print it for
confirmation first**; never hard-code a literal `/dev/sdX`.
