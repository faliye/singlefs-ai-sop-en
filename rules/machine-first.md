<!-- generated-from: rules/machine-first.md sha256:4f389a6d813a2344c8ec8a66736c2d9fe0a631f4c11ae5e866ed3f5b23c390e0 -->
<!-- doc-lint:rule-definition -->
# Machine first: separate "readable" from "verifiable"

Many common software engineering principles grew on the premise that **one person
has to hold the whole thing in their head**. That premise is failing: code is
produced by humans and models together, and reading code is itself a job that can
go to a model.

**This project does not inherit those principles by default.** Each one is put
through the criterion again; the ones that fail are dropped.

## Stance: inherit sceptically, not by default

The default attitude toward established "best practice" is **doubt**, not compliance.

The reason is direct: **a substantial share of these principles exist to hold up a
quality floor under the constraint of limited human cognitive capacity.** They are
calibrated to the human limit. Machine capacity passed that line some time ago, and
**a floor set to the old limit is a floor set too low.**

This does not mean they are all wrong — it means **their reasons must be re-checked**,
and "everyone does it this way" is not one of them.

## What this stance assumes, and how to overturn it

"Machine capacity passed that line some time ago" is the foundation the whole
methodology rests on. **It is a falsifiable empirical claim, not an article of
faith.** So this section states the two premises it depends on, and what
observation would overturn them — **to argue against this stance, aim at these two
premises, not at the "relaxed" and "kept" tables.**

### Premise 1: context windows are now in the 2×10⁵ – 10⁶ token range

Three tiers: 200K / 500K / 1M. The 1M tier holds one crate together with its
tests, its kb, and its decision record in a single window. Basis: this round of
work on this repository runs in a 1M-tier session. Other vendors' numbers are not
written down here — a number you cannot check on the spot is not evidence.

**Calibration**: this is the **advertised window ceiling**, not "the same accuracy
across the whole window." Recall decay in the middle of a long context is a known
phenomenon, and this project has never measured at what scale it starts to bite.

- **What it holds up**: the relaxations of "keep PRs small", "minimise concepts",
  and "DRY by hand". Their original reason was entirely "it does not fit at once",
  and at the 1M tier that reason no longer holds automatically.
- **What it does not hold up**: "bound the scope of reasoning" is not among the
  relaxations. The window got bigger but is **still finite**, and accuracy decays
  with distance — so module boundaries and explicit contracts are worth more than
  before, not less.

### Premise 2: concurrent interleavings can now be decided exhaustively

For a judgement like "is this barrier enough", **intuition cannot produce a
re-checkable answer.** That job now splits in two:

| Who | Does what | Precision |
|---|---|---|
| herd7 / LKMM | **Exhausts** every interleaving the memory model permits, returns a Never / Sometimes verdict | Complete for the litmus you actually wrote |
| The model | Translates the synchronisation pattern in the code into a litmus; enumerates which scenarios to ask about | **Incomplete — the gap is here** |

**Calibration**: the tool is precise, the model is not. The tool answers only the
litmus you wrote; nothing guarantees you did not omit a scenario that should have
been asked. That is why `scripts/lkmm.sh` requires controls on both sides: with a
Never and no Sometimes, you cannot tell whether the barrier held or whether the
pattern never had a chance to hit.

- **What it holds up**: "exhaustiveness is machine-checkable", and what follows
  from it — "prefer exhaustive explicit branches" and "a bounded path count means
  it can be verified".

### When a premise fails, take these back

Premises are not permanent. Observe any of the following and the matching
relaxation is withdrawn on the spot:

| What you observe | What is withdrawn |
|---|---|
| The model drops a mid-context fact at this project's actual scale — misses a constraint already in the window | The "keep PRs small" relaxation; go back to slicing at a size that fits |
| Generated duplicate code no longer matches its generator | The "DRY" relaxation; back to no duplication |
| A litmus verdict contradicts a real-hardware stress result | "Exhaustiveness is machine-checkable"; `lkmm.sh` demotes to reference and stops being a gate |

