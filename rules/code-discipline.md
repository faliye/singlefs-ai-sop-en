<!-- generated-from: rules/code-discipline.md sha256:f2aebb3050a6fa8595182ca917ccfecc0eca2516614b34eecd81687214b66834 -->
<!-- doc-lint:rule-definition -->
# Code discipline: how machine-first lands in code

`machine-first.md` says why the old rules need re-examining and how to examine them; this file is the
result of that examination, and it is what you follow when writing code.
The governing principle is "AI-friendly ≠ unreadable" in `engineering-philosophy.md`: the only thing
relaxed is compression done so it would fit in a human head; making meaning explicit is not relaxed
at all.

**There is one principle: past best practice does not get to decide for us.**
Popular practices were examined in batches, grouped by where they come from; the verdicts are in
"Popular practices, examined one by one", and the sections before it are how we write as a result.
For a principle not listed here, run the procedure in `machine-first.md` before using it — the
default is doubt, not compliance.

This covers all the code we write in this project: product code, tests, experiment code, scripts.
How far the gate checks is at the end, in "Which half the gate handles".

## Names: the name alone says what it is and what it does

**The criterion: without the comments, without the call site, without the implementation — does the
name alone say what it is, what it does, and what it does not do?**
If not, rename it. Failing to produce such a name usually means the job of this code is not yet
pinned down.

`fn f(a: u64, b: u64)` and `fn commit_transaction(generation: Generation, root: LogicalAddress)`
differ by a whole layer of meaning, and a model cannot recover it by reading the implementation.

This covers **every name we declare ourselves**: variables, parameters, functions, methods, types,
fields, enum variants, constants, modules, files, generic parameters, lifetimes, closure parameters,
loop variables, test functions, macros.
Names fixed from outside are not ours to decide: implement `Display` and the method is `fmt`.

