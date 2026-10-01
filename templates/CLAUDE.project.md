<!-- generated-from: templates/CLAUDE.project.md sha256:b6a339375b62754a6b736b0802348250e8468767e48ed62d53ebe227e7ea6371 -->
# <project name>

<Three to five lines: what this project is, which milestone it is at, how it relates to
the main line. No longer.>

## Rules (always in force)

@.claude/singlefs-ai-sop/rules/sop-first.md
@.claude/singlefs-ai-sop/rules/show-me-test.md
@.claude/singlefs-ai-sop/rules/code-discipline.md
@.claude/singlefs-ai-sop/rules/writing-discipline.md
@.claude/singlefs-ai-sop/rules/kb-discipline.md
@.claude/singlefs-ai-sop/rules/rules-discipline.md
@.claude/singlefs-ai-sop/rules/test-discipline.md
@.claude/singlefs-ai-sop/rules/test-script-discipline.md
@.claude/singlefs-ai-sop/rules/evidence-discipline.md
@.claude/singlefs-ai-sop/rules/verify-before-claiming.md
@.claude/singlefs-ai-sop/rules/command-safety.md
@.claude/singlefs-ai-sop/rules/preflight-discipline.md
@.claude/singlefs-ai-sop/rules/session-wrapup.md

The four philosophy rules are not `@`-loaded permanently: read each in full once at the start of a session, then look it up by name: `.claude/singlefs-ai-sop/rules/engineering-philosophy.md`, `.claude/singlefs-ai-sop/rules/machine-first.md`, `.claude/singlefs-ai-sop/rules/design-doc-discipline.md`, `.claude/singlefs-ai-sop/rules/pushback-discipline.md`.
<!-- doc-lint:read-once engineering-philosophy.md machine-first.md design-doc-discipline.md pushback-discipline.md -->

**Rules specific to the thing under test** (its own design discipline, its verification methods, that family) go in `.claude/rules/` and are `@`-referenced here as well. They
are not upstreamed — the shared SOP holds collaboration rules only.

(The `@.claude/singlefs-ai-sop/...` entries above are the shared rules distributed by
[singlefs-ai-sop](.claude/singlefs-ai-sop/README.md). **Changing one changes the floor
under every contributor** — change it upstream and bump `VERSION`; never edit it in
place inside a project.)

## Project-local facts

| File | Contents |
|---|---|
| `.claude/kb/decisions.md` | Design decisions: what is settled, why, what is not |
| `.claude/kb/experiments.md` | Experiments: the question, criteria fixed in advance, controls and mutations, the rerun command |
| `.claude/kb/invariants.md` | The invariant list; the project's checks are its executable form |
| `.claude/kb/prior-art.md` | Research into other implementations, with sources and measurement bases |
| `.claude/kb/pitfalls.md` | The pitfall list; come back to it for every design decision |
| `.claude/kb/checks-owed.md` | Debt table: checks we know we want but cannot enforce yet, with prerequisites |
| `records/` | How it was built |

## The gate

The gate exists to **raise every submission to the line where it is worth a person's
time to review**, not to keep anyone out. It does not sort submissions by where they
came from; it sorts them into those carrying evidence and those not.

```bash
bash .claude/scripts/gate.sh          # the acceptance gate; mandatory before submitting

bash .claude/scripts/check.sh         # fast feedback (format/lint/build/unit tests)
bash .claude/scripts/gate-lint.sh     # the gate itself: does every rejection give a next step
bash .claude/scripts/shell-lint.sh    # shell discipline: pkill -f / pgrep -f, values carried out of subshells, git undo commands, unguarded rm -rf, wait with no arguments, pipelines ending in grep -q under pipefail
bash .claude/scripts/naming-lint.sh   # naming discipline: single-letter names and common abbreviations in .rs
bash .claude/scripts/env.sh           # environment check
```

**Gate proves evidence requirements, not semantic correctness.**
Green means the evidence requirements are met, not that the semantics are right —
`gate.sh` lists the unimplemented stages every time.

## What is particular about this project

<One to three items; only what changes how you work. Put the detailed criteria in kb and leave
only a signpost here.>

## The one-paragraph version

<Three or four sentences, each one criterion this project most easily gets wrong. Do not
copy general discipline in here; it lives in rules.>