**This table is this file's own disproof step** (`evidence-discipline.md`): a
premise for which you cannot state "what would overturn it" is not a premise, it
is a belief.

## When you meet an established principle, run this procedure

1. **Ask what problem it originally solved.** If you cannot say, drop it — a rule
   whose reason cannot be stated will not hold anyway.
2. **Ask whether that problem still exists.** If what it solved was "people cannot
   remember", "people cannot read that much", or "review bandwidth is short", it
   most likely does not.
3. **Ask whether it has a second reason.** Many rules happen to also solve a
   problem that has nothing to do with humans (see the "kept" table). If so →
   keep it, **and rewrite its stated reason to that one.**
4. **The rewritten reason must be able to land in the gate.** If it cannot, demote
   it to advice; do not write it as a rule.

## The criterion

> **Does this rule make the code easier to verify mechanically, or only easier on
> the human eye?**

- Only easier on the eye → **it can be dropped**
- Makes verification easier → **keep it, but rewrite the reason** — stop hanging it
  on "readability"; a rule hung on readability will not survive the next time
  someone asks "why?"

## Relaxed (artefacts of human bandwidth)

| Old principle | Its original reason | Disposition |
|---|---|---|
| unify the design, minimise concepts | the maintainer has to fit it in | **relaxed** — this is exactly where "one tree holds everything" came from |
| DRY, avoid duplication | a change in one place must be mirrored in many | **relaxed**: mirroring changes is a machine strength. But duplication must be **generated**, not hand-copied |
| keep PRs small | review bandwidth | **relaxed** — bisectability and revertibility still stand |
| avoid premature optimisation | human time budget | demoted to advice |
| functions must fit on one screen | short-term memory and screen height | **kept, reason rewritten**: length is set by "one independently verifiable thing", not by the screen |
| keep nesting shallow | human parsing capacity | **relaxed**: no cap on depth. The real constraint is **path count** — loop nesting adds almost no paths, conditional nesting multiplies them exponentially. See `engineering-philosophy.md` |
| names must be self-explanatory | for the next person reading | **kept, reason rewritten**: a name is information, not decoration, and a model cannot recover meaning that was omitted either |

⚠️ "Keep functions short" and "names must be self-explanatory" are the two most
easily damaged by this rule. **What is relaxed is compression done to fit a human
head, not making meaning explicit** — the latter is a gain for models too. See
"AI-friendly ≠ unreadable" in `engineering-philosophy.md`.

## Kept (independent of who reads the code)

| Principle | The real reason (rewritten) |
|---|---|
| crash consistency, invariants | physics, not psychology |
| on-disk format compatibility | a permanent external contract; it cannot be changed |
| reproducible tests | the precondition for any judgement, human or machine |
| fault domain isolation | a bug's blast radius has nothing to do with who reads the code |
| bisectable, revertible | **the larger the change volume, the more these are worth**, not less |
| explicit contracts at module boundaries | **without a contract you cannot verify one piece on its own** |
| bounded reasoning scope | **incremental verification requires bounded reasoning scope — context windows are finite too** |

The last two are the ones most easily discarded as "readability". They are not.

## Corollary: so write code this way

- Prefer **explicit exhaustive branches** over a clever general path —
  **exhaustiveness is machine-checkable**, "one function handles every case" is not
- Prefer **types that make invalid states unrepresentable** over comments explaining
  constraints
- Prefer **invariants written as assertions** over invariants written in docs
- Prefer **generated** duplication over hand-maintained duplication
- Comments say "why it is this way", not "what this does" — the latter a machine
  reads for itself

## What these two corollaries look like: code examples

The first two corollaries are the ones people argue about, so here they are in a form
you can point at. The argument usually starts as "is this branch really necessary?" —
**but the criterion is not necessity, it is who checks exhaustiveness.**

