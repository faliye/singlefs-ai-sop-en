<!-- generated-from: rules/machine-first.md sha256:2b4c8c20f0354313b67712d81676a5899dfb44a47bf294e54fd4765b6ec4d8cf -->
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

## When you meet an established principle, run this procedure

1. **Ask what problem it originally solved.** If you cannot say, drop it — a rule
   whose reason cannot be stated will not hold anyway.
2. **Ask whether that problem still exists.** If what it solved was "people cannot
   remember", "people cannot read that much", or "review bandwidth is short", it
   most likely does not.
3. **Ask whether it has a second reason.** Many rules happen to also solve a
   problem that has nothing to do with humans (see the "kept" table below). If so →
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
