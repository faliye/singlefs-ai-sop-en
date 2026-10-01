<!-- generated-from: rules/rules-discipline.md sha256:da9396081f8c1be1fb32197439ec6e63e9ed3d1096113f89cdf143b27b11493f -->
<!-- doc-lint:rule-definition -->
# Rule-file discipline

**Scope**: `rules/*.md` and a project's own `.claude/rules/*.md` — the files that are **executed**.

Design documents follow `design-doc-discipline.md`; knowledge written for model retrieval follows
`kb-discipline.md`. Agent definitions and skill bodies follow this same discipline, each with its own check.

## 1. The body carries four things only

| Write | Do not write |
|---|---|
| How to do it: steps, order, criteria, thresholds | Why it was decided this way: argument, trade-off, derivation |
| What to do and what not to do: scope and exceptions | Where it came from: measurements, dates, who decided, potholes hit |
| What to do by default when unsure | What it used to be: old values, old practice, how it changed |
| Which rule lives where: pointers to other rules, scripts or kb | How the gate's discriminating power was demonstrated, and the numbers |

## 2. Keep the criterion, move the argument out

| | What it looks like | Where it goes |
|---|---|---|
| **Criterion** | You can judge from it whether to do something, and when it is done | **Stays in the body** |
| **Argument** | It explains why that criterion holds | **Deleted** |

- "Delete this sentence — what does the reader now not know? Cannot say? Delete it." — criterion, stays.
- "The cost of verbosity is not space; it is that the useful sentence gets buried." — argument, deleted.

When one paragraph holds both, keep the criterion sentence in the body and delete the rest whole.
Do not leave a summary behind.

## 3. What moves out is deleted

**No new home for it, and no copy written somewhere else.** A shared rule's history goes where it
always did — `CHANGELOG.md`, one section per version; that is the only destination.
If a deleted argument is needed again, argue it again.

**Once moved, leave nothing behind**: no "(was X)", no "this used to be…" in the body.

## 4. A link to history must carry a deterrent

When the body points at history in `CHANGELOG.md`, `records/` or kb, put the deterrent **on the same line**:

> History and evidence: the 0.0.53 section of `CHANGELOG.md`. **Do not read it unless you are tracing where this came from.**

Write the deterrent as "do not read" — not one word of it may be dropped. A link without it is not allowed.

## 5. Rule files keep no history section

`CLAUDE.md` and `rules/*.md` keep no `## Revision history` section, not even at the end of the file.
Shared rules record their history in `CHANGELOG.md`, one section per version.

## 6. Before adding a rule, ask whether it can become a check that fails

The criterion and the how-to are in the "Boundary" section of `sop-first.md`.

## 7. No positional references in the body, and no self-reference

Never write anything that points at a position in the document: "the previous section",
"the table below", "mentioned earlier", "the above", "as follows:", "ditto", "see the section below",
nor self-references such as "this clause", "this section", "this table", "this document".

Write the position as a name: a section becomes its heading, a table becomes the thing it judges,
a fact elsewhere becomes a link to the file it lives in.

The scope is `rules/*.md`, `CLAUDE.md`, `agents/*.md` and `skills/*/SKILL.md`. For positional
references the criterion is the same one `kb/*.md` is held to, with the per-word scope in
`kb-discipline.md` clause 1.

Self-reference is judged only for the words that name a document structure: "this section", "this
chapter", "this table", "this document", "this clause". Domain objects — "this experiment", "this
decision", "the present experiment" — are **not** judged, and neither are references to time ("this round",
"the current round"). Both remain judged in `kb/*.md`.

When a rule file carries the `<!-- doc-lint:rule-definition -->` marker, examples inside backticks
and 「」 are exempt; the body itself is still judged.

Enforced by `scripts/doc-lint.sh`.

## 8. The body carries nothing specific to one consumer

The body may not name a project that uses these rules, nor its paths,
nor its file names, and may not use its decision or experiment numbers as examples. Write examples in a
form that names no one: a wiring path like `.claude/gate.d/` is a shared convention and may be written;
a specific `.claude/gate.d/55-xxx.sh` may not.

The gate does not judge this clause; it is left to review.

## What the gate covers

`scripts/rules-lint.sh` judges the part of clauses 1, 3, 4 and 5 that can be reduced to literal patterns:
record sections, argument sections, dated lines, explanatory paragraphs and half-sentences, lexical explanations, and history
links with no deterrent. It scans this package's `rules/`, and also the project's `.claude/rules/` (or `.claude/agents/`
when that directory does not exist): the latter is started by the `gate.sh` stage "Rule discipline (project-local)"
(`规则纪律（项目本地）`), so the project wires up nothing of its own. Files not yet swept are registered
one per line in the project root's `.claude/rules-lint-exclude`; delete a line once that file is done —
the exclusion list only shrinks.

Clause 7 is judged by `scripts/doc-lint.sh`, not by `rules-lint`; clause 8 is not judged by the gate.

**What it cannot cover**: the line in clause 2 (which sentence is criterion and which is argument), and
whether the link under the deterrent points at the right place. Those rest on a person reading them aloud, and on review.

The criteria are chosen by this package's language. Chinese judges all seven; English and Japanese judge record sections, argument sections, dated lines, and links without a deterrent,
and recognise explanatory paragraphs and half-sentences only in the "label word plus colon" form (`Observed:`, `実測：` and the like); those opening with a conjunction, and lexical explanations, are reported explicitly as **not implemented** in these two languages,
rather than passing silently (`show-me-test.md`: the gate must not pretend to pass).