### Corollary one: exhaustive explicit branches beat a clever general path

```rust
// ✗ General path: when a new case appears, the compiler says nothing
fn node_size(dev: &Device) -> u32 {
    if dev.rotational { 64 * 1024 } else { 16 * 1024 }
}
```

```rust
// ✓ Exhaustive branches: add Zoned and this stops compiling until you handle it
enum Medium {
    Rotational,
    Ssd,
    Zoned { zone_size: u32 },
}

fn node_size(m: &Medium) -> u32 {
    match m {                       // the point is the `_ =>` arm that is **not** there
        Medium::Rotational          => 64 * 1024,
        Medium::Ssd                 => 16 * 1024,
        Medium::Zoned { zone_size } => (*zone_size).min(256 * 1024),
    }
}
```

**The load-bearing part is the wildcard arm you did not write.** Add `_ =>` and the code
looks shorter and more "general", while what it actually did was switch off the
compiler's exhaustiveness check — a new case will quietly fall into the wildcard and
behave wrongly with nobody raising an alarm.

**So the criterion is not "are there many branches", it is "when a case is missed,
who notices first"**: the compiler, or production. The former means explicit branches;
the latter is not generality, it is a deleted check.

### Corollary two: let types make illegal states unrepresentable

```rust
// ✗ One underlying type carrying several meanings: mixing them compiles, and fails at runtime
fn read_block(addr: u64) -> Block;
fn free_extent(start: u64, len: u64);
// A caller passes a logical address where a physical one belongs; the compiler has nothing to say
```

```rust
// ✓ Newtypes: mixing them does not compile
#[derive(Clone, Copy, PartialEq, Eq)] pub struct Lba(pub u64);  // logical address
#[derive(Clone, Copy, PartialEq, Eq)] pub struct Pba(pub u64);  // physical address

fn read_block(addr: Pba) -> Block;
// read_block(Lba(x)) fails to compile — a whole class of runtime bugs becomes a compile error
```

Go one step further and **make "not yet validated" a type too**, so that "forgot to
validate" cannot be expressed:

```rust
// ✗ State as a boolean field: forget to check it and the compiler does not care
struct Node { bytes: Vec<u8>, checksum_ok: bool }

// ✓ State as a type: what has not been verified simply cannot become the verified type
struct RawNode(Vec<u8>);        // straight off the disk, unverified
struct VerifiedNode(Vec<u8>);   // checksum compared and matched

impl RawNode {
    fn verify(self, expect: Checksum) -> Result<VerifiedNode, CorruptBlock> { /* ... */ }
}

fn walk(node: &VerifiedNode) { /* ... */ }   // skip verification and you cannot build the argument
```

**The two corollaries are two sides of one thing**: the first makes "a missed case" something
the compiler catches, the second makes an illegal combination something you cannot write.
**Both move the check off the human and onto the machine** — which is exactly this file's
criterion: does it make the code easier to verify mechanically.

### When not to apply them

**These two hold unconditionally for things you can delete, and need a separate calculation
for things you cannot.** A wrong code branch gets deleted; a branch written into an external
contract — an on-disk format, a protocol, a public API — cannot be, and every implementation
afterwards has to support it forever. That side needs its own criterion; you cannot get there
by saying "explicit branches are safer".

## One reminder in the other direction: documentation discipline is not up for dropping

The `doc-discipline.md` family (body text states only the current state, history at
the end, decision records in the kb) is **not an artefact of human bandwidth**.

In a document mixing history with the present, **a model has a harder time than a
person telling which value is current** — it has no "I remember this changed last
week" to fall back on, so it takes everything at face value.
**Stale or self-contradictory documents hurt models no less than people; they hurt
them more.**

By the same token, `kb/decisions.md` is rising in value, not falling: it is the only
place that can tell whoever comes next — human or model — **why this is the way it
is, and on what basis.**
