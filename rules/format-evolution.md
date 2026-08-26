<!-- generated-from: rules/format-evolution.md sha256:0b40cab3a0e8b970f9d0a8f40ecfaebad96c5047e8ade05462030e2c5a1c424b -->
<!-- doc-lint:rule-definition -->
# Format evolution discipline

**Until the first external user appears, the on-disk format is soft** — it can be
torn up and redone at any time, with no backward compatibility required. This is the
largest degree of freedom a personal project has over an upstream one; use it fully.

But **the conceptual model must move slowly**: throwing away code and bytes costs
nothing, while a mental model like "how is space accounted for" or "what is a
snapshot", once wrong, soaks into a hundred decisions before anyone notices.

**Corollary: the thing to be careful with is `kb/decisions.md`, not the `.rs` files.**

## Hard constraints

- A format change **must update** `kb/invariants.md` and the checker in the same
  step. A commit where the three are out of sync is not accepted.
- A decision change **must be recorded** in `kb/decisions.md`, with what overturned it.
- Once there is an external user this document is void and strict compatibility takes
  over — at that point, change this document and bump `VERSION`.

## Start from writing, not from reading

Building the read-only side first leads you to design a format that is elegant to
read and miserable to write.
**How you write determines how you read**: allocation policy, transaction boundaries
and COW ordering are all products of the write path; the read path just follows
pointers.

All the hard problems are on the write side. Doing reads first is mistaking the easy
half for progress.
