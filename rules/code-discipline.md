<!-- generated-from: rules/code-discipline.md sha256:86d7e7a6f7da94dd1f0b5c1e3abdd32cc4f7d045c839370b4d932302574efe84 -->
<!-- doc-lint:rule-definition -->
# Code discipline: how machine-first lands in code

Follow this when writing code. The procedure for examining old rules is in `machine-first.md`; the
governing principle is "AI-friendly ≠ unreadable" in `engineering-philosophy.md`: the only thing
relaxed is compression done so it would fit in a human head; making meaning explicit is not relaxed
at all.

**Past best practice does not get to decide for us.** The verdicts on popular practices, examined one by one,
are in "Popular practices, examined one by one".
For a principle not listed here, run the procedure in `machine-first.md` before using it — the
default is doubt, not compliance.

This covers all the code we write in this project: product code, tests, experiment code, scripts.
How far the gate checks is in "Which half the gate handles".

## Names: the name alone says what it is and what it does

**The criterion: without the comments, without the call site, without the implementation — does the
name alone say what it is, what it does, and what it does not do?**
If not, rename it. If you cannot produce such a name, first work out what this code is supposed to do.

This covers **every name we declare ourselves**: variables, parameters, functions, methods, types,
fields, enum variants, constants, modules, files, generic parameters, lifetimes, closure parameters,
loop variables, test functions, macros.
Names fixed from outside are not ours to decide: implement `Display` and the method is `fmt`.

