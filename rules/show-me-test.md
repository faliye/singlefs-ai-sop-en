<!-- generated-from: rules/show-me-test.md sha256:81c42f4b11ad0b53e15af9c4126b1336b34ce1a8faa2b0400fe920fb4a6737be -->
<!-- doc-lint:rule-definition -->
# The acceptance rule: Show me test

> **Make every submitted patch review-worthy.**
>
> **Contribution throughput may be unbounded; acceptance throughput is evidence-bound.**

**This is the largest difference between this project and the Linux kernel, and
the most important rule here.**

## The gate raises the floor; it does not screen people out

The first line is the purpose, the second is the reason. That order matters.

**The gate does not exist to keep anyone out. It exists to raise every submission
to the line where it is worth spending human time on.** The mechanical parts —
whether tests exist, whether declarations match what was measured, whether docs
and implementation are in sync — are done by scripts, which frees human attention
for the part only humans can judge: **is this test testing the right thing?**

The second line says why this is necessary: wherever submissions come from, that
side only grows, while human review bandwidth has not changed. **The only thing
that scales with it is evidence.**

Put the other way round: a submission that arrives without evidence spends someone
else's time asking "did you verify this, and how?" — the gate moves that round trip
forward onto the submitter's own machine. **That saves work on both sides; it is
not an obstacle course.**

## The criterion is the test

**Whether a patch lands is decided by automated verification.** Every patch should
be rigorously tested; **every carefully tested, responsible submission is welcome.**

Pass is pass and fail is fail — and that protects the submitter: whether your patch
lands depends on how solidly it is verified, not on who you are.

## No patch without tests is accepted

**Change `crates/*/src/` and you must bring tests.** No exceptions, no "this one is
too simple", no "I will add them in the next patch". Documentation and script
changes are exempt.

Enforced by `scripts/show-me-test.sh` (`gate.sh` runs it as one stage), not by
good intentions.

## A new test must first be shown to go red

**Every "verification" must be able to fail.** After writing one, ask yourself: if
this code really were broken, would my check raise the alarm?

The right procedure: **break the code under test, confirm the test goes red, then
put it back.** A test that cannot be shown to go red is the same as no test.

Scripts cannot verify this step (they only see whether a test exists), so **state
in the commit message how you confirmed it goes red** — which line you broke, which
assertion you saw fail. If you cannot write that down, you did not do it.

Where a mutation harness exists, go one step further: turn "break this → that assertion
goes red" into a **checked-in mutation list** that the replay gate keeps re-running. The
commit-message account is the fallback for repos without a harness, not the first
choice — an account is read once, whereas a checked-in list re-proves itself on every
replay.

**This also governs design argument**: for every hole an adversarial round finds, first
build a world in the model that **must report non-zero**, then change the rule. Otherwise
nothing can check whether the fix is right — it stays green afterwards, because that
error was never under test in the first place.

## Turn traps you have hit into checks that fail, not into reminder sentences

"Careful not to do Y while X" stops nobody who is typing commands by hand. **A check
that refuses to run at that moment does.** When a new trap is found, the first
reaction is "how does this become a red line in the gate", not "which document does
this go in".

### But first ask which layer the trap is on: inside one apparatus, or in the measurement basis

**A check standing inside one apparatus does not govern the next one.** Assertions and
mutation testing only take effect inside the apparatus they live in; stand up a second
apparatus, model the same thing again from scratch, and that check will not say a word.

Measured (2026-09-06): a counting model took the "capacity × fill rate" budget for the
number of objects of one kind, then added a second kind on top of it — 126% of a disk's
worth of objects. It was fixed the same day, with an assertion left behind to go red.
**Hours later another model made the same mistake** (135% this time), and that assertion,
living in a different apparatus, said nothing: the new apparatus had its own unit tests
green, every mutation caught, the gate green. Two models reported numbers 1.5× apart for
**the same physical quantity** and nothing anywhere compared them. The wrong number went
into a settled clause.

⇒ **The criterion is not "was this trap turned into a check that goes red", it is "which
layer is this trap on"**:

| Where the trap is | Where the check goes |
|---|---|
| in the behaviour of one piece of code | next to that code is enough |
| **in the measurement basis of the model** (how the same quantity is to be computed, whether the boundary counts, what the unit is) | **between apparatuses**: when two of them compute the same quantity, a check must force them onto the same number, and go red when they disagree |

**Quick criterion**: stand up a second apparatus, do the same thing from scratch — would
you step into it again? Yes ⇒ the trap is in the measurement basis, and only a
cross-apparatus check helps.
⚠️ This is where whoever fixes the trap stops most easily: they really did turn it into a
check that goes red, **the evidence is complete and the gate is green**, so they never ask
again how far that check reaches.

