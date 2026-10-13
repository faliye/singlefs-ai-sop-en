<!-- generated-from: agents/INDEX.md sha256:15ba3972baf020b2904c501b950dd8c2f9bfd34c0dc0503fd876298003588cf0 -->
# Shared subagent definitions

**This layer is governed; it is currently empty** (`rules/kb-discipline.md` section 3, "Record "we do not know" explicitly").

## What belongs here

**Subagents that get delegated repeatedly across rounds of work, with fixed criteria.**
The line against `skills/`:

| What work | Where |
|---|---|
| A procedure a person or the main model reads and follows, with no separate context | `skills/` |
| Work that needs its own context and returns only a conclusion | `agents/` |
| A delegation that only makes sense in one project, or one that governs how the system under test is designed | the project's `.claude/agents/` |

**When in doubt, keep it project-local.**

## How to write one

One subagent per file, `agents/<name>.md`, YAML frontmatter first (it must start on
line 1):

```markdown
---
name: <lower-case-hyphenated, identical to the filename>
description: <when to delegate it: state the trigger>
---

<Body: its remit, its criteria, and the **format of what it returns**.>
```

**`description` states the trigger**: write "after a mutation run, decides which uncaught
mutants are equivalent mutants", not "reviews code".

## Three disciplines

1. **Pin down the output format.** Say what it returns, in what order, and which pieces
   of evidence each item carries.
2. **Report only what was run, never what was inferred.** The report carries its own
   evidence (which command, which file, what output) — the same rule as
   `rules/evidence-discipline.md`, applied to delegation.
3. **It has to be able to say what it did not do.** What could not run, what was not
   verified, listed explicitly (`rules/show-me-test.md`: unimplemented stages stay
   listed).

## How it is wired in

Same shape as `skills/`: the project gets a **stub**, the body exists in exactly one
place.

| Layer | Wiring |
|---|---|
| Shared definition | this directory, `agents/<name>.md` |
| Project side | `.claude/agents/<name>.md` is a stub: frontmatter plus a pointer to the shared body |

`install.sh` generates the stub. Nothing of the body goes in it.

## Gate

- While the directory is empty, `scripts/manifest.sh` says so explicitly rather than
  passing silently.
- Every `agents/*.md` must carry `name` and `description`, with `name` matching the
  filename.
- Same rule as `rules/`: in the manifest, translated per file, traced per file. Changing
  one bumps `VERSION`.
