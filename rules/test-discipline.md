<!-- generated-from: rules/test-discipline.md sha256:f0f68a10da14e664d5aee90b673633e6f1e19d8066a7725c4d49de0391f5d34a -->
<!-- doc-lint:rule-definition -->
# Testing discipline

## A single observation does not count

N≥5 rounds. To call it "pass" every round must pass; to call it "fail" every round
must fail (**the thresholds are asymmetric**). If neither holds, report "unstable"
honestly and draw no conclusion.

**This applies only when the observation has an intrinsic source of variance** — real
I/O, concurrency, timing, randomness; at least one must be present. With none of them
the thing under test is a deterministic model: the same binary run N times is
byte-identical by necessity, **N=1 and N=5 carry exactly the same information**, and
writing "consistent across N rounds" merely dresses determinism up as stress-test-grade
evidence. Observed: three pure-arithmetic experiments all wrote "byte-identical across
N=5 rounds", and the whole batch was later struck down on review.

⇒ For a deterministic experiment the only honest phrasing is "running N rounds proves
there is no hidden state, not statistical stability"; its evidence strength comes from
mutation testing and from assertions that pin down absolute values, not from the round
count.

## Could not read ≠ read zero

If a metric comes back empty, retry and **void the whole round**. Never let it enter
the judgement as a zero.

## Crash consistency can only be verified by crash-point replay

Record every write request at the block layer, then truncate at **every** possible
crash point, replay, and run the checker.
**A write path that has not been through this is not verified** — an all-green unit
test suite is no evidence of crash consistency whatsoever.

## Functional correctness comes from model-based differential testing

Maintain an "ideal filesystem" in memory (just a HashMap; no performance, no crashes),
apply the same random operation sequence to both the model and the implementation, and
compare.

**This is the only functional oracle this project has.** Designing from scratch means
there is no reference implementation to compare against — the convenience a porting
project has, of "treat an existing tool's output as the right answer", does not exist
here; it has to be built.

## Before an experiment runs, the answer must not already exist

**Do not presuppose a conclusion, and do not consult any existing conclusion while
designing one** — this project's conclusion from last round, a settled result from
elsewhere, your own intuitive expectation: none of them.

The reason is mechanical: once the criterion is set to match the expected conclusion, it
can only pass when the result matches expectation. **This is the same disease as "an
experiment's failure clause must not make its conclusion unfalsifiable", with the lesion
moved forward into design** — at that point there are no numbers yet, so editing the
criterion leaves no trace and cannot be spotted afterwards.

**What you must read is the definition, not the answer.** The basis, parameters and
semantics of the thing under test have to be read verbatim (`verify-before-claiming.md`,
"'Is it settled' and 'what does it actually say' are two different questions"), whereas
"last round measured X" and "project Y says this way is faster" are answers, and reading
one is handing yourself the answer key.

**What to do**: pin down criteria, thresholds and discard clauses before the run, then go
look at the existing conclusions. Where the two disagree, record both as they stand —
never go back and edit the criterion. **Edit it and it is a new experiment: re-run.**

## An experiment's failure clause must not make its conclusion unfalsifiable

The easiest mistake to make when designing an experiment, and the hardest to catch
yourself: **defining "the effect did not show up" as an implementation bug**.

Write it that way and the experiment can only ever produce supporting evidence —
the effect appears and you record a finding, it does not appear and you record
"implementation is broken, discard the round". **No observation can refute the conclusion.**

Keep the two apart, and **run both controls**:

| Control | What it is for | Verdict when no difference shows |
|---|---|---|
| **Positive control**: a baseline where the effect is firmly established in the literature | Proves the measurement has discriminating power | **The implementation is broken, discard the round** |
| **Real baseline**: the design this project actually intends to build | Answers what the experiment is really asking | **A legitimate result, record it as such** |

Without the positive control, "no advantage" cannot be told apart from a broken
implementation. Without the real baseline you are measuring "does this mechanism exist"
rather than "what does this mechanism buy over what we already have".

⚠️ **The mirror image of the same discipline, and it has to be written down too: a failure
clause's antecedent can be written backwards, and once it is, the clause can never fire.**
Written backwards it does not look like *wrong*, it looks like **not triggered** — and in a
round's report "this clause did not fire" is **indistinguishable** from "this clause was checked
and the conclusion is fine".

Measured (2026-09-09): a clause pinned before the run said "if an arm's benefit appears **only
under the non-interleaved load**, record it as condition-unknown", while that round's finding was
that the benefit appears **only under the interleaved load** ⇒ the antecedent is permanently false, the
clause never fired, **and it should have**. The person who wrote it self-checked twice without noticing;
another leg caught it by reading the antecedent word for word.

⇒ **Every time you pin a failure clause, write the next sentence: "what observation would make
this fire".** If you cannot write that sentence, the clause is either backwards or vacuous, and
both amount to having no clause. This is the same thing as this section's main point seen from
the other end: the main point governs "the clause makes the conclusion unfalsifiable", this one
governs "the clause itself can never be triggered".

### Do not write a criterion as a conjunction; a threshold must not be a tautology of the arm's definition

The same discipline has two quieter failures; one experiment on 2026-09-13 hit each of them once:

| Form | Measured | Consequence |
|---|---|---|
| **Two quantities folded into one true / false** | The pre-run criterion said "peak doubles **and** runs never increase ⇒ sensitive"; one half held and the other half went the opposite way | The artifact reported a single `false`, and what each half said could only be recovered by going back through the grid rows — the conjunction erased the most informative cell |
| **The threshold is the definition of the arm under test** | The criterion said "≤ 3 segments per directory ⇒ locality holds", while that arm is defined as "home segment + overflow segment + at most one fallback" = 3 segments | That cell is **always true** for that arm; it was the only `true` of the round, and it proved nothing. Only against the other arm does it have any discriminating power |