⚠️ **A cross-apparatus check may pin only the value, not the quantity.** Measured (2026-09-13): a format constant had
the same name and the same value, 78, in four apparatuses, and the check comparing the kb's registered value with each
apparatus's source stayed green. Two of the apparatuses used it as the whole record header; the other two used it as the
header's ten fields and added three later-settled increments themselves to get 95 — two quantities, one name, one value.
The registered value happened to be the smaller quantity, so the two apparatuses that treated it as the whole header
undercounted by 17 bytes all along, and nothing raised an alarm.
⇒ **A cross-apparatus check must pin "which quantity this name refers to"**: two quantities get two registered names,
each pinned to its own value; a check that only asks "is the value right" stays silent when one name is used for two
quantities (the other side of "one semantic concept, one name across the repo" in `code-discipline.md`).

## The final criterion is set by the project

Unit tests and model-based differential testing are fast feedback. **They are not
the acceptance criterion.** The acceptance criterion is bound to the thing under test: which
environment to bring up, which workload to run, which faults to inject — the project decides,
tests and verifies all of it itself, and the apparatus and its gate stages live in the project;
the shared gate carries none. The unimplemented list always carries `最终判据` (final criterion). Before
writing that key, write the acceptance criterion into the project's own kb: which environment, which workload,
which faults. Only a stage that does all of it declares `# gate-covers: 最终判据` in its header; a stage that
does part of it does not, because the gate cannot tell whether it is complete and the summary would claim too much.
A stage covering any other item on the list declares `# gate-covers: <that item>` the same way (keys are copied
literally from `gate.sh`). Only when that stage ran and passed this round does
`gate.sh` move the item from the unimplemented list to "covered by which stage".

## The gate must not pretend to pass

Unimplemented gate stages must be **reported explicitly as unimplemented**, never
silently skipped. A green gate that quietly did not run the crash tests is far more
dangerous than a red one.

**A project-local stage with nothing to judge this round exits 77**: `gate.sh` records it as "not run
this time" — not a pass, and not coverage. A skip that exits 0 looks exactly like "judged and passed"
in the summary, and the gate cannot tell the two apart; that half rests on whoever writes the stage.

Likewise: **batch scripts must not swallow per-round failures** (`|| true` and
friends), and output paths must not be reused across rounds. Put those two together
and a failed round quietly passes off the previous round's output as its own —
everything looks fine, only the numbers do not move. The criterion is "can this
output prove it came from this round": delete old output before the run, and check
for a completion marker that could only have been produced by this run.

**Scanning zero items is not passing either.** Write the search scope of a criterion a
little too narrowly and every object gets skipped at the first step — neither passing nor
failing, and the tail still reports green with nobody able to tell.
⇒ A check that scans a set of objects must report **how many items it checked** in its
success line; `gate-lint.sh` enforces this.

