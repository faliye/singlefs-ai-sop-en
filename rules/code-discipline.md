<!-- generated-from: rules/code-discipline.md sha256:8773ef2a4755189f8ffb27129f70aa1df1cf126850f810f50cdf919389e0e77c -->
<!-- doc-lint:rule-definition -->
# Code discipline

Follow this when writing code. The only thing relaxed is compression done so it would fit in a human head;
making meaning explicit is not relaxed at all.

This covers all the code and text we write in this project, in every language and every directory: product code, tests, experiment code, scripts, configuration and build files, and documents (README, design notes, kb, rules, skills, agent definitions, commit messages, gate failure messages).
The examples are in Rust; other languages follow the same criterion, written the way that language does it.
Documents follow two things: names they mention are formed by "Names"; their text follows the rule "Comments contain no abbreviations and refer to nothing by a code name".
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
| **No ordinals or positions as names** | Functions, variables, parameters, types, fields, constants, modules and files alike: names such as `layer0`, `stream_0`, `step2`, `phase_1`, `first_item`, which only say "which number", are not allowed; name what it does and what it handles: `stream_0` → `metadata_journal_stream`, `layer0` → `checksum_verification_layer`. Instances that truly differ only by position go into one collection, told apart by a typed index (`stream_index: StreamIndex`), not given numbered names one by one. A registered project number with its short name is used as is — see "The abbreviation registry" |
| **A new meaning gets a new name** | Shadow a name only when the meaning stays the same; when raw data becomes verified data, or a byte count becomes a block count, pick a new name |
| **One definition per name per scope** | Within one scope of one file (a module, a class body, a function body; for shell, the whole file), no function, method or class is defined twice under the same name. Before adding a definition, search that file for the name; on a clash, rename one or delete one. A property setter, `@overload`, and singledispatch `register` do not count |

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
name carries its short name here too (`kb-discipline.md`, item 5 "A number can serve as an index, never as a name"): write `e57_field_authority`, not a
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

## Comments, assertions, constants, duplication

- **Comments say "why it is this way"**, not "what this does".
  Constraints go into types, names or assertions, not comments.
- **A comment that points at a place in another file names a function, a heading or a unique anchor string, not a line number.**
- **Comments contain no abbreviations and refer to nothing by a code name.** Spell abbreviations out; the only exception is a registered domain abbreviation (see "The abbreviation registry").
  Do not use stand-ins that only give a letter, an ordinal or a position — `a`, `b`, `c`, `layer0`, `stream_0`, "layer 0", "the first item" — and do not shorten several things into a list like "a b c":
  write out what each one is and what it does, for example "the first item" becomes "the checksum item".
- **Doc comments state the contract a type cannot**: which errors it returns, when it panics, what an
  `unsafe` function requires of its caller. Doc comments that restate the name are deleted.
- **Every `unsafe` block has a `// SAFETY:` comment above it** saying why it is safe.
- **Invariants are written as assertions**, not only in documents.
- **Magic numbers become named constants**: a number has exactly one definition, with its unit in the
  name or the type.
- **Duplication is generated, never hand-copied.** Macros and `build.rs` are the proper way to generate;
  they only generate and do not hide logic, and what they generate has tests watching it.
- **Unused code is deleted**, commented-out code included.
  A `never used` from the compiler is treated as an error.
- **A `TODO` says what is missing and why it is not done now.** A check that is owed goes into the
  project kb's debt table, and the code points there.

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
| The same name defined twice in one scope of a `.py` or `.sh` file | `scripts/duplicate-definitions.py` (gate stage "同名定义"): scans the whole repository, skipping frozen evidence; for shell it judges only definitions written at column 0. When a later definition overriding an earlier one is intended, write `# duplicate-definitions:allow <reason>` on the later definition's line. An assignment overriding a function, an imported name overridden by a def, and same names across files are beyond it, left to review |
| Warnings are errors | clippy `-D warnings` |
| Whether a name says enough, whether one concept has one name, whether a function does one thing, whether a comment says why, whether a comment cites a line number in another file, whether a name or a comment uses an ordinal or a position as a stand-in, whether a comment contains an abbreviation, whether error variants follow the caller's decisions, whether loop state is written out, `..Default::default()` | review |
| Names, abbreviations and stand-ins in code other than `.rs` (shell, python and the like) and in documents | **not yet a check**; `gate.sh` lists it among the unimplemented stages (key 「命名纪律（shell）」) |

The clippy ones run inside `scripts/check.sh`, that is, the gate's "Build and unit tests" stage; that stage does not run when the project root has no `Cargo.toml`.

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
