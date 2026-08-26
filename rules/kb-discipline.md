<!-- generated-from: rules/kb-discipline.md sha256:58ce193023eca2b75952994b5357abc40e9370e78e7d4e01894bfae420970b5f -->
<!-- doc-lint:rule-definition -->
# Knowledge document discipline

**Applies to**: `kb/*.md` — **things models retrieve from.**

**The kb is not reading material.** With a model at hand, nobody reads the kb start
to finish; they have the model retrieve, extract and summarise. So the kb is
designed for **being pulled out one item at a time**, not for being read through.

**Its one goal is to keep the model from making things up.** Being pleasant to read
is incidental, not the objective.

## 1. Every fact stands on its own

A fact must still hold when retrieved alone, without the paragraph above it.

**No dangling references** — "as stated above", "same as above", "see above",
"mentioned earlier", "the aforementioned", "as described below". They are harmless
when read through and break on the spot when retrieved; and **a model will not say
"I do not follow" — it will fill in something.**

Enforced by `scripts/doc-lint.sh`.

## 2. Every entry carries its source and status

Source, date, measured or inferred, on what measurement basis.

This is not pedantry: **it is the only cue a model has for telling "this is
established" from "this was a guess at the time".** Without the cue, the two look
identical in a retrieval result.

- Numbers must carry their basis: blocks or bytes, metadata included or not, on what
  hardware under what workload.
- Conclusions read elsewhere carry their source and the note "not verified in this
  project".
- **When you cite someone else's measurement, write down what it does and does not prove.**
  Provenance is not enough: provenance answers "where did this number come from",
  while whoever retrieves it actually needs to know "does this number apply to me".
  There is a layer of extrapolation between the two, and **the person writing it down
  must do that extrapolation and record it** — leave it to the next reader and they
  will most likely skip it and use the number as is.
- **All historical data is reference only.** Every measurement is bound to the build
  it was taken on; to use it in support of a new conclusion, re-run it first and
  confirm it still holds today.

## 3. Record "we do not know" explicitly

Anything investigated without a conclusion, anything unverified, anything
deliberately set aside — write it down and mark it as such.

**The way a model fills a gap is by inventing.**
A gap is more dangerous than an error — an error can be argued with; a gap gives you
nothing to argue with.

## 4. A contradiction is worse than a gap

A given fact gets **exactly one** authoritative record; everywhere else links to it.

A person reading through notices when two passages disagree. **Retrieval will not
serve up both — it picks one, and does not tell you it picked.**

## 5. Tables beat prose

Filterable, alignable, and every row stands alone.

## 6. Do not optimise for reading order

The kb has no "read it from the top" use case. No setup, no transitions, no
conclusions to round things off.

## 7. Body text states only the current state; history goes to a "Revision history" at the end

The rule is the same as for design documents, but the **reason differs, and is
harder**: retrieval serves the stale entry up **on its own**, with no context and
nothing to compare against; a model has no "I remember this changed last week" and
takes it all at face value.

Every kb document must close with a "## Revision history" section — keep the section
even with no history yet, for later.
Enforced by `scripts/doc-lint.sh`.
