---
name: crash-test
description: Run singlefs's verification suite — LKMM memory ordering, QEMU/KVM stress, crash-point replay, model-based differential testing. Use it to judge whether a write path is correct, or when a concurrency change needs verification.
---
<!-- generated-from: skills/crash-test/SKILL.md sha256:69b01b1630109225d2d11dde172ffaae70a758a4c4a11265e9ea536ea050c60e -->

# The verification suite

Rules live in `rules/test-discipline.md` and `rules/show-me-test.md`.

## What the four methods each cover

| Method | What it verifies | Provided by |
|---|---|---|
| **LKMM (herd7)** | Memory ordering on concurrent paths: lock-free structures, barriers, cross-core visibility | The shared script `lkmm.sh`; the gate's "LKMM" stage |
| **QEMU/KVM** | End-to-end behaviour under a real workload; the final acceptance criterion | The project: the VM harness and its gate stages both live there |
| Crash-point replay | Whether any power-cut point recovers | The project |
| Model-based differential | Functional correctness: is the result of an operation sequence right | The project |

The last three need the thing under test's own write stream, image format and checker, which the
shared gate cannot carry. When the project wires one of them in under `.claude/gate.d/`, that stage's
header states which item it covers:

```bash
# gate-stage: layer-0 crash-point replay (every crash state of the first transaction)
# gate-covers: 崩溃点重放
```

The only keys are `模型对拍` (model-based differential), `崩溃点重放` (crash-point replay),
`QEMU 真实负载` (QEMU real workload) and `QEMU 崩溃注入` (QEMU crash injection), copied literally from
`gate.sh`; one wrong character turns it red. Only when that stage ran and passed this round does the
unimplemented list at the end of `gate.sh` move the item under "covered by project-local stages".
Coverage says only "some stage is doing this, and it passed this time"; how far it reaches is in that
stage's own name and description. A stage with nothing to judge this round exits 77: it is recorded
as "not run this time" — not as a pass, and not as coverage.

## LKMM

```bash
bash .claude/scripts/lkmm.sh
SINGLEFS_KERNEL_TREE=/path/to/linux bash .claude/scripts/lkmm.sh   # point at a tree
bash .claude/scripts/lkmm.sh --static-only    # only the checks that need no herd7; exits 3 even when all pass — not a pass
```

Every `litmus/*.litmus` must declare its expected verdict; the script compares it
against herd7's `Observation` line:

```
(* singlefs-expect: Never *)      the bad outcome must be impossible
(* singlefs-expect: Sometimes *)  the bad outcome is possible (control case)
```

### The control: the same test with the barriers removed

**Every Never needs a control case**; otherwise a Never verdict cannot distinguish "the barrier held"
from "this pattern never had a chance to hit".

Pairing starts from the filename: the control for `x.litmus` is `x-<suffix>.litmus` (`x-nofence.litmus`
by convention), declared Sometimes. Then the content decides: with comments and the first line stripped,
the two are compared line by line, and the control may only drop barrier lines (`smp_wmb`, `smp_rmb`,
`smp_mb` and the like) or relax `smp_store_release` / `smp_load_acquire` to `WRITE_ONCE` / `READ_ONCE`,
at least once. `exists`, init and the reader must not change by a single character — change them and it
answers a different question. However many Sometimes exist elsewhere counts even less.

### Bound to the code

herd7 judges the shape the litmus writes down. If the code changes its publication order and the litmus
does not follow, the verdict is still Never and the gate stays green. So every Never names, in its file
header, the code it models — one anchor per line; after the anchor, a space and then a note is allowed:

```
(*
 * singlefs-expect: Never
 * singlefs-models: crates/<crate>/src/transaction.rs::publish_first_file writer
 * singlefs-models: crates/<crate>/src/recovery.rs::choose_root reader
 *)
```

The script checks three things: the file exists; it contains that `fn`; some `.rs` under `crates/`
spells out this litmus's filename. That last one is the binding test: it reads this litmus and compares
its writer / reader order against the order the code actually issues today (for example: record the
write-request stream, classify it into steps, compare item by item). The gate recognises it by filename;
whether it tests the right thing is for a person to judge.

A Never that models no code (such as the template pair) writes `singlefs-models: none — <why>`; the
reason may not be empty.

### Two traps that only klitmus7 trips over

The script catches them earlier: using `rN` requires `int rN;`; initialising an `atomic_t` parameter in
the init block requires the type.

## QEMU

The shared gate carries no VM harness. How many disks to attach, which binary to put in, how to capture
results, whether to record independently on the device side — all of it is bound to the thing under
test, so the harness and its gate stages live in the project.

The rules the harness must keep are in `rules/command-safety.md`: the VM's pid goes into a file and
cleanup kills that literal pid; result capture has a count gate; the exit code must make it back to the
host faithfully — the harness carries a deliberately failing case, and if it cannot recognise that
failure, the harness is broken. With no readable kernel it fails outright rather than falling back to
software emulation, which would be unusably slow while looking like it is running.

Once a gate stage is wired in, one that only runs the real workload declares
`# gate-covers: QEMU 真实负载`; one that also does crash injection and ends with the checker all green
declares `# gate-covers: QEMU 崩溃注入` as well — that is the final acceptance criterion.

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
  `Observation` fails outright, and so does the project's VM harness with no exit marker.
