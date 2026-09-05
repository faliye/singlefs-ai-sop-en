<!-- generated-from: agents/INDEX.md sha256:4ae652fd692ec2d89f4d71c38b7b2b79b924f43047f178013253bd484fc3ab5c -->
# Shared subagent definitions

**This layer is governed; it is currently empty.** Empty is a state, not an oversight —
writing it down explicitly, because "there is none" and "we forgot" look identical in a
directory listing (`rules/kb-discipline.md`: a blank is more dangerous than an error).

## What belongs here

**Subagents that get delegated repeatedly across rounds of work, with fixed criteria.**
The line against `skills/`:

| | Where | Why |
|---|---|---|
| A procedure a person or the main model follows | `skills/` | It is **read**; it needs no separate context |
| Work that needs its own context and returns only a conclusion | `agents/` | It is **delegated**; the main line should not drown in its intermediate output |
| A delegation that only makes sense in one project | the project's `.claude/agents/` | Not upstreamed (the division criterion in `CLAUDE.md`) |

**When in doubt, keep it project-local.** A subagent that should not have been
upstreamed has to be worked around in every round that follows.

## How to write one

One subagent per file, `agents/<name>.md`, YAML frontmatter first (it must start on
line 1):

```markdown
---
name: <lower-case-hyphenated, identical to the filename>
description: <when to delegate it. This sentence is the main model's only basis for
             picking it, so state the trigger>
---

<Body: its remit, its criteria, and the **format of what it returns**.>
```

**`description` decides whether it gets used correctly.** "Reviews code" tells nobody
when to reach for it; "after a mutation run, decides which blind spots are equivalent
mutants" does.

## Three disciplines

1. **Pin down the output format.** A subagent that comes back with a wall of prose has
   pushed the sorting cost onto the main line. Say what it returns, in what order, and
   which pieces of evidence each item carries.
2. **Report only what was run, never what was inferred.** Nobody watches the middle of
   delegated work, so its report must carry its own evidence (which command, which
   file, what output) — the same rule as `rules/evidence-discipline.md`, applied to
   delegation.
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

`install.sh` generates the stub. Nothing of the body goes in it — writing it there
creates a second copy, and two copies eventually say different things.

## Gate

- While the directory is empty, `scripts/manifest.sh` says so explicitly rather than
  passing silently.
- Every `agents/*.md` must carry `name` and `description`, with `name` matching the
  filename.
- Same rule as `rules/`: in the manifest, translated per file, traced per file. Changing
  one bumps `VERSION`.
