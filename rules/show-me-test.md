<!-- generated-from: rules/show-me-test.md sha256:649a28bd3481acbe4b11d62f6ff62e60440c169b8a7c85802a11e5f00dea750f -->
<!-- doc-lint:rule-definition -->
# The acceptance rule: Show me test

> **Make every submitted patch review-worthy.**

## The criterion is the test

**Whether a patch lands is decided by automated verification.** Pass is pass and fail is fail.

## No patch without tests is accepted

**Change `crates/*/src/` and you must bring tests.** No exceptions; "this one is too simple" and
"I will add them in the next patch" do not count. Documentation and script changes are exempt.

Enforced by `scripts/show-me-test.sh` (`gate.sh` runs it as one stage).

## A new test must first be shown to go red

**Every "verification" must be able to fail.** The right procedure: **break the code under test,
confirm the test goes red, then put it back.** A test that cannot be shown to go red is the same as no test.

Scripts only see whether a test exists and cannot verify this step: **state in the commit message
how you confirmed it goes red** — which line you broke, which assertion you saw fail. If you cannot
write that down, you did not do it.

Where a mutation harness exists, go one step further: turn "break this → that assertion
goes red" into a **checked-in mutation list** that the replay gate keeps re-running. The
commit-message account is the fallback for repos without a harness, not the first
choice.

**Design argument follows the same procedure**: for every hole an adversarial round finds, first
build a world in the model that **must report non-zero**, then change the rule.

## Turn traps you have hit into checks that fail, not into reminder sentences

When to add one follows the "Boundary" section of `sop-first.md`: record it as debt first and add it in a batch; one that keeps recurring right now is added on the spot.

When a new trap is found, the first reaction is "how does this become a red line in the gate",
not "which document does this go in": make it a check that refuses to run on the spot, not a
reminder sentence like "careful not to do Y while X".

### But first ask which layer the trap is on: inside one apparatus, or in the measurement basis

**The criterion is not "was this trap turned into a check that goes red", it is "which
layer is this trap on"**:

| Where the trap is | Where the check goes |
|---|---|
| in the behaviour of one piece of code | next to that code is enough |
| **in the measurement basis of the model** (how the same quantity is to be computed, whether the boundary counts, what the unit is) | **between apparatuses**: when two of them compute the same quantity, a check must force them onto the same number, and go red when they disagree |

**Quick criterion**: stand up a second apparatus, do the same thing from scratch — would
you step into it again? If yes, the trap is in the measurement basis, and the check has to
stand across apparatuses.

**A cross-apparatus check must pin "which quantity this name refers to"**: two quantities get two
registered names, each pinned to its own value ("one semantic concept, one name across the repo" in
`code-discipline.md`).

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

**Batch scripts must not swallow per-round failures** (`|| true` and
friends), and output paths must not be reused across rounds. The criterion is "can this
output prove it came from this round": delete old output before the run, and check
for a completion marker that could only have been produced by this run.

**Scanning zero items is not passing either.**
A check that scans a set of objects must report **how many items it checked** in its
success line; `gate-lint.sh` enforces this: it judges the last success line in the script,
which must carry a quantity counted in the script (a variable that was accumulated, an array length, the result of `wc -l` / `grep -c`); a script that genuinely scans no objects writes its own comment line
`# gate-lint:nocount <reason>`, with a reason of at least 8 characters.

**Reporting "how many were checked" is not enough: "which were not checked" must be listed one by one too, and that list must be computed on the spot.**
**The skip list must come from the same data as the scanned set, computed on the spot**; the success line reports both "ran N; did not run M: named one by one".
This is not a check yet: `gate-lint` does not look at whether a script that reports a count also lists what it skipped, so it rests on whoever writes the stage.

# When a change is not rolled out across the whole repo at once, every excluded file gets registered

When a spec, an operation or a change
covers only part of the files, **whatever it did not cover gets registered in an exclusion
table**, one entry per line with the reason after `#`. **No exclusion table means the whole
repo has to change.**

The test is not "it is a hassle to change", it is "changing it would make the sentence false,
or would make some gate stop working": another project's terminology and verbatim citations,
a check's own input, a tool's own pattern table, the sentence that states this very change —
those change into errors, so register them; everything else changes.

**An entry in the exclusion table pointing at a path that does not exist is a failure.** The table has to be readable by one command.

**The number in the success line needs something pinning it too.** `gate-lint` does not cover this; it rests on whoever writes the stage.

**Project-local stages follow the same rules as shared ones**, and are subject to `gate-lint` and `shell-lint` too:
`gate.sh` hands `.claude/gate.d/` to both lints.

**The reach stops at `.claude/gate.d/`.** Scripts elsewhere in the project (research scripts, hooks) sit outside the reach of these two lints.
If a project has such directories, hook up a local stage in `.claude/gate.d/` that hands them to both lints — both, not one:
set `GATE_LINT_DIR` and `SHELL_LINT_DIR` respectively to the target directory when calling them.

# What the gate can and cannot prove

> **Gate proves evidence requirements, not semantic correctness.**

## Passes every gate stage and is still wrong

| Case | What the gate cannot reach |
|---|---|
| the test tests the implementation, not the contract | it counts whether a test exists, it does not judge whether the test is right |
| the declaration is itself wrong | declaration and measurement agree → "matches", though the declaration is wrong. **Without a control case** that proves nothing |
| the invariant itself is wrong | the checks will faithfully judge a wrong rule, all green |
| the covered path is not the one that breaks | coverage is not correctness |
| the number in the prose disagrees with its own artifact | the replay's **range** assertion pins only which band the conclusion lands in, not the number in the prose: the wider the range, the further the prose can drift |
| two apparatuses compute different numbers for the same quantity | assertions and mutations take effect **inside one apparatus**; across apparatuses nothing is comparing anything |
| the design is wrong | the gate cannot reach this layer at all |

## Corollaries

1. **A green gate does not mean the code need not be read.** People look at
   **"is the test testing the right thing"**.

2. **Semantic correctness lives in `kb/decisions.md` and `kb/invariants.md`, not in
   the gate.** The gate only guarantees those two are **written down, implemented,
   and in sync with the code**; whether they are *right* is on humans.

3. **The gate is a floor, not a ceiling** (`rules/sop-first.md`).

4. **Unimplemented stages stay explicitly listed.** `gate.sh` prints that list every time.
   When a project stage declares coverage of an item (`# gate-covers:`),
   the item leaves the list only if that stage ran and passed this round.

5. **A handed-back list that a machine confirms is complete is not thereby judged right.**
   For work a subagent judges item by item (sweeps, re-reviews), what can be made into a check is "every item has a verdict, every verdict is one of the recognised kinds, and the reason quotes that item's own words";
   that stops "the sample was enough" and "wave the whole group through", but not wrong verdicts. So after the completeness check, still spot-check and re-judge across **every kind of verdict**; redo the stretch where a spot check hits a wrong verdict, then sample again.

## No categories by source; evidence only

This project **defines no categories by source** and writes no special rules for
any of them. There is one division only: **submissions that carry evidence, and
those that do not.**
