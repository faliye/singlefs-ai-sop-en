<!-- generated-from: rules/show-me-test.md sha256:7624502887ba282ada04571205f712729610047362e2f756fe7b508af8df117d -->
<!-- doc-lint:rule-definition -->
# The acceptance rule: Show me test

> **Make every submitted patch review-worthy.**

## The criterion is the test

**Whether a patch lands is decided by automated verification.** Pass is pass and fail is fail.

## No patch without tests is accepted

**Change `crates/*/src/` or `crates/*/build.rs` and you must bring tests**, no exceptions.
Documentation and script changes are exempt, and so are changes that only touch comments in `.rs` files; a change touches only comments only when the code, with comments and whitespace removed, is token for token the same as before; code blocks in doc comments that compile as doctests, and adding or removing doc comments or `// SAFETY:` comments, count as code (the judgement is `scripts/comment-only-change.py`).

Enforced by `scripts/show-me-test.sh`.

## The final criterion is set by the project

Unit tests are fast feedback. **They are not the acceptance criterion.** The acceptance criterion is set by the project: which
environment to bring up, which workload to run, which faults to inject; the apparatus and its gate stages live in the project.

- Before writing `# gate-covers: 最终判据` (final criterion), write the acceptance criterion into the project's own kb.
- Only a stage that does all of it declares that key in its header; a stage that does part of it does not.
- A stage covering any other item on the unimplemented list declares `# gate-covers: <that item>` the same way (keys are copied
  literally from `gate.sh`). Only when that stage ran and passed this round does `gate.sh` change the item to "covered by which stage".

## The gate must not pretend to pass

| Situation | What to do |
|---|---|
| A stage is not implemented | Report it explicitly as unimplemented; never skip it silently |
| Nothing to judge this round | Exit 77 (`Show me test` and the naming lint exit 3), not `exit 0`; `gate.sh` records it as "not run this time" — not a pass, and not coverage |
| Only part of the work ran (cases skipped for a missing tool, samples not implemented for this language, local stages with no fixtures) | Report the skipped part item by item with `report_not_run` from `lib.sh` (a stage that does not source `lib.sh` appends a line to `$GATE_NOT_RUN_FILE`, and writes nothing when that variable is absent); a warning alone does not count as reporting. A run that reported such a part is not recorded as "the last success" (`rules/preflight-discipline.md`), and does not count as covering its `# gate-covers:` item either |
| A count such as "not recorded N", "skipped N", "discarded N" or "not run N" is reported (the words are `没记`, `跳过`, `作废`, `没跑`, `未跑`) | The same output lists those entries one by one (when there are many, the first N, then the count of the rest); writing them only into a temporary file that is deleted at the end does not count. The skip list comes from the same data as the scanned set, computed on the spot |
| A set of objects is scanned | The success line reports "ran <count>; did not run <count>: named one by one" |

**Scanning zero items is not passing either.** The last success line in the script must carry a quantity counted in the script (a variable that was accumulated, an array length, the result of `wc -l` / `grep -c`).

`scripts/gate-lint.sh` judges whether the success line carries a count, and whether a script that reports a skip count lists the entries one by one (that half `.sh` only); whether the list is computed on the spot and comes from the same data as the scanned set, it does not judge; that rests on whoever writes the stage. Both exemptions take a reason of at least 8 characters:
a script that genuinely scans no objects writes its own comment line `# gate-lint:nocount <reason>`; where the entries are listed elsewhere, write `# gate-lint:skip-listed <reason>` at the end of the line that reports the count.

**Project-local stages follow the same rules as shared ones**: `gate.sh` hands the project's registered gate-stage directory (default `.claude/gate.d/`) to `gate-lint` and `shell-lint` as well.
The reach of the two lints stops at the registered gate-stage directory: if a project has script directories elsewhere (research scripts, hooks), hook up a local stage in the gate-stage directory and set `GATE_LINT_DIR` and `SHELL_LINT_DIR` respectively to point at them — both lints, not one.

## Time phases inside the process under test, not in the loop that relays its output

When a wrapper starts a child process, reads its output lines and relays each one onward, **do not use the moment each line was read as a phase clock**.

- Time inside the process under test and write the number into its result line. If arrival times must be used, read the child's output to the end first and relay afterwards.
- When both an outer and an inner number exist, report a containment self-check: the outer phase must contain the inner one, and rounds where it does not are left out of the statistics.

The gate stage "Relay timing" (`转发计时`, `scripts/relay-timing-lint.py`) judges this: a loop in `.rs` and `.py` files that reads a child process's output must not both timestamp lines and relay them (which directories are scanned follows that script).

## Every breakage switch must go red

A breakage switch is a switch, used only by the self-test, that deliberately breaks one place when turned on.

- Turn every switch on one at a time and run the self-test: the self-test must exit non-zero, and go red in the cell the switch table names.
- A switch that does not go red when turned on, or goes red somewhere other than the named cell, is fixed on the spot: change the self-test so it goes red in that cell, or change the switch so it really breaks the named place; if neither can be fixed, delete the switch. No switch that cannot go red is kept.
- After adding a switch, changing the switch table, or changing the code a switch breaks, rerun the affected switches one by one.

Gates and hooks written in python check every switch, turned on one at a time, at the end of `--selftest` the way `scripts/break-switch-selftest.py` does; those written in shell carry one must-go-red case per switch in `scripts/selftest.sh`.
