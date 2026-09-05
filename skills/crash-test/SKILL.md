---
name: crash-test
description: Run singlefs's verification suite — LKMM memory ordering, QEMU/KVM stress, crash-point replay, model-based differential testing. Use it to judge whether a write path is correct, or when a concurrency change needs verification.
---
<!-- generated-from: skills/crash-test/SKILL.md sha256:03baaac78f5de9508b3473142cef72d25fdd23bd8138713225d7a659d955f9f8 -->

# The verification suite

Rules live in `rules/test-discipline.md` and `rules/show-me-test.md`.

## What the four methods each cover

| Method | What it verifies | State |
|---|---|---|
| **LKMM (herd7)** | Memory ordering on concurrent paths: lock-free structures, barriers, cross-core visibility | ✅ available |
| **QEMU/KVM** | End-to-end behaviour under a real workload; the final acceptance criterion | ⚙️ harness ready, no workload |
| Crash-point replay | Whether any power-cut point recovers | ❌ not implemented |
| Model-based differential | Functional correctness: is the result of an operation sequence right | ❌ not implemented |

## LKMM

```bash
bash .claude/scripts/lkmm.sh
SINGLEFS_KERNEL_TREE=/path/to/linux bash .claude/scripts/lkmm.sh   # point at a tree
```

Every `litmus/*.litmus` must declare its expected verdict; the script compares it
against herd7's `Observation` line:

```
(* singlefs-expect: Never *)      the bad outcome must be impossible
(* singlefs-expect: Sometimes *)  the bad outcome is possible (control case)
```

**Every Never needs its own control case with the barrier removed.** With only one
file, a Never verdict cannot distinguish "the barrier held" from "this pattern never
had a chance to hit" — the script rejects any Never without a pairing.

Pairing is **by filename**: the control for `x.litmus` is `x-<suffix>.litmus`
(`x-nofence.litmus` by convention), declared Sometimes. However many Sometimes exist
elsewhere does not count — they answer a different question.

Two traps that only klitmus7 trips over (the script catches them earlier): using `rN`
requires `int rN;`; initialising an `atomic_t` parameter in the init block requires
the type.

## QEMU

```bash
bash .claude/scripts/qemu.sh --selftest        # verify the harness itself
bash .claude/scripts/qemu.sh . payload.sh      # run one workload
GATE_QEMU=1 bash .claude/scripts/gate.sh       # include the harness self-test in the gate
```

**`--selftest` runs a deliberately failing payload** to confirm the harness recognises
failure. If it cannot, it would report failures as successes — and it says so itself.

With no readable kernel the script fails and offers two routes (`SINGLEFS_KERNEL=`, or
making `/boot/vmlinuz-*` readable) — **it does not silently fall back to software
emulation**, which would be unusably slow while looking like it is running.

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
- **An unreadable verdict voids the round**, never counts as a pass — `lkmm.sh` with no
  `Observation`, `qemu/run.sh` with no exit marker, both fail outright.