| Rule | Detail |
|---|---|
| **No length cap** | There is no "keep names short". Ten more characters for one more layer of meaning is always a good trade on the machine's side |
| **No abbreviations** | Spell out `cnt`, `idx`, `buf`, `tmp`. The only exception is a registered domain abbreviation — see "The abbreviation registry" |
| **No single letters** | Loop variables, closure parameters, generic parameters, lifetimes and error bindings all count: `i` → `stripe_index`, `T` → `Key`, `'a` → `'journal`, `Err(e)` → `Err(error)` |
| **Put preconditions and scope into the name** | `write_node` says nothing; `append_verified_node_to_journal` says three things: it appends rather than overwrites, its argument is verified, it lands in the journal |
| **State the unit** | If a type can express it, leave it to the type (`LogicalAddress`, `Duration`); if not, put it in the name: `offset_in_blocks`, not `offset` |
| **Booleans read as predicates** | Say which side is true: `is_checksum_verified`, not `flag`, `status`, `ok` |
| **Test names state the scenario and the expected outcome** | `crash_between_data_write_and_commit_keeps_previous_generation`, not `test_commit` or `it_works` |
| **One name per semantic concept across the repository** | The unit is the semantic concept, not the word: one concept may not be `generation` here and `epoch` there; two concepts, however close their underlying meaning (the generation recorded on disk, the generation in a transaction's lifecycle), may not share a name — each carries its role: `committed_generation`, `transaction_generation` |
| **A name stands on its own, without the module path** | Write `block::BlockAddress`, not `block::Address`: after the `use`, the call site has only `Address`, and the model cannot see the path. This is **deliberately making the name carry namespace information**; the cost is that type names form a hand-made namespace (`BlockAddress`, `JournalAddress`, `LogicalAddress`…), and we accept that cost. The reason is that the path is absent from retrieval results and call sites — not that "longer names are better" |
| **A new meaning gets a new name** | Shadow a name only when the meaning stays the same; when raw data becomes verified data, or a byte count becomes a block count, pick a new name |

### Why an abbreviation is pure loss

It saves characters and drops meaning, and the model cannot recover it — worse, it will fill it in.
`cnt` means block count in one place and retry count in another, and the model that retrieves one of
them will not say "I can't tell"; it will proceed on the most common reading. In a filesystem, `ext`
can be extent or extension; `gen` can be generation or generator.

**The criterion is "is there one authoritative definition", not "will everyone recognise it".**
So Rust keywords and primitive type names (`mut`, `fn`, `str`, `u64`) are not abbreviations — the
language reference is their one definition; the project's own domain abbreviations (`lba`, `crc`) may
be used only once registered.

### The abbreviation registry

`.claude/abbreviations` at the project root, one entry per line, with the full form and meaning after
`#`:

```text
lba  # logical block address: the number the filesystem gives a block
crc  # cyclic redundancy check
```

The registry is that abbreviation's single authoritative definition; everywhere else follows it, and
nobody writes a second one. **Single letters may not be registered**: one letter has no single meaning.

**The project's own numbers** (experiment E57, decision D22 and the like, registered in the kb), when
used as one segment of a name, are registered as a class — "this letter followed by digits":

```text
e<数字>  # experiment numbers, registered in .claude/kb/experiments.md
```

(`<数字>`, "digits", is literal syntax that the lint reads as written.)
Once registered, the `e57` in `e57_field_authority` no longer counts as a single letter. A number in a
name carries its short name here too (`kb-discipline.md`, item 5): write `e57_field_authority`, not a
bare `e57`.

### Where meaning lives: comment < name < type

Three ways to write the same thing, worst to best:

```rust
// 1. Meaning in a comment — the worst
// dim0: ethnicity  dim1: gender  dim2: age band
pop[i][j][k]

// 2. Meaning in the names — long-winded by old taste, but machines like it
number_of_china_people[han][woman][teenager_18_to_24]

// 3. Meaning in the types — best; getting a dimension wrong will not compile
number_of_china_people[Ethnicity::Han][Gender::Woman][AgeBand::Teenager18To24]
```

The first is compression done so it would fit in a human head: names cut to the shortest, meaning
moved into a comment. When line width and screens were limited that was reasonable, but a comment
drifts from the code and a name does not — the name *is* the code.
In the second, every dimension describes itself, so **you can judge whether an access is correct
without any context**.
The third pushes the information from "readable" to "checkable": swap two dimensions and it will not
compile.

**Constraints the type system can express go into types; those it cannot go into names — neither
belongs in a comment. Later is better, because later is harder to drift from the implementation.**

## Branches: write them out exhaustively, no wildcard arm

```rust
// ✗ General path: when a new case appears, the compiler says nothing
fn node_size_in_bytes(device: &Device) -> u32 {
    if device.is_rotational { 64 * 1024 } else { 16 * 1024 }
}
```

```rust
// ✓ Exhaustive branches: add Zoned and this stops compiling until you handle it
enum Medium {
    Rotational,
    SolidState,
    Zoned { zone_size_in_bytes: u32 },
}

fn node_size_in_bytes(medium: &Medium) -> u32 {
    match medium {                                  // the point is the `_ =>` arm that is **not** there
        Medium::Rotational                   => 64 * 1024,
        Medium::SolidState                   => 16 * 1024,
        Medium::Zoned { zone_size_in_bytes } => (*zone_size_in_bytes).min(256 * 1024),
    }
}
```

**The load-bearing part is the `_ =>` you did not write.** Add a wildcard arm and the code looks
shorter and more "general", while what it actually did was switch off the compiler's exhaustiveness
check: a new case will quietly fall into the wildcard and behave wrongly with nobody raising an alarm.
**The criterion is not "are there many branches", it is "when a case is missed, who notices first"**:
the compiler, or production.

- **For an enum that is a closed set, a `match` has no `_ =>`.** Our own enums are closed sets by
  default: how many cases there are is ours to decide.
- **Where the semantics allow unknown values** (codes read from disk, received over a protocol, or
  that will gain new values later), make "unknown" an explicit variant (`Unrecognized(raw_code)`) and
  keep the `match` exhaustive. A wildcard arm is just as harmful here: add a new known variant later
  and `_ =>` quietly routes it into the "unknown" arm — exactly the mistake protocol evolution makes
  most easily.
- **Only two kinds are genuinely open**: enums from outside (another crate's, or marked
  `#[non_exhaustive]`), and a public API we have committed to externally and must leave room to evolve
  (see "When not to apply them"). Where a wildcard arm really is needed, write
  `#[allow(clippy::wildcard_enum_match_arm, reason = "…")]` at that spot.
- **Enums used inside the workspace are not marked `#[non_exhaustive]`**: it forces every `match` in
  the workspace's other crates to carry a wildcard arm, which switches the exhaustiveness check off
  wholesale.
- **A closed set (how many cases there are is ours to decide) is an `enum` with an exhaustive `match`,
  not a trait object.** An enum puts every case in one place, so the path count can be stated;
  interactions between two cases (state transitions, pairwise comparisons) can be written as an
  exhaustive `match` on a tuple, and a missing combination does not compile. A trait's default
  methods, by contrast, are wildcard arms: add an implementation and every method it did not override
  quietly keeps the default behaviour. Open sets (plug-ins, test doubles) still use traits.
- **A trait with only one implementation is not extracted.** Extract a trait only when there are at
  least two real implementations (the model used for differential testing counts as one); a layer
  extracted "so it is easy to swap later" is an open set nobody verifies.
- **The arm count of an exhaustive `match` does not count as complexity.** The compiler counts it for
  you; what needs a bound is the number of independent conditions, which make paths grow
  exponentially.

## Types: make illegal states impossible to write

```rust
// ✗ One underlying type carrying several meanings: mixing them compiles, and fails at runtime
fn read_block(address: u64) -> Block;
fn free_extent(start: u64, length_in_blocks: u64);
// A caller passes a logical address where a physical one belongs; the compiler has nothing to say
```

```rust
// ✓ Newtypes: mixing them does not compile
#[derive(Clone, Copy, PartialEq, Eq)] pub struct LogicalAddress(pub u64);
#[derive(Clone, Copy, PartialEq, Eq)] pub struct PhysicalAddress(pub u64);

fn read_block(address: PhysicalAddress) -> Block;
// read_block(LogicalAddress(value)) fails to compile — a whole class of runtime bugs becomes a compile error
```

Go one step further and **make "not yet validated" a type too**, so that "forgot to validate" cannot
be written:

```rust
// ✗ State as a boolean field: forget to check it and the compiler does not care
struct Node { bytes: Vec<u8>, is_checksum_verified: bool }

// ✓ State as a type: what has not been verified cannot become the verified type
struct RawNode(Vec<u8>);        // straight off the disk, unverified
struct VerifiedNode(Vec<u8>);   // checksum compared and matched

impl RawNode {
    fn verify(self, expected_checksum: Checksum) -> Result<VerifiedNode, CorruptBlock> { /* ... */ }
}

fn walk(node: &VerifiedNode) { /* ... */ }   // skip verification and you cannot build the argument
```

- One underlying type carrying different meanings (logical address, physical address, generation;
  byte count, block count) gets one newtype per meaning.
- States like "verified or not" and "durable or not" become types, not boolean fields. **Verify once
  at the boundary**, turn the value into a type that carries the proof, and from there on trust the
  type instead of re-checking at every layer.
- **Replace a boolean parameter with an enum**: `write(node, true)` says nothing at the call site;
  `write(node, Durability::Synced)` does.
- **Encapsulation exists to protect an invariant, not to hide things.** A field with an invariant is
  private and can only come in through a constructor that checks it; a field without one is simply
  public, with no boilerplate getter / setter.
- **A type with an invariant does not implement `Default`**: a value produced without passing the check
  is exactly what the type exists to keep out.
- **When constructing our own structs, write every field out**, not `..Default::default()`.
  That is a wildcard arm over fields: add a field and every place that did not write it quietly gets
  the default. Write every field out and a missing one does not compile. Required fields are not left
  for a builder to check at runtime; they become constructor parameters, and only genuinely optional
  ones go into a builder.
- **No `as` for numeric conversions that can lose a value.** `as` truncates, drops the sign and wraps
  without a word; in a lossy direction use `try_from` and handle the failure, in a lossless one use
  `from`.

**Branches and types are two sides of one thing**: one makes "a missed case" something the compiler
catches, the other makes "an illegal combination" something you cannot write. Both move the check off
the human and onto the machine.

## Errors: the recoverable go into types, a broken invariant is asserted

- **Recoverable failures** (I/O errors, corrupted on-disk data, running out of resources) **go through
  `Result`**. The variants of an error enum are **split by the decision the caller has to make**, not
  expanded one per underlying cause: what the layer above actually branches on may be only "not
  found", "corrupt", "no space" and "other I/O error", with the dozens of underlying OS errors carried
  as the source inside "other I/O error". The caller `match`es on it, and a case left unhandled does
  not compile.
- **The criterion is whether the caller loses a decision it should be able to make.** A library's
  public boundary does not use a catch-all error type like `anyhow` or `Box<dyn Error>`: it mashes
  "retry", "report corruption" and "report no space" into one lump, the caller cannot branch, and that
  is a wildcard arm over errors. The other extreme is wrong too — re-expanding every underlying failure
  into its own variant at every layer: with too many variants, the few that matter get buried.
  Tests and command-line tools may use a catch-all error type.
- **A broken invariant is a bug, and is asserted** — do not carry bad state onward, and never write it
  to disk.
- **No `unwrap()`; write `expect`**, with a message stating which invariant it relies on:
  `expect("journal head was verified in open(); None here means verification was skipped")`. That
  makes it an assertion with a name.

## Functions and nesting: cut by "independently verifiable", bound by path count

**A function's length is set by "one independently verifiable thing", not by screen height.** One thing
in three hundred lines is fine; two things squeezed into twenty lines get split.

**Nesting depth is not capped; path count is.** "No more than three levels of nesting" measures the
wrong thing:

| | Paths | Testable? |
|---|---|---|
| 12 nested `for` loops over a 12-dimensional structure | **1** | fully testable as control flow; the state inside the loop body is verified separately |
| 12 nested independent `if/else` | **2¹²** | not testable |
| 3 nested independent `if/else` | 8 | barely |

Loop nesting without early exits adds almost no paths (each `break`, `continue` and `?` adds one);
conditional nesting multiplies them exponentially.
The old rule capped both together because both are equally hard on a human to read — that is a human
constraint, not a verification constraint.
So any depth is fine, provided both hold: the path count is statable and bounded;
and each level's iteration target has a name and a type, not `a[i][j][k][l]`. The problem with that form
is not that it is deep, it is that it carries no information.

**Path count measures control flow only; it is not the whole of verification difficulty.** How many
times a loop iterates, the state carried from one iteration to the next, early exits, and data
dependencies between nested loops can each make the space to verify larger without adding a single
path. So path count is a hard metric of control-flow complexity: over the bound, split it; under the
bound does not mean easy to verify. A loop must additionally state three things:
an upper bound on its iterations, the state it carries across iterations together with that state's
invariant (written as an assertion), and every early exit.
**"There is only one path" is no defence for an algorithm that is hard to verify.**

- **Parameters are not capped by count.** Each needs a name and a type; only when several parameters
  together form a concept with an invariant do they become one type — never a struct assembled just to
  shorten a signature.
- **Results go in the return type**, not back through `&mut` out-parameters: where data flows is written
  in the signature.
- **Push side effects to the edge; write the core as pure functions.** Pure functions can be exhausted
  and differentially tested against a model; the I/O shell can only be verified by crash-point replay,
  so keep it thin. A function that both queries and modifies says both in its name.
- **Early returns are fine.** Single exit was for humans following control flow; we count paths, not
  exits.

## Comments, assertions, constants, duplication

- **Comments say "why it is this way"**, not "what this does" — the latter a machine reads for itself.
  Constraints go into types, names or assertions, not comments: comments drift away from the
  implementation; names and types do not.
- **Doc comments state the contract a type cannot**: which errors it returns, when it panics, what an
  `unsafe` function requires of its caller. Doc comments that restate the name are deleted.
- **Every `unsafe` block has a `// SAFETY:` comment above it** saying why it is safe. That is an
  invariant that cannot be written into a type, so it can only go into a comment.
- **Invariants are written as assertions**, not only in docs.
- **Magic numbers become named constants**: a number has exactly one definition, with its unit in the
  name or the type.
- **Duplication is generated, never hand-copied.** Mirroring a change in many places is a machine
  strength, provided those places come from one generator: generated duplication does not diverge;
  hand-copied duplication does. Macros and `build.rs` are the proper way to generate, but they only
  generate and do not hide logic, and what they generate has tests watching it — names that come out of
  a macro expansion are invisible to the naming check.
- **Unused code is deleted**, commented-out code included: git can bring it back, and what stays is a
  path nobody runs. A `never used` from the compiler is treated as an error (`command-safety.md`: a
  warning is a free signal).
- **A `TODO` says what is missing and why it is not done now.** A check that is owed goes into the
  project kb's list of owed checks, and the code points there; a bare "TODO: fix later" is never
  revisited.

## Popular practices, examined one by one

Each one was put through the procedure in `machine-first.md`: what problem did it originally solve,
does that problem still exist today, does it have a second reason. There are only six **dispositions**:
dropped, relaxed, rewritten, kept (reason rewritten), left to tooling, demoted to advice.

### Overall stance

| Popular practice | Source | What it originally solved | Disposition | How we write it |
|---|---|---|---|---|
| Code is read more often than written; optimise for the reader | *Clean Code* and others | people must maintain the code | **rewritten** | The main reader is a model: optimise for machine verification and information content; human attention goes to whether the test is right, whether the invariant is right, whether the basis holds, whether to accept the trade-off (`engineering-philosophy.md`) |
| Don't write clever code | common saying | people cannot follow it | **kept, reason rewritten** | A clever general path eats the exhaustiveness check, and a missed case raises no alarm |
| KISS, keep it simple | common saying | it fits in a head | **rewritten** | Simple means few control-flow paths, little state to verify, and every path written out — not few lines |
| YAGNI, don't write what you won't use | Extreme Programming | human time | **kept, reason rewritten** | Unused code is a path nobody verifies |
| Principle of least astonishment | common saying | the user can guess what it does | **kept, reason rewritten** | Names and types must let a model guess the behaviour correctly; a wrong guess means the name or type did not carry enough information |
| Convention over configuration | Rails and other frameworks | less configuration to write | **dropped** | Implicit default behaviour is an invisible path: write out every value you need |
| Separation of concerns; high cohesion, low coupling | common saying | a person thinks about one thing at a time | **kept, reason rewritten** | Verifying one piece on its own depends on clear boundaries and explicit contracts |
| Files and modules must not be too long | common saying | people cannot page through them | **relaxed** | Modules are cut by contract, not by line count |

### Naming

| Popular practice | Source | What it originally solved | Disposition | How we write it |
|---|---|---|---|---|
| Names should be short, shorter the closer to the declaration | Go Code Review Comments, Linux kernel coding style | line width and human short-term memory | **dropped** | No length cap; see "Names" |
| Call the loop counter `i` and the temporary `tmp` | Linux kernel coding style | fewer characters, convention | **dropped** | Name it for what it is: `stripe_index`, `unflushed_block` |
| Generic parameters `T`, `U`; lifetimes `'a` | Rust convention | fewer characters | **dropped** | Name them for their role: `Key`, `Value`, `'journal` |
| Abbreviations everyone knows are fine | common saying | fewer characters | **dropped** | Spell them out; a domain abbreviation may be used once registered |
| Names must be pronounceable | *Clean Code* | people discuss code out loud | **this reason dropped** | What matters is no ambiguity and searchability; with no abbreviations, a name is pronounceable anyway |
| Names must be searchable | *Clean Code* | people search in an editor | **kept, and stronger** | grep is how a model finds things: one name per semantic concept, and one search across the repository finds them all |
| Hungarian notation, `m_` prefixes, `I` on interface names | old C / C++ / C# conventions | the editor did not show types | **dropped**; what it wanted goes to types | A different meaning is a different newtype; a unit the type cannot express goes into the name |
| No noise words like `Manager`, `Helper`, `Data`, `Info` | *Clean Code* and others | they carry no information | **kept** | If deleting the word takes nothing away from the name, it was saying nothing |
| Don't repeat the module name in the type name (`block::Address`) | clippy's `module_name_repetitions`, Go package naming conventions | the full path reads as repetitive | **dropped** | A name stands on its own: write `BlockAddress`. This deliberately makes the name carry namespace information, and accepts longer type names as the cost |
| Getters have no `get_` prefix | Rust API Guidelines C-GETTER | consistency with std | **kept, reason rewritten** | Consistent conventions keep retrieval and mechanical checks reliable |
| Conversion methods are named `as_`, `to_`, `into_` | Rust API Guidelines C-CONV | consistency with std | **kept, reason rewritten** | The prefix carries information, but it is a convention the compiler does not check: `as_` is borrowed to borrowed and conventionally near-free; `to_` is conventionally expensive and usually yields a new value, though some are borrowed to borrowed, like `to_str`; `into_` consumes ownership. The C-CONV table is the authority, and an implementation must deliver the cost class its name promises |
| Boolean names start with `is_`, `has_` | common saying | reads like a sentence | **kept, reason rewritten** | Say which side is true |
| Types are nouns, methods are verbs | *Clean Code* | reads like a sentence | **kept, reason rewritten** | Part of speech is information: a noun says what it is, a verb says what it does |
| Rust's case conventions (`snake_case`, `CamelCase`) | Rust convention, rustc default warnings | tell types from values at a glance | **kept, reason rewritten** | Consistent conventions are what make mechanical checks work |
| Shadowing (`let text = text.trim()`) is idiomatic | Rust convention | one name fewer to invent | **rewritten** | Shadow when the meaning stays the same; a new meaning gets a new name. clippy stops shadowing with an unrelated meaning |
| Test functions are called `test_<function under test>` | common convention | find what is being tested at a glance | **dropped** | State the scenario and the expected outcome |

### Functions

| Popular practice | Source | What it originally solved | Disposition | How we write it |
|---|---|---|---|---|
| Functions short: a screen or two, ideally under 20 lines | *Clean Code*, Linux kernel coding style | screen height and short-term memory | **kept, reason rewritten** | Cut by "one independently verifiable thing", not by line count |
| Do one thing | *Clean Code* | a person thinks about one thing at a time | **kept, reason rewritten** | Only one thing has one independently verifiable contract |
| One level of abstraction per function | *Clean Code* | reads smoothly | **relaxed** | Not required; what matters is a clear contract and a bounded path count |
| Read top-down (the stepdown rule) | *Clean Code* | people read from the top | **dropped** | Models read by retrieval, not in order |
| At most three parameters | *Clean Code* | the caller cannot remember the order | **relaxed** | Each parameter needs a name and a type; they become one type only when they form a concept |
| No flag arguments | *Clean Code* | the reader cannot remember what `true` means | **kept, reason rewritten** | The call-site line must carry its own information: use an enum |
| No output arguments | *Clean Code* | the reader takes them for inputs | **kept, reason rewritten** | Results go in the return type, so where data flows is written in the signature |
| No side effects; functional core, imperative shell | *Clean Code*, Gary Bernhardt | state changed behind your back | **kept, reason rewritten** | Pure functions can be exhausted and differentially tested; I/O is pushed into the shell, which is verified by crash-point replay |
| Separate queries from modifications | Bertrand Meyer (CQS) | calling twice gives different results, which surprises people | **kept, reason rewritten** | A function that both queries and modifies says both in its name |
| Single exit | structured programming | people can follow the control flow | **dropped** | Early returns are fine; we count paths, not exits |
| No more than three levels of nesting | Linux kernel coding style | layer-by-layer parsing is hard for people | **relaxed** | No cap on depth; control-flow path count is what is bounded, and loop state is verified separately |
| Cyclomatic complexity below some number | McCabe, static-analysis tools | too many branches for a person to follow | **rewritten** | The arms of an exhaustive `match` do not count — the compiler counts them for you; what is bounded is the number of independent conditions |

### Types, data and encapsulation

| Popular practice | Source | What it originally solved | Disposition | How we write it |
|---|---|---|---|---|
| Value objects instead of primitive types | *Refactoring* (Primitive Obsession) | domain meaning | **kept, and stronger** | A different meaning is a different type |
| Make illegal states unrepresentable | Yaron Minsky | fewer checks to write | **kept** | See "Types" |
| Parse, don't validate | Alexis King | the conclusion of validation gets lost | **kept** | Verify once at the boundary, turn it into a type that carries the proof, and trust the type from there on |
| Private fields with getters / setters | Java-family encapsulation convention | freedom to change the implementation later | **rewritten** | Encapsulation exists to protect an invariant: fields with one are private, fields without one are simply public |
| Immutable by default | Rust convention, functional programming | fewer mistakes | **kept, reason rewritten** | Less state means fewer combinations to verify |
| No global mutable state; no singletons | common saying; *Design Patterns* as a cautionary example | nobody can trace who changed it | **kept, reason rewritten** | It defeats local reasoning and makes tests irreproducible |
| Derive `Default` whenever you can | Rust convention | less code | **rewritten** | A type with an invariant does not implement `Default` |
| Struct update syntax `..Default::default()` | Rust convention | fewer fields to write | **dropped for our own structs** | It is a wildcard arm over fields: write every field out, and a missing one does not compile |
| Many fields means a builder | Rust API Guidelines C-BUILDER | many optional parameters | **rewritten** | Required fields are constructor parameters, forced at compile time; only genuinely optional ones go into a builder |
| Numeric conversion with `as` | C-family convention | short to write | **dropped in lossy directions** | Use `try_from` and handle the failure; clippy stops it |
| Don't typedef a struct to a new name | Linux kernel coding style | the reader cannot see it is a struct | **dropped** | Wrapping is the whole point of a newtype: it wraps meaning, it does not hide structure |
| Name magic numbers | common saying | the reader does not know what the number is | **kept, reason rewritten** | A number has exactly one definition, with its unit in the name or type |

### Branches and abstraction

| Popular practice | Source | What it originally solved | Disposition | How we write it |
|---|---|---|---|---|
| Replace conditional with polymorphism | *Refactoring* | add a case without touching old code | **dropped on closed sets** | An `enum` with an exhaustive `match`; see "Branches" |
| Open/closed principle | SOLID | add features without changing old code | **dropped on closed sets** | What we want is the opposite: add a case and the compiler forces every piece of old code to be revisited |
| Single responsibility | SOLID | a class has one reason to change | **kept, reason rewritten** | One piece has one independently verifiable contract |
| Liskov substitution | SOLID | a subclass can stand in for its parent | **kept, reason rewritten** | A trait's contract holds for every implementation: contract tests run against every implementation — the same thing as running the positive control against every arm under test (`test-discipline.md`) |
| Interface segregation | SOLID | not forced to depend on unused methods | **kept, reason rewritten** | The smaller the trait, the smaller the contract each implementation must verify, and the fewer default methods (wildcard arms) |
| Dependency inversion: depend on abstractions, not concretions | SOLID | easy to swap, easy to mock | **rewritten** | Extract a trait only with at least two real implementations; the model used for differential testing counts as one |
| Law of Demeter: talk only to immediate friends | Northeastern University's Demeter project | fewer knock-on changes when structure changes | **relaxed** | Chain freely inside a module; never reach through another module's internals, which bypasses its contract |
| Design patterns: factory, strategy, visitor | *Design Patterns* | reuse a known structure | **rewritten** | Strategy and visitor over a closed set are just a `match`; a layer added for the pattern's sake is an open set nobody verifies |
| DRY, avoid duplication | *The Pragmatic Programmer* | a change in one place must be mirrored in many | **relaxed** | Mirroring changes is a machine strength, but duplication must be generated, never hand-copied |
| Abstract only after the third duplication | *Refactoring* (Don Roberts) | a wrong abstraction is expensive | **kept, reason rewritten** | An extracted general path eats the exhaustiveness check; hand duplication to a generator first |
| A `default:` / `_ =>` fallback is safer | defensive programming | a missed case does not crash | **dropped on closed sets** | Let the compiler catch the missed case; where unknown values are allowed, make "unknown" an explicit variant |

### Error handling

| Popular practice | Source | What it originally solved | Disposition | How we write it |
|---|---|---|---|---|
| Fail fast | *The Pragmatic Programmer* | don't run on with bad state | **kept** | Assert when an invariant is broken; never write bad state to disk |
| Library code never panics; always return `Result` | Rust convention | the caller can recover | **rewritten** | The recoverable go through `Result`; a broken invariant is a bug and is asserted |
| A catch-all error type such as `anyhow`, `Box<dyn Error>` | Rust ecosystem convention | fewer error types to write | **dropped at a library's public boundary** | The caller cannot branch: variants are split by the decision the caller must make, with the underlying cause carried as the source |
| `unwrap()` when you are sure it cannot fail | Rust convention | one line shorter | **dropped** | Use `expect`, with a message naming the invariant it relies on |
| Defensive programming: check arguments everywhere | common saying | keep bad input out | **rewritten** | Check once at the boundary, turn it into a type that carries the proof, and do not re-check inside |
| Error codes or exceptions | a debate across languages | each has its cost | **left to types** | Failure is written into the return type, and the compiler forces it to be handled |

### Comments and documentation

| Popular practice | Source | What it originally solved | Disposition | How we write it |
|---|---|---|---|---|
| Write more comments | early convention | for people to read | **rewritten** | Comments say only why |
| Comments are failures; good code needs none | *Clean Code* | comments go stale | **rewritten** | "What it is" goes to names and types — agreed on that half; "why it is this way" is still written, because a name cannot hold it |
| Every public item gets a doc comment | rustc's `missing_docs` | users can understand it | **rewritten** | Delete the ones that restate the name; write only the contract a type cannot express |
| `unsafe` blocks get a `// SAFETY:` comment | Rust convention | say why it is safe | **kept, and clippy enforces it** | It is an invariant that cannot be written into a type |
| Keep commented-out code for later | common practice | afraid it will be lost | **dropped** | git can bring it back; what stays is a path nobody runs |
| Record change history and authors in comments | old convention | know who changed it | **dropped** | History goes to git and the CHANGELOG (`design-doc-discipline.md`) |
| `TODO` comments | common practice | a reminder to come back | **rewritten** | Say what is missing and why it is not done now; a check that is owed goes into the kb |

### Formatting

| Popular practice | Source | What it originally solved | Disposition | How we write it |
|---|---|---|---|---|
| No line longer than 80 columns | Linux kernel coding style, PEP 8 and others | terminal width | **left to tooling** | `rustfmt`'s defaults decide; a long name wraps the line, and no name is shortened to fit |
| 8-column indentation to force shallow nesting | Linux kernel coding style | deep nesting is hard for people to follow | **dropped** | No cap on depth; see "Functions and nesting" |
| Vertical openness; related lines kept together | *Clean Code* | people scan faster | **left to tooling** | No hand-tuning |

### Test code

| Popular practice | Source | What it originally solved | Disposition | How we write it |
|---|---|---|---|---|
| One assertion per test | *Clean Code* | see at a glance which one went red | **dropped** | One scenario per test; several assertions are fine, each with a message saying what it expects |
| Coverage must reach some percentage | common practice | quantify whether testing is enough | **rewritten** | Coverage is not correctness; argue with path counts and mutation lists (`test-discipline.md`) |
| Mock every dependency | common practice | fast, isolated | **rewritten** | A mock encodes the test author's assumptions, which amounts to handing yourself the answer; prefer the differential-testing model and real implementations |
| The test pyramid: mostly unit tests | Mike Cohn | speed | **rewritten** | Unit tests are fast feedback; acceptance rests on model-based differential testing, crash-point replay and QEMU stress (`show-me-test.md`) |
| Test code must be DRY too | common practice | less to write | **relaxed** | Each test reads as its own scenario; setup may move into helpers, assertions may not |
| A flaky test passes after a few reruns | common practice | don't block the pipeline | **dropped** | Unstable is reported as unstable, never rerun until green (`test-discipline.md`) |
| Write the test first | Kent Beck (TDD) | derive the design from usage | **kept, reason rewritten** | Watching it go red first is what proves it can go red (`show-me-test.md`) |

### How changes are made

| Popular practice | Source | What it originally solved | Disposition | How we write it |
|---|---|---|---|---|
| The Boy Scout rule: tidy whatever you pass | *Clean Code* | code rots over time | **rewritten** | Tidying goes into its own commit; mixed into a feature commit, it makes both bisecting and reverting harder |
| Tolerate no broken windows | *The Pragmatic Programmer* | small problems invite more | **kept, enforced by the gate** | Compiler and clippy warnings are treated as errors |
| Avoid premature optimisation | Knuth | human time is short | **demoted to advice** | — |

Design- and process-level principles (unify the design, keep PRs small, and the ones kept because they
have nothing to do with who reads the code) are in `machine-first.md`.

## When not to apply them

**These rules hold unconditionally for things you can delete, and need a separate calculation for
things you cannot.** A wrong name, branch or type in code can be changed; what is written into an
external commitment — an on-disk format, a protocol, a public API — cannot, and every implementation
afterwards has to support it forever. That side needs its own criterion; you cannot get there by saying
"explicit is safer".

## Which half the gate handles

| Rule | Who checks it |
|---|---|
| Single-letter names, common abbreviations | `scripts/naming-lint.sh` (gate stage "Naming discipline"): scans every `.rs` in the project, looking only at names we declare |
| No `_ =>` on an enum that is a closed set | clippy `wildcard_enum_match_arm`; one declared open gets an `allow` with a reason at that spot |
| `#[allow]` must carry `reason = "…"` | clippy `allow_attributes_without_reason` |
| Lossy `as` conversions | clippy `cast_possible_truncation`, `cast_sign_loss`, `cast_possible_wrap` |
| `unsafe` blocks need `// SAFETY:` | clippy `undocumented_unsafe_blocks` |
| Shadowing with an unrelated meaning | clippy `shadow_unrelated` |
| Warnings are errors | clippy `-D warnings` |
| Whether a name says enough, whether one concept has one name, whether a function does one thing, whether a comment says why, whether error variants follow the caller's decisions, whether loop state is written out, `..Default::default()` | review. A machine cannot judge these, or no ready-made check exists yet |
| Names in shell scripts | **not yet a check**; `gate.sh` lists it among the unimplemented stages |

The clippy ones run inside `scripts/check.sh`, i.e. the gate's "Build and unit tests" stage; that stage does not run when the project root has no `Cargo.toml`.

The abbreviation list in `naming-lint.sh` holds only the most common ones — **an abbreviation missing
from the list is still a violation**; the gate just cannot see it.
Forms it cannot recognise (patterns spanning lines, names produced by macro expansion) are not judged and
go to review; not recognised is not the same as passed.
Method names and associated types in a trait implementation, and declarations in an `extern` block, are
named from outside and are not checked.

Two configuration files on the project side, one entry per line, the reason after `#` — the reason may
not be omitted:

| File | What it governs |
|---|---|
| `.claude/abbreviations` | registered domain abbreviations; format in "The abbreviation registry" |
| `.claude/naming-lint-exclude` | directories or single `.rs` files not scanned. Excluding one means nobody watches the names in it, so say why; an entry pointing at a path that does not exist, or that excludes no `.rs` at all, is red. When sweeping old code, list the files not yet fixed one by one and delete a line as each is fixed: the exclusion only shrinks, and new files are checked |

Other names fixed from outside (field names of an external format and the like): write
`// naming-lint:external <why this name is not ours to decide>` on that line.
It exempts that one line only, and the reason may not be omitted either.
