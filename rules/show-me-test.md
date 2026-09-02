<!-- generated-from: rules/show-me-test.md sha256:49a8f4026cac58978ca49d6e53508d3ecf63b48b4302195797a5ebf34f7c4982 -->
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

Enforced by `scripts/gate.sh`, not by good intentions.

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

## Turn traps you have hit into checks that fail, not into reminder sentences

"Careful not to do Y while X" stops nobody who is typing commands by hand. **A check
that refuses to run at that moment does.** When a new trap is found, the first
reaction is "how does this become a red line in the gate", not "which document does
this go in".

## The final criterion is QEMU/KVM stress testing

Unit tests and model-based differential testing are fast feedback. **They are not
the acceptance criterion.** The acceptance criterion is running a real workload plus
crash injection under QEMU/KVM, with the checker all green afterwards.

## The gate must not pretend to pass

Unimplemented gate stages must be **reported explicitly as unimplemented**, never
silently skipped. A green gate that quietly did not run the crash tests is far more
dangerous than a red one.

Likewise: **batch scripts must not swallow per-round failures** (`|| true` and
friends), and output paths must not be reused across rounds. Put those two together
and a failed round quietly passes off the previous round's output as its own —
everything looks fine, only the numbers do not move. The criterion is "can this
output prove it came from this round": delete old output before the run, and check
for a completion marker that could only have been produced by this run.

# What the gate can and cannot prove

> **Gate proves evidence requirements, not semantic correctness.**

**This is the necessary counterweight to the epigraph "Make every submitted patch review-worthy".** Without it, "the gate
is all green" gets read as "the code is correct" — which is precisely the kind of
silent error this project most wants to avoid.

## Passes every gate stage and is still wrong

| Case | Why the gate cannot see it |
|---|---|
| the test tests the implementation, not the contract | it counts whether a test exists, it does not judge whether the test is right |
| the litmus declaration is itself wrong | declared Sometimes, measured Sometimes → "matches". **Without a control case** that proves nothing |
| the invariant itself is wrong | the checker will faithfully check a wrong rule, all green |
| the covered path is not the one that breaks | coverage is not correctness |
| the design is wrong | the gate cannot reach this layer at all |

## Corollaries

1. **A green gate does not mean the code need not be read.**
   Its value is freeing people from "is there a test at all" so they can look at
   **"is the test testing the right thing"** — the part only humans can do.

2. **Semantic correctness lives in `kb/decisions.md` and `kb/invariants.md`, not in
   the gate.** The gate only guarantees those two are **written down, implemented,
   and in sync with the code**; whether they are *right* is on humans.

3. **The gate is a floor, not a ceiling** (`rules/sop-first.md`). It guarantees
   nobody falls below a line; it guarantees nobody reaches correctness.

4. **Unimplemented stages stay explicitly listed.** A missing verification method
   means a whole class of errors that has never been looked at. `gate.sh` prints
   that list every time precisely so "it passed" is not mistaken for "it was
   verified".

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
