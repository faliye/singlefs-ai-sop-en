<!-- generated-from: templates/kb/invariants.md sha256:46504eaa947a9f4d77daed9664c2b8d73c40ffbdda25e16d1ce645cc36378123 -->
# Invariant list

**The project's checks are the executable form of `invariants.md`.** Every entry added here means
a check added. A commit where the two disagree is not accepted.

Every invariant must be written in a **decidable** form — answerable "holds / does not
hold" against a single state or artefact of the system under test. Something that cannot be written that way is not yet
understood, and does not belong here.

## I-1 <category name>

**A number is an index, not a name**: every invariant has a short name, and every
citation elsewhere is written `<number> (<short name>)`.
Enforced by `doc-lint.sh` (`../singlefs-ai-sop/rules/kb-discipline.md`, item 5).

<!-- doc-lint:registry name-col=2 -->

| ID | Short name | Invariant | Check state |
|---|---|---|---|
| I-1.1 | <the short name> | <decidable statement> | not implemented |

## Still to write

<Categories that cannot be written until some decision is settled.>

---

## Revision history

### YYYY-MM-DD
- Created.
