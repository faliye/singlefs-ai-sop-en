<!-- generated-from: rules/design-doc-discipline.md sha256:42bf1c9d03def1c42c37ea4295d22ce9467ff8056e0229aae1f3f3e37bb92d3c -->
<!-- doc-lint:rule-definition -->
# Design document discipline

**Applies to**: README, design notes, `records/` — **things humans read through.**

Knowledge documents for model retrieval follow `kb-discipline.md`.

## 1. Body text states only the current state; history goes to the end

No historical statements in the body — no "it used to be <old approach>, now it is <new approach>", no "it was
once called <old name>". History that must be kept goes into a "Revision history" section at
the end:

```markdown
## Revision history

### YYYY-MM-DD
- Was <old approach> / now <new approach> / basis for the change: <why it changed>
```

**Corollaries**:
- Changed a decision? **Edit the body directly.** Do not annotate "(was <old value>)" beside it.
- A conclusion overturned? **Delete the old conclusion from the body** and put
  "was <old conclusion> / now <new conclusion> / what overturned it" at the end.
- No `~~strikethrough~~`, `[deprecated]`, `(superseded by <the later decision>)` in the body.
- Same for code comments: write "why it is this way now", not "how it used to be".
  **Pitfall comments in checking code are the exception** ("this bypass measurably
  got through, hence this check"). The criterion is what it points at:
  pointing at **why this code looks the way it does now** → keep;
  pointing at **a version that no longer exists** → delete.

`CLAUDE.md` and `rules/*.md` follow a different discipline; see `rules-discipline.md`.

History statements in body text, and where the history section sits, are enforced by
`scripts/doc-lint.sh`, which scans only `.md`; the clause on code comments rests on review.
