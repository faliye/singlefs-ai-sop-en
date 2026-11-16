<!-- generated-from: rules/design-doc-discipline.md sha256:d2e707023a70c72a8b9a3016ea7c0668b6891606317f28fae23dda50007de156 -->
<!-- doc-lint:rule-definition -->
# Design document discipline

**Applies to**: README, design notes, `records/` — **things humans read through.**
Knowledge documents for model retrieval follow `kb-discipline.md`; `CLAUDE.md` and `rules/*.md` follow `rules-discipline.md`.

## 1. Body text states only the current state; history goes to the end

No historical statements in the body — no "it used to be <old approach>, now it is <new approach>", no "it was
once called <old name>". History that must be kept goes into a "Revision history" section at
the end:

```markdown
## Revision history

### YYYY-MM-DD
- Was <old approach> / now <new approach> / basis for the change: <why it changed>
```

- Changed a decision? **Edit the body directly.** Do not annotate "(was <old value>)" beside it.
- A conclusion overturned? **Delete the old conclusion from the body** and put
  "was <old conclusion> / now <new conclusion> / what overturned it" at the end.
- No `~~strikethrough~~`, `[deprecated]`, `(superseded by <the later decision>)` in the body.
- Same for code comments: write "why it is this way now", not "how it used to be".
  Pitfall comments in checking code ("this bypass measurably got through, hence this check") are the exception: one pointing at why this code looks the way it does now is kept; one pointing at a version that no longer exists is deleted.

History statements in body text, and where the history section sits, are enforced by
`scripts/doc-lint.sh`, which scans only `.md`; the clause on code comments rests on review.
