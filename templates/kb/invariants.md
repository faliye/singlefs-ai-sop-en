<!-- generated-from: templates/kb/invariants.md sha256:6ece5493d403cb1adabd445011af2256ee1118c77e3d71d112ce1ad5f31cbab9 -->
# Invariant list

**The project's checks are the executable form of `invariants.md`.** Every entry added here means
a check added. A commit where the two disagree is not accepted.

Every invariant must be written in a **decidable** form — answerable "holds / does not
hold" against a single state or artefact of the system under test. Something that cannot be written that way is not yet
understood, and does not belong here.

## I-1 <category name>

**A number is an index, not a name**: every invariant has a short name, and every
citation elsewhere is written `<number> (<short name>)`.
Enforced by `doc-lint.sh` (`../singlefs-ai-sop/rules/kb-discipline.md`, item 5 "A number can serve as an index, never as a name").

<!-- doc-lint:registry name-col=2 -->

| Number | Short name | Invariant | Check state |
|---|---|---|---|
| I-1.1 | <the short name> | <decidable statement> | not implemented |

## Still to write

<Categories that cannot be written until some decision is settled.>

---

## Revision history

### YYYY-MM-DD
- Created.
