---
name: decide
description: Record or change this project's design decision. Use it when settling a decision, overturning an old one, or finding that a choice cascades into others — covers the record format, the state machine, and how it must stay in step with the invariant list and the checks that implement it.
---
<!-- generated-from: skills/decide/SKILL.md sha256:10171207514886e8d1cc98c70b8e87ac125ccd38b6d71ef7bba1e68f100d90fb -->

# Recording a design decision

The rule lives in `rules/kb-discipline.md`, plus whatever the project has locally
about format and structure evolution.

## Only three states

| State | Meaning |
|---|---|
| **Settled** | Direction and details are both fixed; you can write code against it |
| **Half-settled** | Direction is fixed, details are not. **Say which detail is open** |
| **Undecided** | Not decided. Say what has to be answered first |

## Format in `kb/decisions.md`

```markdown
## D<ordinal> <name> —— <state>

<The conclusion in one sentence, imperative or declarative. Not "we might".>

Basis: <why. Numbers from elsewhere carry their source and measurement basis, and
are marked as not verified in this project.>

**Open**: <required when half-settled: name the gap.>
```

## Hard requirements

1. **A changed decision must state what overturned it**, and no old conclusion stays in the body.
   If the file is registered in `.claude/history-carriers` at the project root (without that registry, every file under kb counts),
   the old conclusion moves into the closing "## Revision history"; if it is not registered, edit the body directly and the history lives in git
   (`rules/kb-discipline.md` item 8 "Body text states only the current state; history goes to a "Revision history" at the end").
2. **Changing the format means updating `kb/invariants.md` and the check that implements it in the same
   change.** A commit where the three disagree is not accepted.
3. **Before settling a decision, go through `kb/pitfalls.md`** and confirm you are not
   walking back into the same trap.
4. Where decisions cascade, **note it on both**, not just one.
5. **Finding that the reason that actually carries a settled decision has changed is
   itself a decision change.** Swap the basis in the body for the
   measured one; if the file is registered in `.claude/history-carriers`, also record the change in "## Revision history" — even when the state does not move.
6. **Touch a clause a person settled, and you owe an open entry in `kb/checks-owed.md`**,
   not just a "pending review" note in the body. Name the decision
   and the item in the entry; clear it only once the review has happened.

7. **An open item's question must have exactly one reading.**
   Test: write out each reading of the question separately — do they get the same answer?
   If not, the question is not finished. **Settle the question before arguing the
   rationale**.


## When to settle and when to wait

The criterion: **does this decision change what the first piece of code looks like?**

Yes → settle it now.
No → it can wait; mark it undecided and say what has to be answered first.