**Reporting "how many were checked" is not enough: "which were not checked" must be listed one by one too, and that list must be computed on the spot.**
A stage can honestly report how many items it checked while missing a whole other half of its objects: both statements are true,
and the reader cannot see that the second one exists. Measured (2026-09-16, singlefs's rerun stage): it reported "ran 119 this time",
`gate-lint` was all green, and none of the 9 timing rows in that table had run; 6 of them neither ran nor appeared on any line.
What it hid were two real problems: one experiment's retained artifact had long stopped matching its source, and another panicked outright in a release build.
The root cause was a hand-copied skip list — hard-coded as 4 numbers, while what really did not run was every timing row in the table plus those 4.
**So the skip list must come from the same data as the scanned set, computed on the spot**; the success line reports both "ran N; did not run M: named one by one".
This is not a check yet: `gate-lint` does not look at whether a script that reports a count also lists what it skipped, so it rests on whoever writes the stage.

**The number in the success line needs something pinning it too.** It is a success line, so it never goes red; it reports a count,
so it satisfies "a check that scans a batch reports how many it checked"; it is not a rejection, so none of the other three `gate-lint` rules reaches it.
Put together, a miscounted statistic can stay green in the gate indefinitely, and it is exactly the conclusion people read.
Measured (2026-09-16, singlefs's kb-rot stage): it reported "326 checks owed, 0 paid off"; the true numbers were 297 and 29 —
the boundary was hard-coded to one heading, the table had been moved above that heading, and so everything paid off was counted as owed.
**So a statistic in a success line is either pinned with `want=` in a discrimination fixture, or it stays out of the success line.**

**Project-local stages follow the same rules as shared ones.** They reject submitters
just like shared stages, so they are subject to `gate-lint` and `shell-lint` too —
`gate.sh` hands `.claude/gate.d/` to both lints (the first scan over singlefs's local
stages found 7 rejections with no way out).

⚠️ **The reach stops at `.claude/gate.d/`.** Scripts elsewhere in the project (research scripts, hooks) reject people just the same, yet sit outside the reach of these two lints.
Measured (2026-09-18, singlefs): running gate-lint on its own over the whole repository found 84 rejections in these two kinds of directory with no way out, and nobody had reported them before.
⇒ If a project has such directories, hook up a local stage in `.claude/gate.d/` that hands them to both lints, setting `GATE_LINT_DIR` and `SHELL_LINT_DIR` respectively to the target directory when calling them —
without them the shared scripts still scan the SOP's own package by default, and judging a fixture directory red or green then mixes in the real repository's scripts.
Hooking up gate-lint alone does only half the job: measured (2026-09-19, singlefs), shell-lint run on its own over the same research scripts still reports 18 findings, and the local stage that hooks up only gate-lint sees none of them.

# What the gate can and cannot prove

> **Gate proves evidence requirements, not semantic correctness.**

**This is the necessary counterweight to the epigraph "Make every submitted patch review-worthy".** Without it, "the gate
is all green" gets read as "the code is correct" — which is precisely the kind of
silent error this project most wants to avoid.

## Passes every gate stage and is still wrong

| Case | Why the gate cannot see it |
|---|---|
| the test tests the implementation, not the contract | it counts whether a test exists, it does not judge whether the test is right |
| the declaration is itself wrong | declaration and measurement agree → "matches", though the declaration is wrong. **Without a control case** that proves nothing |
| the invariant itself is wrong | the checker will faithfully check a wrong rule, all green |
| the covered path is not the one that breaks | coverage is not correctness |
| the number in the prose disagrees with its own artifact | the replay's **range** assertion pins only which band the conclusion lands in, not the number in the prose: the wider the range, the further the prose can drift (measured: the prose said 1.475×, the kept artifact says 1.625×, the range is [1.15, 2.10] — all green) |
| two apparatuses compute different numbers for the same quantity | assertions and mutations take effect **inside one apparatus**; across apparatuses nothing is comparing anything |
| the design is wrong | the gate cannot reach this layer at all |

## Corollaries

1. **A green gate does not mean the code need not be read.**
   Its value is freeing people from "is there a test at all" so they can look at
   **"is the test testing the right thing"** — the part only humans can do.

2. **Semantic correctness lives in `kb/decisions.md` and `kb/invariants.md`, not in
   the gate.** The gate only guarantees those two are **written down, implemented,
   and in sync with the code**; whether they are *right* is on humans.

3. **The gate is a floor, not a ceiling** (`rules/sop-first.md`). It guarantees
   nobody falls below a line; it does **not** guarantee that anyone reaches
   correctness.

4. **Unimplemented stages stay explicitly listed.** A missing verification method
   means a whole class of errors that has never been looked at. `gate.sh` prints
   that list every time precisely so "it passed" is not mistaken for "it was
   verified". When a project stage declares coverage of an item (`# gate-covers:`),
   the item leaves the list only if that stage ran and passed this round.

5. **A handed-back list that a machine confirms is complete is not thereby judged right.**
   For work a subagent judges item by item (sweeps, re-reviews), what can be made into a check is "every item has a verdict, every verdict is one of the recognised kinds, and the reason quotes that item's own words" —
   that stops "the sample was enough" and "wave the whole group through", but not wrong verdicts. So after the completeness check, still spot-check and re-judge across **every kind of verdict**; redo the stretch where a spot check hits a wrong verdict, then sample again.
   Measured (2026-09-18, singlefs): candidates were handed to sweep agents by group, dozens to a hundred-odd rows per group; a sweep agent wrote one boilerplate sentence and waved a whole group through, and every group containing known rot was waved through;
   after switching to row-by-row verdicts with a machine completeness check, an attacker could still mechanically generate a boilerplate report that passed the checker, and a report judging everything "needs a change" still fell outside the re-judging pool, which sampled only "unrelated".

## Why this rule

**Contribution throughput is becoming unbounded, while human review bandwidth has
not changed.**

Wherever submissions come from — more people, better tools, more automation — the
supply side only grows. "Humans reading code line by line" does not keep up. The
only thing that scales with it is **automated verification**: it does not care where
a patch came from or how elegant it is, only whether it passes.

## Putting the criterion on evidence is exactly how we avoid treating sources differently

**Screening by source is the discriminatory option, and it does not even work.**

The moment review intensity is decided by who submitted, identity has replaced
evidence — **and a patch does not become better or worse because of who wrote it.**
Identity is a poor predictor: it wrongs the careful newcomer and waves through the
familiar name's careless submission alike.

Evidence is different. It is the same measure for everyone, and **the submitter can
apply it in advance** — run the gate before sending and you know where you stand.
An identity-based criterion can never offer that.

So this project **defines no categories by source** and writes no special rules for
any of them. There is one division only: **submissions that carry evidence, and
those that do not.**

**So the quality of the gate is this project's ceiling.** `scripts/` matters more
than any crate.
