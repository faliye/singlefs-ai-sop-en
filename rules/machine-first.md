<!-- generated-from: rules/machine-first.md sha256:0aa7ae3483c2b45e258cf9d1adae32ecb642b93571424c102c36200d6a86ea4b -->
<!-- doc-lint:rule-definition -->
# Machine first: separate "readable" from "verifiable"

Many common software engineering principles grew on the premise that **one person
has to hold the whole thing in their head**. That premise is failing: code is
produced by humans and models together, and reading code is itself a job that can
go to a model.

**This project does not inherit those principles by default.** Each one is put
through the criterion again; the ones that fail are dropped.

## Stance: inherit sceptically, not by default

The default attitude toward established "best practice" is **doubt**, not compliance.
The reason is in `engineering-philosophy.md`, "Why the old commandments need re-deriving", and is
not repeated here: most of these principles are floors set to the limit of the human brain, and
that limit is no longer the constraint.

This does not mean they are all wrong — it means **their reasons must be re-checked**,
and "everyone does it this way" is not one of them.

## What this stance assumes, and how to overturn it

"Machine capacity passed that line some time ago" is the foundation the whole
methodology rests on. **It is a falsifiable empirical claim, not an article of
faith.** So this section states the two premises it depends on, and what
observation would overturn them — **to argue against this stance, aim at these two
premises, not at the "relaxed" and "kept" tables.**

### Premise 1: context windows are now in the 2×10⁵ – 10⁶ token range

Three tiers: 200K / 500K / 1M. The 1M tier holds one crate together with its
tests, its kb, and its decision record in a single window. Basis: this round of
work on this repository runs in a 1M-tier session. Other vendors' numbers are not
written down here — a number you cannot check on the spot is not evidence.

**Calibration**: this is the **advertised window ceiling**, not "the same accuracy
across the whole window." Recall decay in the middle of a long context is a known
phenomenon, and this project has never measured at what scale it starts to bite.

- **What it holds up**: the relaxations of "keep PRs small", "minimise concepts",
  and "DRY by hand". Their original reason was entirely "it does not fit at once",
  and at the 1M tier that reason no longer holds automatically.
- **What it does not hold up**: "bound the scope of reasoning" is not among the
  relaxations. The window got bigger but is **still finite**, and accuracy decays
  with distance — so module boundaries and explicit contracts are worth more than
  before, not less.

### Premise 2: concurrent interleavings can now be decided exhaustively

For a judgement like "is this barrier enough", **intuition cannot produce a
re-checkable answer.** That job now splits in two:

| Who | Does what | Precision |
|---|---|---|
| The tool | **Exhausts** every interleaving a given formal model permits, and returns a verdict | Complete for the formal model you actually wrote |
| The model | Translates the synchronisation pattern in the code into that formal model; enumerates which scenarios to ask about | **Incomplete — the gap is here** |

**Calibration**: the tool is precise, the model is not. The tool answers only the
question you wrote down; nothing guarantees you did not omit a scenario that should have
been asked, or that what you wrote down is what the code does today.

Which tool to use, how to show its verdicts have discriminating power, and how to bind the formal
model you wrote to the code are all bound up with the thing under test, so **the project decides,
tests and verifies them itself**, in its project-local rules. The shared gate carries none of this layer.
Basis: singlefs judges its memory-ordering declarations with herd7 / LKMM, and records the tool version in its `.claude/kb/verification-build.md`.

- **What it holds up**: "exhaustiveness is machine-checkable", and what follows
  from it — "prefer exhaustive explicit branches" and "only a bounded control-flow
  path count can be verified to the end". How to write that is in the "Branches" and
  "Functions and nesting" sections of `code-discipline.md`.

### When a premise fails, take these back

Premises are not permanent. Observe any of the following and the matching
relaxation is withdrawn on the spot:

| What you observe | What is withdrawn |
|---|---|
| The model drops a mid-context fact at this project's actual scale — misses a constraint already in the window | The "keep PRs small" relaxation; go back to slicing at a size that fits |
| Generated duplicate code no longer matches its generator | The "DRY" relaxation; back to no duplication |
| The verdict of the tool the project uses to exhaust interleavings contradicts what real hardware shows | "Exhaustiveness is machine-checkable"; that tool demotes to reference and stops being a gate |

**This table is this file's own disproof step** (`evidence-discipline.md`): a
premise for which you cannot state "what would overturn it" is not a premise, it
is a belief.

## When you meet an established principle, run this procedure

1. **Ask what problem it originally solved.** If you cannot say, drop it — a rule
   whose reason cannot be stated will not hold anyway.
2. **Ask whether that problem still exists.** If what it solved was "people cannot
   remember", "people cannot read that much", or "review bandwidth is short", it
   most likely does not.
3. **Ask whether it has a second reason.** Many rules happen to also solve a
   problem that has nothing to do with humans (see the "kept" table). If so →
   keep it, **and rewrite its stated reason to that one.**
4. **The rewritten reason must be able to land in the gate.** If it cannot, demote
   it to advice; do not write it as a rule.

## The criterion

> **Does this rule make the code easier to verify mechanically, or only easier on
> the human eye?**

- Only easier on the eye → **it can be dropped**
- Makes verification easier → **keep it, but rewrite the reason** — stop hanging it
  on "readability"; a rule hung on readability will not survive the next time
  someone asks "why?"

## Where the conclusions live

**How code is written** (names, branches, types, functions and nesting, comments,
duplication): the per-principle conclusions, together with how to write each one, are in
`code-discipline.md`, and that is the only place they live.
**Design- and process-level** principles are in the two tables below.

## Relaxed (artefacts of human bandwidth)

| Old principle | Its original reason | Disposition |
|---|---|---|
| unify the design, minimise concepts | the maintainer has to fit it in | **relaxed** — this is exactly where "one tree holds everything" came from |
| keep PRs small | review bandwidth | **relaxed** — bisectability and revertibility still stand |

⚠️ **What is relaxed is compression done to fit a human head, not making meaning
explicit** — the latter is a gain for models too. See "AI-friendly ≠ unreadable" in
`engineering-philosophy.md`.

## Kept (independent of who reads the code)

| Principle | The real reason (rewritten) |
|---|---|
| crash consistency, invariants | physics, not psychology |
| on-disk format compatibility | a permanent external contract; it cannot be changed |
| reproducible tests | the precondition for any judgement, human or machine |
| fault domain isolation | a bug's blast radius has nothing to do with who reads the code |
| bisectable, revertible | **the larger the change volume, the more these are worth**, not less |
| explicit contracts at module boundaries | **without a contract you cannot verify one piece on its own** |
| bounded reasoning scope | **incremental verification requires bounded reasoning scope — context windows are finite too** |

The last two are the ones most easily discarded as "readability". They are not.

## One reminder in the other direction: documentation discipline is not up for dropping

The `writing-discipline.md` family (body text states only the current state, history at
the end, decision records in the kb) is **not an artefact of human bandwidth**.

In a document mixing history with the present, **a model has a harder time than a
person telling which value is current** — it has no "I remember this changed last
week" to fall back on, so it takes everything at face value.
**Stale or self-contradictory documents hurt models no less than people; they hurt
them more.**

By the same token, `kb/decisions.md` is rising in value, not falling: it is the only
place that can tell whoever comes next — human or model — **why this is the way it
is, and on what basis.**
