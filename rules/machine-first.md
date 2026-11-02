<!-- generated-from: rules/machine-first.md sha256:18d50cd9b0241541c2df727a3a32aeeff6861cf0ebea22317726d09d3a5235d8 -->
<!-- doc-lint:rule-definition -->
# Machine first: separate "readable" from "verifiable"

**This project does not inherit common software engineering principles by default.** Each one
is put through the criterion again; the ones that fail are dropped.

## Stance: inherit sceptically, not by default

The default attitude toward established "best practice" is **doubt**, not compliance:
re-check each one's reasons, and do not keep doing it because "everyone does it this way".

## What this stance assumes, and how to overturn it

The whole methodology rests on one premise. **To argue against this stance, aim at this
premise, not at the "relaxed" and "kept" tables.**

### Premise: context windows are now in the 2×10⁵ – 10⁶ token range

Three tiers: 200K / 500K / 1M. Other vendors' numbers are not
written down here.

**Calibration**: this is the **advertised window ceiling**, not "the same accuracy
across the whole window." At what scale recall decay in the middle of a long context
starts to bite, this project has never measured.

- **What it holds up**: the relaxations of "keep PRs small", "minimise concepts",
  and "DRY by hand".
- **What it does not hold up**: "bound the scope of reasoning" is not among the
  relaxations.

### When a premise fails, take these back

This table states what observations would overturn the premise. Observe the
phenomenon in a row and the relaxation in that same row is withdrawn on the spot:

| What you observe | What is withdrawn |
|---|---|
| At this project's actual scale, the model misses a constraint that is already in its context | The "keep PRs small" relaxation; go back to slicing at a size that fits |
| Generated duplicate code no longer matches its generator | The "DRY" relaxation; back to no duplication |

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

- invariants
- compatibility of external formats and protocols
- reproducible tests
- fault domain isolation
- bisectable, revertible
- explicit contracts at module boundaries
- bounded reasoning scope

## One reminder in the other direction: documentation discipline is not up for dropping

The `writing-discipline.md` family (body text states only the current state, history at
the end, decision records in the kb) is not among the relaxations.
