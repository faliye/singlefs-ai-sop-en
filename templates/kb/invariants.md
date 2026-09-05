<!-- generated-from: templates/kb/invariants.md sha256:61cdf75db8550ec3b4edf6b0059d27abaac24744d47bdb6917777728dbe030d4 -->
# Invariant list

**The checker is the executable form of `invariants.md`.** Every entry added here means
a check added to the checker. A commit where the two disagree is not accepted.

Every invariant must be written in a **decidable** form — answerable "holds / does not
hold" against a single image. Something that cannot be written that way is not yet
understood, and does not belong here.

## I-1 <category name>

**A number is an index, not a name**: every invariant has a short name, and every
citation elsewhere is written `<number> (<short name>)`.
Enforced by `doc-lint.sh` (`singlefs-ai-sop/rules/kb-discipline.md`, item 5).

<!-- doc-lint:registry name-col=2 -->

| ID | Short name | Invariant | Checker state |
|---|---|---|---|
| I-1.1 | <the short name> | <decidable statement> | not implemented |

## Still to write

<Categories that cannot be written until some decision is settled.>

---

## Revision history

### YYYY-MM-DD
- Created.
