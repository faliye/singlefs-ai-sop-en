<!-- generated-from: templates/CLAUDE.project.md sha256:aaa2f5201bce0c5efceaae3c2728171bd7118240f89be1257b2b9856c9188914 -->
# <project name>

<Three to five lines: what this project is, which milestone it is at, how it relates to
the main line. No longer.>

## Rules (always in force)

@.claude/singlefs-ai-sop/rules/sop-first.md
@.claude/singlefs-ai-sop/rules/show-me-test.md
@.claude/singlefs-ai-sop/rules/writing-discipline.md
@.claude/singlefs-ai-sop/rules/design-doc-discipline.md
@.claude/singlefs-ai-sop/rules/communication-discipline.md
@.claude/singlefs-ai-sop/rules/kb-discipline.md
@.claude/singlefs-ai-sop/rules/rules-discipline.md
@.claude/singlefs-ai-sop/rules/command-safety.md
@.claude/singlefs-ai-sop/rules/pushback-discipline.md

Two rules are not `@`-loaded permanently: read each in full once before writing code or scripts, then look it up by name: `.claude/singlefs-ai-sop/rules/code-discipline.md`, `.claude/singlefs-ai-sop/rules/preflight-discipline.md`.
<!-- doc-lint:read-once code-discipline.md preflight-discipline.md -->

**Rules specific to the thing under test** (its own design discipline, its verification methods, that family) go in `.claude/rules/` and are `@`-referenced here as well. They
are not upstreamed — the shared SOP holds collaboration rules only.

(The `@.claude/singlefs-ai-sop/...` entries above are the shared rules distributed by
[singlefs-ai-sop](.claude/singlefs-ai-sop/README.md). Change them upstream and bump
`VERSION`; never edit them in place inside a project.)

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

The gate does not sort submissions by where they
came from; it sorts them into those carrying evidence and those not.

```bash
bash .claude/scripts/gate.sh          # the acceptance gate; mandatory before submitting

bash .claude/scripts/check.sh         # fast feedback (format/lint/build/unit tests)
bash .claude/scripts/doc-lint.sh      # documentation discipline check
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
