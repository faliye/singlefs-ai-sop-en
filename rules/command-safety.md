<!-- generated-from: rules/command-safety.md sha256:94a3aaf6271cf6f5381b02e1be5f49669c25736afb14c167ff1ea6b91f1782f4 -->
<!-- doc-lint:rule-definition -->
# Process and command discipline

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
**Pass values through a file, not through a variable.**

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
