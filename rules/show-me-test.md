<!-- generated-from: rules/show-me-test.md sha256:936f73f30f956b9bcb2b1a03eac561f334459a10e5a584ac9c33a69fcce508d5 -->
<!-- doc-lint:rule-definition -->
# The acceptance rule: Show me test

> **Make every submitted patch review-worthy.**

## The criterion is the test

**Whether a patch lands is decided by automated verification.** Pass is pass and fail is fail.

## No patch without tests is accepted

**Change `crates/*/src/` or `crates/*/build.rs` and you must bring tests.** No exceptions; "this one is too simple" and
"I will add them in the next patch" do not count. Documentation and script changes are exempt, and so are changes that only touch comments in `.rs` files (only when the code, with comments and whitespace removed, is token for token the same as before; code blocks in doc comments that compile as doctests, and adding or removing doc comments or `// SAFETY:` comments, count as code; the judgement is `scripts/comment-only-change.py`).

Enforced by `scripts/show-me-test.sh` (`gate.sh` runs it as one stage).

## The final criterion is set by the project

Checks such as unit tests are fast feedback. **They are not
the acceptance criterion.** The acceptance criterion is bound to the thing under test: which
environment to bring up, which workload to run, which faults to inject — the project decides,
tests and verifies all of it itself, and the apparatus and its gate stages live in the project;
the shared gate carries none. The unimplemented list always carries `最终判据` (final criterion). Before
writing that key, write the acceptance criterion into the project's own kb: which environment, which workload,
which faults. Only a stage that does all of it declares `# gate-covers: 最终判据` in its header; a stage that
does part of it does not.
A stage covering any other item on the list declares `# gate-covers: <that item>` the same way (keys are copied
literally from `gate.sh`). Only when that stage ran and passed this round does
`gate.sh` move the item from the unimplemented list to "covered by which stage".

## The gate must not pretend to pass

Unimplemented gate stages must be **reported explicitly as unimplemented**, never
silently skipped.

**A project-local stage with nothing to judge this round exits 77**: `gate.sh` records it as "not run
this time" — not a pass, and not coverage. A skip that exits 0 cannot be told apart from "judged and passed"
in the summary; that half rests on whoever writes the stage.
Shared stages follow the same rule: with nothing to judge they exit 77 (`Show me test` and the naming lint exit 3), and `gate.sh` starts them in a way that understands 77.

**When a stage runs only part of its work, the skipped part is reported into the summary item by item**: cases skipped for a missing tool, samples not implemented for this language, local stages with no fixtures
are reported with `report_not_run` from `lib.sh` (a stage that does not source `lib.sh` appends a line to `$GATE_NOT_RUN_FILE`, and writes nothing when that variable is absent: it is absent when the stage runs on its own or is fed fixtures),
and `gate.sh` gathers them into the summary's "not run this time"; a warning printed only in the stage's own output does not count as reporting. A run that reported such a part is not recorded as "the last success" (`rules/preflight-discipline.md`),
and a local stage that reported one does not count as covering its `# gate-covers:` item either.

**Entries not recorded, skipped or discarded go into the log one by one**: when a success line or summary reports a count such as "not recorded N", "skipped N" or "discarded N", the same output lists those entries one by one (when there are many, the first N, then the count of the rest); writing them only into a temporary file that is deleted at the end does not count as listing them.

**Scanning zero items is not passing either.**
A check that scans a set of objects must report **how many items it checked** in its
success line; `gate-lint.sh` enforces this: it judges the last success line in the script,
which must carry a quantity counted in the script (a variable that was accumulated, an array length, the result of `wc -l` / `grep -c`); a script that genuinely scans no objects writes its own comment line
`# gate-lint:nocount <reason>`, with a reason of at least 8 characters.

**Reporting "how many were checked" is not enough: "which were not checked" must be listed one by one too, and that list must be computed on the spot.**
**The skip list must come from the same data as the scanned set, computed on the spot**; the success line reports both "ran <count>; did not run <count>: named one by one".
`gate-lint.sh` judges one half (`.sh` only): a script whose output reports a count such as "skipped N", "not recorded N", "discarded N" or "not run N" (the words it matches are `跳过`, `没记`, `作废`, `没跑`, `未跑`) must have, in the same script, an output line that lists the entries one by one; a line written into a file does not count.
Where the entries are listed elsewhere in a way it cannot recognise, write `# gate-lint:skip-listed <reason>` at the end of the line that reports the count, with a reason of at least 8 characters.
Whether the list comes from the same data as the scanned set and is computed on the spot, and whether what is listed is what the count counts, it does not judge; that rests on whoever writes the stage.

**Project-local stages follow the same rules as shared ones**, and are subject to `gate-lint` and `shell-lint` too:
`gate.sh` hands the project's registered gate-stage directory (default `.claude/gate.d/`) to both lints.

**The reach stops at the project's registered gate-stage directory.** Scripts elsewhere in the project (research scripts, hooks) sit outside the reach of these two lints.
If a project has such directories, hook up a local stage in the registered gate-stage directory that hands them to both lints — both, not one:
set `GATE_LINT_DIR` and `SHELL_LINT_DIR` respectively to the target directory when calling them.

## Time phases inside the process under test, not in the loop that relays its output

When a wrapper starts a child process, reads its output lines and relays each one onward, **do not use the moment each line was read as a phase clock**.

**What to do**: time inside the process under test and write the number into its result line. If arrival times must be used, read the child's output to the end first and relay afterwards.
When both an outer and an inner number exist, report a containment self-check: the outer phase must contain the inner one, and rounds where it does not are left out of the statistics.

The gate stage "Relay timing" (`转发计时`, `scripts/relay-timing-lint.py`) judges this: a loop in the project's `.rs` and `.py` files that reads a child process's output (which directories are scanned follows that script) must not both timestamp lines and relay them.

## Every breakage switch must go red

A breakage switch is a switch, used only by the self-test, that deliberately breaks one place when turned on.

- Turn every switch on one at a time and run the self-test: the self-test must exit non-zero, and go red in the cell the switch table names.
- A switch that does not go red when turned on, or goes red somewhere other than the named cell, is fixed on the spot: change the self-test so it goes red in that cell, or change the switch so it really breaks the named place; if neither can be fixed, delete the switch. No switch that cannot go red is kept.
- After adding a switch, changing the switch table, or changing the code a switch breaks, rerun the affected switches one by one.

Gates and hooks written in python check every switch, turned on one at a time, at the end of `--selftest` the way `scripts/break-switch-selftest.py` does; those written in shell carry one must-go-red case per switch in `scripts/selftest.sh`.
