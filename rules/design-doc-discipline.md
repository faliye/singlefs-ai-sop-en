<!-- generated-from: rules/design-doc-discipline.md sha256:11a5769b9d91edff33d7ba418ee08693619a1ddbabb985f3494675466a564361 -->
<!-- doc-lint:rule-definition -->
# Design document discipline

**Applies to**: README, design notes, `records/` — **things humans read through.**

Narrative, argument and context are its job; it does not have to give way to
machines. Being good to read is the goal here, not a compromise. Knowledge
documents have their own set of rules: see `kb-discipline.md`.

## 1. Body text states only the current state; history goes to the end

No historical statements in the body — no "it used to be X, now it is Y", no "it was
once called A". History that must be kept goes into a "Revision history" section at
the end:

```markdown
## Revision history

### YYYY-MM-DD
- Was X / now Y / basis for the change: Z
```

**Why**: history mixed into the body means a reader cannot tell at a glance which
value is current. After a few rounds, even "what is it now" needs archaeology to
answer — and the archaeology is often wrong. This project runs for years; that cost
compounds.

**Corollaries**:
- Changed a decision? **Edit the body directly.** Do not annotate "(was X)" beside it.
- A conclusion overturned? **Delete the old conclusion from the body** and put
  "was X / now Y / what overturned it" at the end.
- No `~~strikethrough~~`, `[deprecated]`, `(superseded by XX)` in the body.
- Same for code comments: write "why it is this way now", not "how it used to be".

`CLAUDE.md` and `rules/*.md` **keep no revision history section at all**; their
history goes to `CHANGELOG.md` — they are read in full at the start of every session,
and history dilutes them.

Enforced by `scripts/doc-lint.sh`.

## 2. Length must match the weight of the change

However solid the argument, that is not a reason to write all of it out. The
paragraph you feel is "load-bearing" is often exactly the one to cut.
Details in `writing-economy.md`.

## 3. Where persuasion is needed, persuade properly

A design document answers "why should it be done this way" and "how do I get
started". Both need setup, examples, and the trade-offs spelled out — **none of that
is redundancy.**

Criterion: with this paragraph removed, would the reader still agree? Could they
still get started? Yes → it can go. No → it is load-bearing.
