<!-- generated-from: rules/doc-discipline.md sha256:0564dc04d2c7287b8899b7053f1b760cea1ec656feafd8f53c623c221e2c02f4 -->
<!-- doc-lint:rule-definition -->
# Documentation discipline: first work out who this document is for

**Mixing them serves neither.** Three kinds of document, three ways of writing:

| Kind | Who uses it | Goal | Rule |
|---|---|---|---|
| **Design docs** | humans read them through | make people **agree** and **get started** | `design-doc-discipline.md` |
| **Engineering kb** | model retrieval | keep the model from **making things up** | `kb-discipline.md` |
| **Rules** | execution | must be turnable into a check that fails | this directory, see `sop-first.md` |

Before writing, ask: will this be read start to finish by a person, or pulled out
one item at a time by a model? Different answers, different writing.

**The one rule common to all three**: body text states only the current state;
history goes to the end. But the *reason* differs for each kind, and is stated in
each rule.
