---
name: crash-test
description: Run singlefs's verification suite — crash-point replay, model-based differential testing. Use it to judge whether a write path is correct.
---
<!-- generated-from: skills/crash-test/SKILL.md sha256:61d9bae9646e4b1ceccf3ae8df7d9a1b11533caca8b9025d5cb3e6e8c1f10136 -->

# The verification suite

Rules live in `rules/test-discipline.md` and `rules/show-me-test.md`.

## What each method covers

| Method | What it verifies | Provided by |
|---|---|---|
| Crash-point replay | Whether any power-cut point recovers | The project |
| Model-based differential | Functional correctness: is the result of an operation sequence right | The project |

Crash-point replay needs the thing under test's own write stream and checker, and model-based
differential testing needs its ideal model; the shared gate can carry neither. When the project wires one of them in under `.claude/gate.d/`, that stage's
header states which item it covers:

```bash
# gate-stage: layer-0 crash-point replay (every crash state of the first transaction)
# gate-covers: 崩溃点重放
```

The only keys are the ones on `gate.sh`'s unimplemented list: `模型对拍` (model-based differential),
`崩溃点重放` (crash-point replay), `最终判据` (final criterion) and `命名纪律（shell）` (naming discipline for shell scripts), copied
literally; one wrong character turns it red. Only when that stage ran and passed this round does the
unimplemented list at the end of `gate.sh` move the item under "covered by project-local stages".
Coverage says only "some stage is doing this, and it passed this time"; how far it reaches is in that
stage's own name and description. A stage with nothing to judge this round exits 77: it is recorded
as "not run this time" — not as a pass, and not as coverage.

## Why crash-point replay cannot be substituted

**Green unit tests, a passing differential model, and a silent checker do not add up to
crash-consistency evidence.**

Those verify "is the state right on the normal path". Crash consistency asks something
else: **cut power after an arbitrary write request — can it still recover?** Only
trying every crash point answers that.

Implementing it needs, in dependency order: the first on-disk format (see the on-disk
format entries in the project's `kb/decisions.md`) → mkfs + checker → the transaction
commit path → block-layer write logging (`dm-log-writes`).

## Reading results

- **"Did not reproduce" is not "no problem".** To claim crash consistency, say how
  many crash points this round enumerated and whether that was all of them.
- **A silent checker may just mean the check is not implemented.** Look at the status
  column in `kb/invariants.md` first.
- **An unreadable verdict voids the round**, never counts as a pass. The same goes for the
  project's apparatus when it cannot read its end marker.

Concurrent memory ordering, and real workloads and crash injection inside a VM, are bound even more
tightly to the thing under test; the project decides, tests and verifies them itself, and they are
not part of this shared skill.
