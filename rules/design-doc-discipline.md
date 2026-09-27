<!-- generated-from: rules/design-doc-discipline.md sha256:dbe3b89b77beab2c73bab53ab1bd12b8b6fb716c61adbbbfd8c568c3230f7eda -->
<!-- doc-lint:rule-definition -->
# Design document discipline

**Applies to**: README, design notes, `records/` — **things humans read through.**

Give the context, make the argument, write it to be read; do not give way to machines.
Knowledge documents for model retrieval follow `kb-discipline.md`.

## 1. Body text states only the current state; history goes to the end

No historical statements in the body — no "it used to be X, now it is Y", no "it was
once called A". History that must be kept goes into a "Revision history" section at
the end:

```markdown
## Revision history

### YYYY-MM-DD
- Was X / now Y / basis for the change: Z
```

**Corollaries**:
- Changed a decision? **Edit the body directly.** Do not annotate "(was X)" beside it.
- A conclusion overturned? **Delete the old conclusion from the body** and put
  "was X / now Y / what overturned it" at the end.
- No `~~strikethrough~~`, `[deprecated]`, `(superseded by XX)` in the body.
- Same for code comments: write "why it is this way now", not "how it used to be".
  **Pitfall comments in checking code are the exception** ("this bypass measurably
  got through, hence this check"). The criterion is what it points at:
  pointing at **why this code looks the way it does now** → keep;
  pointing at **a version that no longer exists** → delete.

`CLAUDE.md` and `rules/*.md` follow a different discipline; see `rules-discipline.md`.

History statements in body text, and where the history section sits, are enforced by
`scripts/doc-lint.sh`, which scans only `.md`; the clause on code comments rests on review.

## 2. Length must match the weight of the change

The criterion and the yardstick are both in the "Length must match the weight of the
change" section of `writing-discipline.md`; they are not copied here.

## 3. Where persuasion is needed, persuade properly

A design document answers "why should it be done this way" and "how do I get
started". Setup, examples and trade-offs spelled out for those two — **none of that
is redundancy.**

Criterion: with this paragraph removed, would the reader still agree? Could they
still get started? Yes → it can go. No → keep it.
