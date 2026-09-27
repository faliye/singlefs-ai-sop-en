<!-- generated-from: templates/kb/checks-owed.md sha256:74c80b383b7c5b7e570e54f7935e18e0adf9d744be214a0abe2774974c2da6be -->
# Checks owed

**`checks-owed.md` holds checks we already know we want but cannot yet enforce.**

The line against `invariants.md`: an invariant is a property **decidable against one
image**, and the project's checks are its executable form; `checks-owed.md` holds requirements on
**code paths** — undecidable from an image, enforceable only by the gate at run time.

**The bar for writing one down**: you can state what it stops, how it goes red, and what
prerequisite is missing. Missing any of the three means it is not thought through yet;
do not write it (`../singlefs-ai-sop/rules/sop-first.md`).

<!-- doc-lint:registry name-col=2 -->

| # | Short name | What it stops | How it goes red | Prerequisite | Source |
|---|---|---|---|---|---|
| C1 | <short name for this owed check> | <the behaviour stopped> | <when it turns red> | <what is needed to implement it> | <which entry of decisions.md> |

**A number is an index, not a name**: cite it elsewhere as `<number> (<short name>)`,
enforced by `doc-lint.sh`.

**State**: <how many remain unimplemented, and what each is waiting on>

## Revision history

### <YYYY-MM-DD>
- Created.
