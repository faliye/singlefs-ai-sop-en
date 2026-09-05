---
name: decide
description: Record or change a singlefs design decision. Use it when settling a decision, overturning an old one, or finding that a choice cascades into others — covers the record format, the state machine, and how it must stay in step with the invariant list and the checker.
---
<!-- generated-from: skills/decide/SKILL.md sha256:e5319864188635c2a370138bf795f4b4647ffcc3b14456684993eb59706caa27 -->

# Recording a design decision

The rule lives in `rules/doc-discipline.md`, plus whatever the project has locally
about format and structure evolution.

## Only three states

| State | Meaning |
|---|---|
| **Settled** | Direction and details are both fixed; you can write code against it |
| **Half-settled** | Direction is fixed, details are not. **Say which detail is open**, or it is merely undecided |
| **Undecided** | Not decided. Say what has to be answered first |

## Format in `kb/decisions.md`

```markdown
## D<n> <name> —— <state>

<The conclusion in one sentence, imperative or declarative. Not "we might".>

Basis: <why. Numbers from elsewhere carry their source and measurement basis, and
are marked as not verified in this project.>

**Open**: <required when half-settled: name the gap.>
```

## Hard requirements

1. **A changed decision must state what overturned it**, and the old conclusion moves
   into the closing "Revision history" — no old conclusions in the body.
2. **Changing the format means updating `kb/invariants.md` and the checker in the same
   change.** A commit where the three disagree is not accepted.
3. **Before settling a decision, go through `kb/pitfalls.md`** and confirm you are not
   walking back into the same trap.
4. Where decisions cascade, **note it on both**, not just one.
5. **Finding that the reason that actually carries a settled decision has changed is
   itself a decision change.** Experiments routinely replace "the reason given when we
   settled it" with a different, decisive one; swap the basis in the body for the
   measured one and record the change — even when the state does not move.
   Leave only the original sentence and, three months on, nobody knows what is
   actually holding it up.
6. **Touch a clause a person settled, and you owe an open entry in `kb/checks-owed.md`.**
   A "pending review" note in the body guarantees nothing will ever look at it again —
   and the person who settled it does not know their ruling changed. Name the decision
   and the item in the entry; clear it only once the review has happened.

## When to settle and when to wait

The criterion: **does this decision change what the first piece of code looks like?**

Yes → settle it now. Dragging it out means rework.
No → it can wait; mark it undecided and say what has to be answered first.
