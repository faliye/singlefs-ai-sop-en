<!-- generated-from: rules/rules-discipline.md sha256:abfcf662b1980cc51f74a6408c15f8c8afda61380b42573723be4e29514eb741 -->
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

Rules are read whole into the context of every round of work. With argument and history mixed in,
whoever executes them — person or model — must first sort out which sentence is the order and which
is the background; sort it wrong and they act on the background.

## 2. Keep the criterion, move the argument out

These two are the easiest to confuse. One line separates them:

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
always did — `CHANGELOG.md`, one section per version. That is the only destination.
If a deleted argument is needed again, argue it again.

**Once moved, leave nothing behind**: no "(was X)", no "this used to be…" in the body.

## 4. A link to history must carry a deterrent

When the body points at history in `CHANGELOG.md`, `records/` or kb, put the deterrent **on the same line**:

> History and evidence: the 0.0.53 section of `CHANGELOG.md`. **Do not read it unless you are tracing where this came from.**

Write the deterrent as "do not read" — not one word of it may be dropped. A link without it is not allowed:
readers and models follow a link by default, and following it loads the very thing you just moved out
back into the context.

## 5. Rule files keep no history section

`CLAUDE.md` and `rules/*.md` keep no `## History` section, not even at the end of the file.
Shared rules record their history in `CHANGELOG.md`, one section per version.

## 6. Before adding a rule, ask whether it can become a check that fails

The criterion and the how-to are in the "Boundary" section of `sop-first.md`; not repeated here.

## What the gate covers

`scripts/rules-lint.sh` judges the part of clauses 1, 3, 4 and 5 that can be reduced to literal patterns —
record sections, dated lines, explanatory paragraphs and half-sentences, lexical explanations, and history
links with no deterrent. It scans this package's `rules/`; a project hands its own rules directory over with
`RULES_LINT_DIR`, and the shared `gate.sh` runs that stage automatically. Files not yet swept are registered
one per line in the project root's `.claude/rules-lint-exclude`; delete a line once that file is done —
the exclusion list only shrinks.

**What it cannot cover**: the line in clause 2 (which sentence is criterion and which is argument), and
whether the link under the deterrent points at the right place. Those are
semantic judgements — a person reads them aloud, and review catches the rest.

⚠️ Its patterns are Chinese. In a repository of another language this check reports **not implemented**
rather than passing silently (`show-me-test.md`: the gate must not pretend to pass).
