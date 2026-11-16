<!-- generated-from: rules/rules-discipline.md sha256:15ec1218c8d96e9f0c698967b0316c617c0778b4343f4c406b400d4af41b30b3 -->
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

What lets you judge whether to do something, and when it is done, is a **criterion** and stays in the body; what explains why that criterion holds is an **argument** and is deleted.

- "Delete this sentence — what does the reader now not know? Cannot say? Delete it." is a criterion.
- "The cost of verbosity is not space; it is that the useful sentence gets buried." is an argument.

When one paragraph holds both, keep the criterion sentence and delete the rest whole.
Do not leave a summary behind.

## 3. What moves out is deleted

No new home for it, and no copy written somewhere else; the body keeps no trace such as "(was <old value>)" or "this used to be…".
A shared rule's history goes only into `CHANGELOG.md`, one section per version. If a deleted argument is needed again, argue it again.

## 4. A link to history must carry a deterrent

When the body points at history in `CHANGELOG.md`, `records/` or kb, put the deterrent **on the same line**:

> History and evidence: the 0.0.53 section of `CHANGELOG.md`. **Do not read it unless you are tracing where this came from.**

Write the deterrent as "do not read" — not one word of it may be dropped.

## 5. Rule files keep no history section

`CLAUDE.md` and `rules/*.md` keep no `## Revision history` section, not even at the end of the file.

## 6. Before adding a rule, ask whether it can become a check that fails

The criterion and the how-to are in the "Boundary" section of `sop-first.md`.

## 7. No positional references in the body, and no self-reference

The scope is `rules/*.md`, `CLAUDE.md`, `agents/*.md` and `skills/*/SKILL.md`.
The criterion and the fix are those of `kb-discipline.md` clause 1 "Every fact stands on its own", with two differences:

- Self-reference is judged only for the words that name a document structure: "this section", "this chapter", "this table", "this document", "this clause". Domain objects such as "the present experiment" and "this decision" are not judged.
- References to time ("this round", "the current round") are not judged.

A fact elsewhere becomes a link to the file it lives in.

In a file that carries the `<!-- doc-lint:rule-definition -->` marker, examples inside backticks
and 「」 are exempt; the body itself is still judged.

## 8. The body carries nothing specific to one consumer

The body may not name a project that uses these rules, nor its paths,
nor its file names, and may not use its decision or experiment numbers as examples. Write examples in a
form that names no one: a wiring path like `.claude/gate.d/` may be written;
a specific numbered file name under it may not.

## What the gate covers

| Clause | Who judges |
|---|---|
| The part of clauses 1, 3, 4 and 5 that can be reduced to literal patterns | `scripts/rules-lint.sh`: record sections, argument sections, dated lines, explanatory paragraphs and half-sentences, lexical explanations, and history links with no deterrent |
| Clause 7 | `scripts/doc-lint.sh` |
| Clause 8 | `scripts/rules-lint.sh`, for two kinds only: the name of a project that uses these rules, and a specific numbered file name in the gate directory (`gate.d/<digits>-`); decision numbers, experiment numbers and other paths it cannot recognise; it judges only this package's own rules |
| The line in clause 2, and whether the link under the deterrent points at the right place | review |

`rules-lint.sh` scans this package's `rules/`, and also the project's `.claude/rules/` (or `.claude/agents/`
when that directory does not exist), the project `CLAUDE.md`, agent definitions and skill bodies; the latter is started by the `gate.sh` stage "Rule discipline (project-local)"
(`规则纪律（项目本地）`).
Files not yet swept are registered one per line in the project root's `.claude/rules-lint-exclude`; delete a line once that file is done —
the exclusion list only shrinks.
The criteria are chosen by this package's language; what is not implemented it reports explicitly as unimplemented.
