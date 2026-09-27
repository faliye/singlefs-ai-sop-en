<!-- generated-from: rules/machine-first.md sha256:3ee7120030743799640ae1bbb582cddd480409c8baf00d84c74a2d799b717d21 -->
<!-- doc-lint:rule-definition -->
# Machine first: separate "readable" from "verifiable"

**This project does not inherit common software engineering principles by default.** Each one
is put through the criterion again; the ones that fail are dropped.

## Stance: inherit sceptically, not by default

The default attitude toward established "best practice" is **doubt**, not compliance:
re-check each one's reasons, and do not keep doing it because "everyone does it this way".

## What this stance assumes, and how to overturn it

The whole methodology rests on two premises. **To argue against this stance, aim at these two
premises, not at the "relaxed" and "kept" tables.**

### Premise 1: context windows are now in the 2×10⁵ – 10⁶ token range

Three tiers: 200K / 500K / 1M. The 1M tier holds one crate together with its
tests, its kb, and its decision record in a single window. Other vendors' numbers are not
written down here.

**Calibration**: this is the **advertised window ceiling**, not "the same accuracy
across the whole window." At what scale recall decay in the middle of a long context
starts to bite, this project has never measured.

- **What it holds up**: the relaxations of "keep PRs small", "minimise concepts",
  and "DRY by hand".
- **What it does not hold up**: "bound the scope of reasoning" is not among the
  relaxations.

### Premise 2: concurrent interleavings can now be decided exhaustively

A judgement like "is this barrier enough" splits in two:

| Who | Does what | Precision |
|---|---|---|
| The tool | **Exhausts** every interleaving a given formal model permits, and returns a verdict | Complete for the formal model you actually wrote |
| The model | Translates the synchronisation pattern in the code into that formal model; enumerates which scenarios to ask about | **Incomplete — the gap is here** |

**Calibration**: the tool is precise, the model is not. Nothing guarantees you did not omit
a scenario that should have been asked, or that what you wrote down is what the code does today.

Which tool to use, how to show its verdicts have discriminating power, and how to bind the formal
model you wrote to the code: **the project decides, tests and verifies them itself**, in its
project-local rules. The shared gate carries none of this layer.

- **What it holds up**: "exhaustiveness is machine-checkable", and what follows
  from it — "prefer exhaustive explicit branches" and "only a bounded control-flow
  path count can be verified to the end". How to write that is in the "Branches" and
  "Functions and nesting" sections of `code-discipline.md`.

### When a premise fails, take these back

Each premise has a row stating what observation would overturn it. Observe the
phenomenon in a row and the relaxation in that same row is withdrawn on the spot:

| What you observe | What is withdrawn |
|---|---|
| At this project's actual scale, the model misses a constraint that is already in its context | The "keep PRs small" relaxation; go back to slicing at a size that fits |
| Generated duplicate code no longer matches its generator | The "DRY" relaxation; back to no duplication |
| The verdict of the tool the project uses to exhaust interleavings contradicts what real hardware shows | "Exhaustiveness is machine-checkable"; that tool demotes to reference and stops being a gate |

## When you meet an established principle, run this procedure

1. **Ask what problem it originally solved.** If you cannot say, drop it.
2. **Ask whether that problem still exists.** If what it solved was "people cannot
   remember", "people cannot read that much", or "review bandwidth is short", it
   most likely does not.
3. **Ask whether it has a second reason**, one that has nothing to do with humans (examples in
   "Kept (independent of who reads the code)"). If so, keep it.
4. **The rewritten reason must be able to land in the gate.** If it cannot, demote
   it to advice; do not write it as a rule.

## The criterion

> **Does this rule make the code easier to verify mechanically, or only easier on
> the human eye?**

- Only easier on the eye → **it can be dropped**
- Makes verification easier → **keep it**.

## Where the conclusions live

**How code is written** (names, branches, types, functions and nesting, comments,
duplication): the per-principle conclusions, together with how to write each one, are in
`code-discipline.md`, and that is the only place they live.
**Design- and process-level** principles are in "Relaxed (artefacts of human bandwidth)"
and "Kept (independent of who reads the code)".

## Relaxed (artefacts of human bandwidth)

| Old principle | Disposition |
|---|---|
| unify the design, minimise concepts | **relaxed** |
| keep PRs small | **relaxed** — bisectability and revertibility still stand |

⚠️ **What is relaxed is compression done to fit a human head, not making meaning
explicit.** See "AI-friendly ≠ unreadable" in `engineering-philosophy.md`.

## Kept (independent of who reads the code)

- crash consistency, invariants
- on-disk format compatibility
- reproducible tests
- fault domain isolation
- bisectable, revertible
- explicit contracts at module boundaries
- bounded reasoning scope

## One reminder in the other direction: documentation discipline is not up for dropping

The `writing-discipline.md` family (body text states only the current state, history at
the end, decision records in the kb) is not among the relaxations.
