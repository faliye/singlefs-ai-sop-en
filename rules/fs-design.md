<!-- generated-from: rules/fs-design.md sha256:a69946bd4acc8ed1f9b6958f5d1e1d3d450c5014af1cd30b1827a7eef36615eb -->
<!-- doc-lint:rule-definition -->
# Design discipline

Concrete design decisions live in the project's `kb/decisions.md`. This document
holds only the **discipline that applies across decisions.**

## Start from the transaction, not from the feature

The hard core of the write path is the COW commit mechanism; everything else rides
on it. Any new feature is "another kind of transaction".
**A feature implementation without a transaction layer is not accepted.**

Corollary: the first runnable target is not "can create a file", it is "can commit a
transaction correctly" — even if all it can do is allocate a block, write it, and
commit, with no directories, no trees, and terrible performance.

## Accounting is a by-product of the transaction, not a later traversal

Space usage, quotas and snapshot usage must be **maintained incrementally at commit
time.** Any accounting design that needs "scan everything afterwards to work it out"
is rejected.

The checker may traverse — **its job is precisely to use traversal to verify that
incremental accounting has not gone wrong.** That pairing is the core means of
verifying accounting correctness; do not conflate the two.

## Metadata blocks must be self-describing

Pick up any single block and it must be able to answer "who am I, whom do I belong
to, which generation am I". This is the precondition that makes "rebuild by scanning
the whole device" physically possible, and **it can only be obtained at the format
layer; it cannot be added afterwards.**

## Feature bits in three tiers, with spare room

`incompat` (unknown → refuse to mount) / `compat_ro` (unknown → mount read-only) /
`compat` (unknown → mount normally).

## Different address spaces must be different Rust types

Logical address, physical address, in-device offset, generation, inode number — all
newtypes, and **mixing them must fail to compile.**

This turns a whole class of runtime bugs into compile errors, and is one of the main
reasons for choosing Rust. Passing `u64` around everywhere is a real source of bugs
in existing implementations.

## One transaction layer, shared by every structure

No second transaction mechanism for any particular path (such as fsync).
Giving fsync its own log tree is a common approach, and a common source of its own
bugs.

# No universal solution: branching is allowed, but pay the right debt

Performance, safety and device constraints often have no single optimum.
**Rather than forcing one universal solution, branch explicitly** — different scales,
different media, different safety requirements take different implementations.

## Why we can branch more finely than before

Filesystems used to favour a single approach, largely not because the problem
demanded simplicity, but because **one maintainer had to hold the whole thing in
their head**: memory load, tracing each case, keeping N paths consistent, knowing
which other places a change must be mirrored to — these are human bottlenecks.

That part of the cost really has been lifted: tracing logic and maintaining
consistency are machine strengths.
**So this project does not presume "simpler is better"; finer branching is allowed
where it fits.**

This is the criterion from `machine-first.md` in its design-level form — the original
reason for "unify the design, minimise concepts" was that the maintainer had to fit
it in, and that reason no longer holds.

## But four costs have not been lifted

1. **Verification is combinatorial, not linear.**
   Crash consistency is not counted per branch but per **interaction**: an operation
   whose conditions change mid-flight and which resumes down a different branch means
   the cross product must be verified. This does not shrink because of who wrote the
   code; it shrinks only if **the branches are isolated from each other.**
   → **Branch only on inputs that cannot change during an operation.** Device type is
   stable, fine; "current fill level" can cross a threshold mid-operation, and that is
   a dangerous branch variable.

2. **A branch in the format is permanent.**
   A code branch can be deleted; **an on-disk layout branch cannot** — every future
   reader must support every historical layout forever.
   → **Branch freely in behaviour; branch sparingly in format.**

3. **Gate wall-clock time is real.**
   Branches × crash points is machine time, but if the gate takes eight hours people
   stop running it before submitting — and "run it locally first, and it agrees with
   the remote" is the pivot of this whole design. **Gate runtime is itself a design
   constraint.**

4. **Rare branches rot.**
   A branch reached one time in a thousand has bugs that only surface in production —
   unless a test can **force** entry into it.

## Five hard requirements

1. **The switching criterion must be explicit, computable, and written down.**
   Not "by feel", and not a magic number buried in the code. A criterion you cannot
   state = a branch you have not thought through.
2. **Every branch must be forcibly reachable from a test.**
   Leave a test-only switch that ignores the criterion and takes a chosen branch
   directly. **If you cannot get in, it is not tested** — a branch covered by luck is
   not covered.
3. **Every branch is verified separately, including crash-point replay.**
   "The other branch was tested" does not count — the whole point of a branch is that
   behaviour differs, so crash semantics may differ too.
4. **Branches must be observable**: at runtime you can see which one was taken.
   **Taking the wrong branch is a performance cliff, and cliffs do not raise errors** —
   with no way to observe, it will never be found.
5. **Branch variables must not change mid-operation.** See requirement 1 above.

**Conversely: when the criterion cannot be written, build only one path — and build
the slow, correct one first.** The fast path is an optimisation; the slow path is the
floor. Get the floor first, then talk about optimisation.

## Index structures may vary; the transaction layer may not

Access patterns differ enormously between kinds of data (reverse index: write-heavy,
read-rare; directory entries: point lookups; extents: range queries). Choosing a
structure per access pattern is permitted and even recommended.
**The one thing that may not vary is the transaction layer.**