| Rule | Detail |
|---|---|
| **No length cap** | There is no "keep names short" |
| **No abbreviations** | Spell out `cnt`, `idx`, `buf`, `tmp`. The only exception is a registered domain abbreviation — see "The abbreviation registry" |
| **No single letters** | Loop variables, closure parameters, generic parameters, lifetimes and error bindings all count: `i` → `stripe_index`, `T` → `Key`, `'a` → `'journal`, `Err(e)` → `Err(error)` |
| **Put preconditions and scope into the name** | `write_node` says nothing; `append_verified_node_to_journal` says three things: it appends rather than overwrites, its argument is verified, it lands in the journal |
| **State the unit** | If a type can express it, leave it to the type (`LogicalAddress`, `Duration`); if not, put it in the name: `offset_in_blocks`, not `offset` |
| **Booleans read as predicates** | Say which side is true: `is_checksum_verified`, not `flag`, `status`, `ok` |
| **Test names state the scenario and the expected outcome** | `crash_between_data_write_and_commit_keeps_previous_generation`, not `test_commit` or `it_works` |
| **One name per semantic concept across the repository** | The unit is the semantic concept, not the word: one concept may not be `generation` here and `epoch` there; two concepts, however close their underlying meaning (the generation recorded on disk, the generation in a transaction's lifecycle), may not share a name — each carries its role: `committed_generation`, `transaction_generation` |
| **A name stands on its own, without the module path** | Write `block::BlockAddress`, not `block::Address`. Type names thereby form a hand-made namespace (`BlockAddress`, `JournalAddress`, `LogicalAddress`…); write them that way |
| **A new meaning gets a new name** | Shadow a name only when the meaning stays the same; when raw data becomes verified data, or a byte count becomes a block count, pick a new name |

### No abbreviations; the criterion is whether there is one authoritative definition

**The criterion is "is there one authoritative definition", not "will everyone recognise it".**
Rust keywords and primitive type names (`mut`, `fn`, `str`, `u64`) are not abbreviations — the
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
nobody writes a second one. **Single letters may not be registered.**

**The project's own numbers** (experiment and decision numbers registered in the kb, and the like), when
used as one segment of a name, are registered as a class — "a letter followed by digits":

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

// 2. Meaning in the names
number_of_china_people[han][woman][teenager_18_to_24]

// 3. Meaning in the types — best; getting a dimension wrong will not compile
number_of_china_people[Ethnicity::Han][Gender::Woman][AgeBand::Teenager18To24]
```

**Constraints the type system can express go into types; those it cannot go into names — neither
belongs in a comment.**

## Branches: write them out exhaustively, no wildcard arm

```rust
enum Medium {
    Rotational,
    SolidState,
    Zoned { zone_size_in_bytes: u32 },
}

fn node_size_in_bytes(medium: &Medium) -> u32 {
    match medium {                                  // no `_ =>` arm: add a new Medium and this stops compiling until you handle it
        Medium::Rotational                   => 64 * 1024,
        Medium::SolidState                   => 16 * 1024,
        Medium::Zoned { zone_size_in_bytes } => (*zone_size_in_bytes).min(256 * 1024),
    }
}
```

**The criterion is not "are there many branches", it is "when a case is missed, who notices first"**:
make it the compiler, not production.

- **For an enum that is a closed set, a `match` has no `_ =>`.** Our own enums are closed sets by
  default: how many cases there are is ours to decide.
- **Where the semantics allow unknown values** (codes read from disk, received over a protocol, or
  that will gain new values later), make "unknown" an explicit variant (`Unrecognized(raw_code)`) and
  keep the `match` exhaustive, with no wildcard arm either.
- **Only two kinds are genuinely open**: enums from outside (another crate's, or marked
  `#[non_exhaustive]`), and a public API we have committed to externally and must leave room to evolve
  (see "When not to apply them"). Where a wildcard arm really is needed, write
  `#[allow(clippy::wildcard_enum_match_arm, reason = "…")]` at that spot.
- **Enums used inside the workspace are not marked `#[non_exhaustive]`.**
- **A closed set (how many cases there are is ours to decide) is an `enum` with an exhaustive `match`,
  not a trait object.** Interactions between two cases (state transitions, pairwise comparisons) can be
  written as an exhaustive `match` on a tuple. Open sets (plug-ins, test doubles) still use traits.
- **A trait with only one implementation is not extracted.** Extract a trait only when there are at
  least two real implementations (the model used for differential testing counts as one); do not
  extract one "so it is easy to swap later".
- **The arm count of an exhaustive `match` does not count as complexity.** What needs a bound is the
  number of independent conditions.

## Types: make illegal states impossible to write

```rust
#[derive(Clone, Copy, PartialEq, Eq)] pub struct LogicalAddress(pub u64);
#[derive(Clone, Copy, PartialEq, Eq)] pub struct PhysicalAddress(pub u64);
fn read_block(address: PhysicalAddress) -> Block;   // read_block(LogicalAddress(value)) does not compile

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
- **Replace a boolean parameter with an enum**: write `write(node, true)` as
  `write(node, Durability::Synced)`.
- **A field with an invariant is private** and can only come in through a constructor that checks it;
  a field without one is simply public, with no boilerplate getter / setter.
- **A type with an invariant does not implement `Default`.**
- **When constructing our own structs, write every field out**, not `..Default::default()`.
  Required fields are not left for a builder to check at runtime; they become constructor parameters,
  and only genuinely optional ones go into a builder.
- **No `as` for numeric conversions that can lose a value**: in a lossy direction use `try_from` and
  handle the failure, in a lossless one use `from`.

## Errors: the recoverable go into types, a broken invariant is asserted

- **Recoverable failures** (I/O errors, corrupted on-disk data, running out of resources) **go through
  `Result`**. The variants of an error enum are **split by the decision the caller has to make**, not
  expanded one per underlying cause: for example, what the layer above branches on may be only "not
  found", "corrupt", "no space" and "other I/O error", with the dozens of underlying OS errors carried
  as the source inside "other I/O error".
- **The criterion is whether the caller loses a decision it should be able to make.** A library's
  public boundary does not use a catch-all error type like `anyhow` or `Box<dyn Error>`; nor does it
  re-expand every underlying failure into its own variant at every layer.
  Tests and command-line tools may use a catch-all error type.
- **A broken invariant is a bug, and is asserted** — do not carry bad state onward, and never write it
  to disk.
- **No `unwrap()`; write `expect`**, with a message stating which invariant it relies on:
  `expect("journal head was verified in open(); None here means verification was skipped")`.

## Functions and nesting: cut by "independently verifiable", bound by path count

**A function's length is set by "one independently verifiable thing", not by screen height.** One thing
in three hundred lines is fine; two things squeezed into twenty lines get split.

**Nesting depth is not capped; path count is.** Loop nesting without early exits adds almost no paths
(each `break`, `continue` and `?` adds one); conditional nesting multiplies them exponentially (12 nested
independent `if/else` make 2¹² paths).
Any depth is fine, provided both hold: the path count is statable and bounded;
and each level's iteration target has a name and a type, not `a[i][j][k][l]`.

**Path count measures control flow only; it is not the whole of verification difficulty.** Over the
bound, split it; under the bound does not mean easy to verify. A loop must additionally state three things:
an upper bound on its iterations, the state it carries across iterations together with that state's
invariant (written as an assertion), and every early exit.
**"There is only one path" is no defence for an algorithm that is hard to verify.**

- **Parameters are not capped by count.** Each needs a name and a type; only when several parameters
  together form a concept with an invariant do they become one type — never a struct assembled just to
  shorten a signature.
- **Results go in the return type**, not back through `&mut` out-parameters.
- **Push side effects to the edge; write the core as pure functions**, and keep the I/O shell as thin as
  possible. A function that both queries and modifies says both in its name.
- **Early returns are fine.**

## Comments, assertions, constants, duplication

- **Comments say "why it is this way"**, not "what this does".
  Constraints go into types, names or assertions, not comments.
- **Doc comments state the contract a type cannot**: which errors it returns, when it panics, what an
  `unsafe` function requires of its caller. Doc comments that restate the name are deleted.
- **Every `unsafe` block has a `// SAFETY:` comment above it** saying why it is safe.
- **Invariants are written as assertions**, not only in docs.
- **Magic numbers become named constants**: a number has exactly one definition, with its unit in the
  name or the type.
- **Duplication is generated, never hand-copied.** Macros and `build.rs` are the proper way to generate;
  they only generate and do not hide logic, and what they generate has tests watching it.
- **Unused code is deleted**, commented-out code included.
  A `never used` from the compiler is treated as an error (`command-safety.md`: a
  warning is a free signal).
- **A `TODO` says what is missing and why it is not done now.** A check that is owed goes into the
  project kb's list of owed checks, and the code points there.

## Popular practices, examined one by one

Each one was put through the procedure in `machine-first.md`. There are only six **dispositions**:
dropped, relaxed, rewritten, kept, left to tooling, demoted to advice.

### Overall stance

| Popular practice | Source | Disposition | How we write it |
|---|---|---|---|
| Code is read more often than written; optimise for the reader | *Clean Code* and others | **rewritten** | Optimise for machine verification and information content; human attention goes to whether the test is right, whether the invariant is right, whether the basis holds, whether to accept the trade-off (`engineering-philosophy.md`) |
| Don't write clever code | common saying | **kept** | Follow it |
| KISS, keep it simple | common saying | **rewritten** | Simple means few control-flow paths, little state to verify, and every path written out — not few lines |
| YAGNI, don't write what you won't use | Extreme Programming | **kept** | Do not write unused code |
| Principle of least astonishment | common saying | **kept** | Names and types carry enough information for a model to guess the behaviour correctly |
| Convention over configuration | Rails and other frameworks | **dropped** | Do not rely on implicit default behaviour: write out every value you need |
| Separation of concerns; high cohesion, low coupling | common saying | **kept** | Clear boundaries and explicit contracts, so each piece can be verified on its own |
| Files and modules must not be too long | common saying | **relaxed** | Modules are cut by contract, not by line count |

### Naming

| Popular practice | Source | Disposition | How we write it |
|---|---|---|---|
| Names should be short, shorter the closer to the declaration | Go Code Review Comments, Linux kernel coding style | **dropped** | No length cap; see "Names" |
| Call the loop counter `i` and the temporary `tmp` | Linux kernel coding style | **dropped** | Name it for what it is: `stripe_index`, `unflushed_block` |
| Generic parameters `T`, `U`; lifetimes `'a` | Rust convention | **dropped** | Name them for their role: `Key`, `Value`, `'journal` |
| Abbreviations everyone knows are fine | common saying | **dropped** | Spell them out; a domain abbreviation may be used once registered |
| Names must be pronounceable | *Clean Code* | **kept** | What matters is no ambiguity and searchability |
| Names must be searchable | *Clean Code* | **kept**, and stronger | One name per semantic concept, and one search across the repository finds them all |
| Hungarian notation, `m_` prefixes, `I` on interface names | old C / C++ / C# conventions | **dropped**; what it wanted goes to types | A different meaning is a different newtype; a unit the type cannot express goes into the name |
| No noise words like `Manager`, `Helper`, `Data`, `Info` | *Clean Code* and others | **kept** | If deleting the word takes nothing away from the name, do not use it |
| Don't repeat the module name in the type name (`block::Address`) | clippy's `module_name_repetitions`, Go package naming conventions | **dropped** | A name stands on its own: write `BlockAddress` |
| Getters have no `get_` prefix | Rust API Guidelines C-GETTER | **kept** | Write them per C-GETTER |
| Conversion methods are named `as_`, `to_`, `into_` | Rust API Guidelines C-CONV | **kept** | `as_` is borrowed to borrowed and conventionally near-free; `to_` is conventionally expensive and usually yields a new value, though some are borrowed to borrowed, like `to_str`; `into_` consumes ownership. The C-CONV table is the authority, and an implementation must deliver the cost class its name promises |
| Boolean names start with `is_`, `has_` | common saying | **kept** | Say which side is true |
| Types are nouns, methods are verbs | *Clean Code* | **kept** | A noun says what it is, a verb says what it does |
| Rust's case conventions (`snake_case`, `CamelCase`) | Rust convention, rustc default warnings | **kept** | Follow the conventions |
| Shadowing (`let text = text.trim()`) is idiomatic | Rust convention | **rewritten** | Shadow when the meaning stays the same; a new meaning gets a new name. clippy stops shadowing with an unrelated meaning |
| Test functions are called `test_<function under test>` | common convention | **dropped** | State the scenario and the expected outcome |

### Functions

| Popular practice | Source | Disposition | How we write it |
|---|---|---|---|
| Functions short: a screen or two, ideally under 20 lines | *Clean Code*, Linux kernel coding style | **kept** | Cut by "one independently verifiable thing", not by line count |
| Do one thing | *Clean Code* | **kept** | One function, one independently verifiable contract |
| One level of abstraction per function | *Clean Code* | **relaxed** | Not required; what matters is a clear contract and a bounded path count |
| Read top-down (the stepdown rule) | *Clean Code* | **dropped** | Functions need not be ordered top-down by call order |
| At most three parameters | *Clean Code* | **relaxed** | Each parameter needs a name and a type; they become one type only when they form a concept |
| No flag arguments | *Clean Code* | **kept** | Use an enum |
| No output arguments | *Clean Code* | **kept** | Results go in the return type |
| No side effects; functional core, imperative shell | *Clean Code*, Gary Bernhardt | **kept** | Write the core as pure functions; I/O is pushed into the shell, which is verified by the project's own rigs |
| Separate queries from modifications | Bertrand Meyer (CQS) | **kept** | A function that both queries and modifies says both in its name |
| Single exit | structured programming | **dropped** | Early returns are fine |
| No more than three levels of nesting | Linux kernel coding style | **relaxed** | No cap on depth; control-flow path count is what is bounded, and loop state is verified separately |
| Cyclomatic complexity below some number | McCabe, static-analysis tools | **rewritten** | The arms of an exhaustive `match` do not count; what is bounded is the number of independent conditions |

### Types, data and encapsulation

| Popular practice | Source | Disposition | How we write it |
|---|---|---|---|
| Value objects instead of primitive types | *Refactoring* (Primitive Obsession) | **kept**, and stronger | A different meaning is a different type |
| Make illegal states unrepresentable | Yaron Minsky | **kept** | See "Types" |
| Parse, don't validate | Alexis King | **kept** | Verify once at the boundary, turn it into a type that carries the proof, and trust the type from there on |
| Private fields with getters / setters | Java-family encapsulation convention | **rewritten** | Fields with an invariant are private, fields without one are simply public |
| Immutable by default | Rust convention, functional programming | **kept** | Follow it |
| No global mutable state; no singletons | common saying; *Design Patterns* as a cautionary example | **kept** | Do not use them |
| Derive `Default` whenever you can | Rust convention | **rewritten** | A type with an invariant does not implement `Default` |
| Struct update syntax `..Default::default()` | Rust convention | **dropped for our own structs** | Write every field out |
| Many fields means a builder | Rust API Guidelines C-BUILDER | **rewritten** | Required fields are constructor parameters; only genuinely optional ones go into a builder |
| Numeric conversion with `as` | C-family convention | **dropped in lossy directions** | Use `try_from` and handle the failure; clippy stops it |
| Don't typedef a struct to a new name | Linux kernel coding style | **dropped** | Use newtypes: they wrap meaning, they do not hide structure |
| Name magic numbers | common saying | **kept** | A number has exactly one definition, with its unit in the name or type |

### Branches and abstraction

| Popular practice | Source | Disposition | How we write it |
|---|---|---|---|
| Replace conditional with polymorphism | *Refactoring* | **dropped on closed sets** | An `enum` with an exhaustive `match`; see "Branches" |
| Open/closed principle | SOLID | **dropped on closed sets** | Add a case and let the compiler force every piece of old code to be revisited |
| Single responsibility | SOLID | **kept** | One piece has one independently verifiable contract |
| Liskov substitution | SOLID | **kept** | A trait's contract holds for every implementation: contract tests run against every implementation (the same as "The positive control must run on **every** arm under test" in `test-discipline.md`) |
| Interface segregation | SOLID | **kept** | Cut traits small |
| Dependency inversion: depend on abstractions, not concretions | SOLID | **rewritten** | Extract a trait only with at least two real implementations; the model used for differential testing counts as one |
| Law of Demeter: talk only to immediate friends | Northeastern University's Demeter project | **relaxed** | Chain freely inside a module; never reach through another module's internals |
| Design patterns: factory, strategy, visitor | *Design Patterns* | **rewritten** | Strategy and visitor over a closed set are written as a `match`; add no layer for the pattern's sake |
| DRY, avoid duplication | *The Pragmatic Programmer* | **relaxed** | Duplication must be generated, never hand-copied |
| Abstract only after the third duplication | *Refactoring* (Don Roberts) | **kept** | Hand duplication to a generator first; do not extract a general path that eats the exhaustiveness check |
| A `default:` / `_ =>` fallback is safer | defensive programming | **dropped on closed sets** | Let the compiler catch the missed case; where unknown values are allowed, make "unknown" an explicit variant |

### Error handling

| Popular practice | Source | Disposition | How we write it |
|---|---|---|---|
| Fail fast | *The Pragmatic Programmer* | **kept** | Assert when an invariant is broken; never write bad state to disk |
| Library code never panics; always return `Result` | Rust convention | **rewritten** | The recoverable go through `Result`; a broken invariant is a bug and is asserted |
| A catch-all error type such as `anyhow`, `Box<dyn Error>` | Rust ecosystem convention | **dropped at a library's public boundary** | Variants are split by the decision the caller must make, with the underlying cause carried as the source |
| `unwrap()` when you are sure it cannot fail | Rust convention | **dropped** | Use `expect`, with a message naming the invariant it relies on |
| Defensive programming: check arguments everywhere | common saying | **rewritten** | Check once at the boundary, turn it into a type that carries the proof, and do not re-check inside |
| Error codes or exceptions | a debate across languages | **rewritten** | Failure is written into the return type |

### Comments and documentation

| Popular practice | Source | Disposition | How we write it |
|---|---|---|---|
| Write more comments | early convention | **rewritten** | Comments say only why |
| Comments are failures; good code needs none | *Clean Code* | **rewritten** | "What it is" goes to names and types; "why it is this way" is still written |
| Every public item gets a doc comment | rustc's `missing_docs` | **rewritten** | Delete the ones that restate the name; write only the contract a type cannot express |
| `unsafe` blocks get a `// SAFETY:` comment | Rust convention | **kept, and clippy enforces it** | Every `unsafe` block gets one |
| Keep commented-out code for later | common practice | **dropped** | Delete it |
| Record change history and authors in comments | old convention | **dropped** | History goes to git and the CHANGELOG (`design-doc-discipline.md`) |
| `TODO` comments | common practice | **rewritten** | Say what is missing and why it is not done now; a check that is owed goes into the kb |

### Formatting

| Popular practice | Source | Disposition | How we write it |
|---|---|---|---|
| No line longer than 80 columns | Linux kernel coding style, PEP 8 and others | **left to tooling** | `rustfmt`'s defaults decide; a long name wraps the line, and no name is shortened to fit |
| 8-column indentation to force shallow nesting | Linux kernel coding style | **dropped** | No cap on depth; see "Functions and nesting" |
| Vertical openness; related lines kept together | *Clean Code* | **left to tooling** | No hand-tuning |

### Test code

| Popular practice | Source | Disposition | How we write it |
|---|---|---|---|
| One assertion per test | *Clean Code* | **dropped** | One scenario per test; several assertions are fine, each with a message saying what it expects |
| Coverage must reach some percentage | common practice | **rewritten** | Argue with path counts and mutation lists (`test-discipline.md`) |
| Mock every dependency | common practice | **rewritten** | Prefer the differential-testing model and real implementations |
| The test pyramid: mostly unit tests | Mike Cohn | **rewritten** | Unit tests are fast feedback; acceptance rests on the final criterion the project sets (`show-me-test.md`) |
| Test code must be DRY too | common practice | **relaxed** | Each test reads as its own scenario; setup may move into helpers, assertions may not |
| A flaky test passes after a few reruns | common practice | **dropped** | Unstable is reported as unstable, never rerun until green (`test-discipline.md`) |
| Write the test first | Kent Beck (TDD) | **kept** | Watch it go red first (`show-me-test.md`) |

### How changes are made

| Popular practice | Source | Disposition | How we write it |
|---|---|---|---|
| The Boy Scout rule: tidy whatever you pass | *Clean Code* | **rewritten** | Tidying goes into its own commit, not mixed into a feature commit |
| Tolerate no broken windows | *The Pragmatic Programmer* | **kept, enforced by the gate** | Compiler and clippy warnings are treated as errors |
| Avoid premature optimisation | Knuth | **demoted to advice** | — |

Design- and process-level principles (unify the design, keep PRs small, and the ones kept because they
have nothing to do with who reads the code) are in `machine-first.md`.

## When not to apply them

**These rules hold unconditionally for things you can delete, and need a separate calculation for
things you cannot.** What is written into an external commitment — an on-disk format, a protocol, a
public API — gets its own criterion; you cannot get there by saying "explicit is safer".

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
| Whether a name says enough, whether one concept has one name, whether a function does one thing, whether a comment says why, whether error variants follow the caller's decisions, whether loop state is written out, `..Default::default()` | review |
| Names in shell scripts | **not yet a check**; `gate.sh` lists it among the unimplemented stages. ⚠️ This discipline applies to shell just the same, but the existing scripts in `scripts/` have never been swept; do not take them as models |

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
| `.claude/naming-lint-exclude` | directories or single `.rs` files not scanned, with the reason for excluding each; an entry pointing at a path that does not exist, or that excludes no `.rs` at all, is red. When sweeping old code, list the files not yet fixed one by one and delete a line as each is fixed: the exclusion only shrinks, and new files are checked |

Other names fixed from outside (field names of an external format and the like): write
`// naming-lint:external <why this name is not ours to decide>` on that line.
It exempts that one line only, and the reason may not be omitted either.
