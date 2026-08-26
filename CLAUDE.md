<!-- doc-lint:rule-definition -->
# singlefs-ai-sop-en

**Contributor governance rules and gate tooling (Contributor Governance).**
It governs **how a project collaborates with AI**, not how any particular kind of
system should be designed.
Work inside this repository is bound by these rules too.

Changing `rules/` or `scripts/` **must bump `VERSION` in the same change**;
otherwise every project's gate will report a version mismatch.

## Language of collaboration

**When working in this repository, conduct the conversation and produce all output in English.**

**Developers pick the edition in their own language.** Chinese, English and Japanese
exist today, more are in progress, and English is the default. Work in the language of
the repository you installed — someone working in English never has to read Chinese.
**The AI does not have to translate a second time either**; one fewer restatement is
one less loss of information.

The rules must say the same thing in every edition. Where they read differently, the
language does not decide — the actual behaviour of `scripts/` does.

**Changing a rule means changing every published edition and merging them together**;
the person making the change is the one who guarantees they agree. The gate can only
see that a hash does not match — not whether the editions say the same thing.

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
@rules/session-wrapup.md

## Where a thing belongs

To decide where a piece of content goes, ask one question: **would another project need it too?**

- Yes → this repository. Methodology into `rules/`, procedure into `skills/`,
  anything runnable into `scripts/`.
- No → the project. Design decisions into `kb/decisions.md`, invariants into
  `kb/invariants.md`, **project-specific rules into `.claude/rules/`**, and
  project-specific scripts into `.claude/scripts/`.

"Another project" means **any** project using this SOP, not "another project of the
same kind". A discipline only a filesystem needs belongs to that project, however
well written it is.

The other direction: if you find a passage in a project's files that **another project
has copied out as well**, this repository is missing a rule — bring it up here.
Do not let it be copied a third time.

## How a project hooks in

Projects do not copy this repository's content. Each of the three layers has its own
hook-up (**no symlinks**):

| Layer | How it hooks in |
|---|---|
| rules | The project's `CLAUDE.md` references `@.claude/singlefs-ai-sop/rules/x.md`; no copy is kept in the project |
| Project-local rules | Put them in `.claude/rules/x.md` and reference them as `@.claude/rules/x.md`. They do not go upstream |
| skills | The project's `.claude/skills/<name>/SKILL.md` is a **stub**: frontmatter plus a pointer to the shared body |
| scripts | The project's `.claude/scripts/x.sh` is a **wrapper**: set up the environment, then `exec` the shared script |

Never put substance or logic inside a stub or a wrapper — written there, no other
project can see it, and it will be copied out again next time.