⇒ **One line per quantity, each with its own verdict**; leave the conjunction to people. Before writing a threshold,
put the definition of the arm under test beside it: a cell whose threshold follows straight from the definition does not
count as a criterion.

## The positive control must run against **every** arm under test

When an experiment has N arms, running the positive control on just one of them means
the other N−1 **never went through the gate at all** — and it is usually one of those
un-gated arms that ends up producing the conclusion.

Observed: an experiment had two rule arms; the positive control was run against only
the first. Once run against the second, **it failed on the very control workload** —
by the experiment's own failure clause it should have been discarded on the spot, but
its numbers had already been written into a decision document.

**What to do**: the control loop must iterate over the full set of arms, not "just run
it against the first one". This is especially dangerous when an arm is added later —
whoever adds it usually only touches the arm under test, and forgets the control loop
exists.

## Comparing the arms only against each other cannot detect "every arm is wrong together"

**Equality between arms is a weak criterion**: it only rules out "one arm alone went
wrong". It does not rule out "the same formula is shared by every arm, and that formula
is wrong". In a cross-arm comparison the latter looks exactly like correctness —
**everything is equal**.

⇒ **Next to every cross-arm assertion there must be an assertion that pins down an
absolute value.** "The three arms have equal overhead" is not enough; you also need
"the overhead is exactly N", with N derived by independent arithmetic.

Observed: a three-arm experiment with 18 unit tests, one conservation check and one set
of positive controls — **all of them cross-arm comparisons and nothing else**. Mutation
testing was run three rounds in a row, and every round had entries where "not a single
test went red". They all exposed the same shape: doubling an interval parameter,
changing the accounting basis for record charging, making one arm perform no publish at
all — all three arms went wrong **together**, and the cross-arm comparisons still came
out equal. One of those errors left an arm **publishing not one root** across 200,000
operations, and no check raised an alarm.

**What to do**: once you have written a cross-arm assertion, ask yourself "if all three
arms were wrong **together**, who would notice". If you cannot answer, add one that
pins down an absolute value. **This and "the positive control must run against every
arm" are two sides of the same discipline** — that one is about every arm going through
the gate, this one is about the gate itself not being relative.

## An endpoint is not a trajectory: a quantity a clause feeds into a predicate must be reported as a trajectory

If some clause uses a quantity as the input of a stop / start / admission predicate, an experiment that reports only its
**end-of-run value** is not enough: an end value of 0 and "never positive" are two different sentences, and they call for
two different fixes.

Measured (2026-09-13): a placement experiment reported the end value of "the number of fully empty segments" — 0 in all
40 cells — and the debt line concluded from that "this quantity is never positive; the predicate can only be fed by
compaction". The second run reported the trajectory instead — a peak of 2, positive in 5% of 2000 rounds — and the truth
was "it does read positive; the foreground just consumes it on the spot", so the fix changed from "find the predicate
another input" to "stop the foreground from eating it". Reading the endpoint as the trajectory erred in exactly the more
hopeless direction.

⇒ A quantity consumed by a predicate is reported with at least three numbers: **the peak after the initial supply is
used up, the number of rounds it was positive, and the end value**. A report that gives only the end value may not use
trajectory words such as "never" or "always" in its body text.

## Mutation testing proves the assertions can go red, not coverage

Mutations only act on **the functions that already have assertions**. What it proves is
"every piece of code with an assertion has been mutation-tested", **not** "every piece
of code the conclusion rests on has been tested".

⇒ Never write "zero blind spots"; write "all N mutations were caught" and nothing more.
And once the conclusion is written, go back and ask: **is the arithmetic this conclusion
comes from covered by an assertion?**
Observed: an experiment claimed "all 9 mutations caught, zero blind spots" while the
arithmetic its conclusion came from had zero unit tests and zero mutations — had it
been wrong, nothing would have raised an alarm.

Two pitfalls that silently disarm mutation testing:

1. **Write constant assertions as addition, not subtraction.** In
   `assert_eq!(A - B, 240)`, a mutation that enlarges one constant **overflows at
   compile time** and gets recorded as an "invalid mutant" instead of "caught" — an
   assertion that should have gone red is silently disarmed. Written as
   `assert_eq!(A, B + 240)`, both sides survive to runtime.
   Observed: two mutations recorded as invalid both went red after the rewrite.
2. **An equivalent mutant is not a blind spot; account for it separately.** A mutation
   that agrees with the original on every input can never be caught, and does not count
   as a miss. When you judge one equivalent, pin the equivalence down as a test for the
   record, then substitute a mutation that really changes behaviour.

## The checker is the specification

Every entry added to the invariant list adds a check to the checker.
**The authoritative answer to "what is this format" is the checker's source, not the
documentation.**

## The check itself can also be wrong

Before asserting, confirm the check **has discriminating power** — e.g. when verifying
"there are no invalid blocks", also confirm the scan actually read some blocks;
otherwise finding none may only mean nothing was scanned.

After writing a check, ask: if the thing under test really were broken, would this go
red? If you cannot answer, it is not finished.

## A negative result must be separable from "the code never ran"

"Did not reproduce" does not mean "no problem". To claim "this path is correct", you
need a reading proving that path **actually executed**.
