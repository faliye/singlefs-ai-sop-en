<!-- generated-from: CLAUDE.md sha256:486d6196426c2e69eb62b59bc7a69bfbf427d082ea5eb3a10a7c27cfe6bd64e8 -->
<!-- doc-lint:rule-definition -->
# singlefs-ai-sop-en

**Contributor governance rules and gate tooling (Contributor Governance).**
It governs **how a project collaborates with AI**, not how a filesystem should be
designed. singlefs is its only user.
Work inside this repository is bound by these rules too.

Changing the spec proper **must bump `VERSION` in the same change**; otherwise
the project's gate will report a version mismatch. **Which paths count as "the spec
proper" is defined by `GOVERNED` in `scripts/version-discipline.sh`** — that is the
only list, and it is not copied here.

## Language of collaboration

**When working in this repository, conduct the conversation and produce all output in English.**
Work in the language of the repository you installed; the reasoning and the list of
editions live in [README](README.md) under "Pick the edition in your own language",
not here.

**Where editions read differently, the language does not decide — the actual
behaviour of `scripts/` does.**

**Changing a rule means changing every published edition and merging them together**;
the person making the change is the one who guarantees they agree. The gate can only
see that a hash does not match — not whether the editions say the same thing.

**What gets translated and what gets copied verbatim** turns on one question: is there
prose written for people in it? The two lists live in `scripts/manifest.sh`
(`translated_paths` and `not_translated_re`). Anything in neither is stopped by the
coverage check.

## Rules (always in force)

@rules/engineering-philosophy.md
@rules/sop-first.md
@rules/show-me-test.md
@rules/machine-first.md
@rules/doc-discipline.md
@rules/design-doc-discipline.md
@rules/kb-discipline.md
@rules/test-discipline.md
@rules/evidence-discipline.md
@rules/verify-before-claiming.md
@rules/command-safety.md
@rules/writing-economy.md
@rules/writing-style.md
@rules/session-wrapup.md

## Where a thing belongs

**This SOP was made for singlefs. singlefs is its only user, and nothing here
presumes it generalises to other projects.** So the criterion is not "would another
project need it too" — there is no other project to look at, so anyone can answer
"yes", and everything ends up upstream.

To decide where a piece of content goes, look at what it governs:

- **Collaboration** — what evidence a submission carries, how documents are written,
  where decisions are recorded, what next step a gate rejection must give.
  → this repository. Methodology into `rules/`, procedure into `skills/`,
  anything runnable into `scripts/`.
- **How the filesystem is designed** — transactions, crash consistency, on-disk
  format, the disciplines specific to this kind of system.
  → the project. Design decisions into `kb/decisions.md`, invariants into
  `kb/invariants.md`, **project-specific rules into `.claude/rules/`**, and
  project-specific scripts into `.claude/scripts/`.

**When in doubt, keep it in the project.** A rule that should not have gone upstream
has to be worked around every round from then on; one misplaced in the project is a
single file to fix.

## How a project hooks in

The project does not copy this repository's content. Each of the three layers has its
own hook-up (**no symlinks**):

| Layer | How it hooks in |
|---|---|
| rules | The project's `CLAUDE.md` references `@.claude/singlefs-ai-sop/rules/x.md`; no copy is kept in the project |
| Project-local rules | Put them in `.claude/rules/x.md` and reference them as `@.claude/rules/x.md`. They do not go upstream |
| skills | The project's `.claude/skills/<name>/SKILL.md` is a **stub**: frontmatter plus a pointer to the shared body |
| scripts | The project's `.claude/scripts/x.sh` is a **wrapper**: set up the environment, then `exec` the shared script |

Never put substance or logic inside a stub or a wrapper — the substance should exist
in exactly one place; put it there and a second copy exists, and two copies will
disagree sooner or later.
