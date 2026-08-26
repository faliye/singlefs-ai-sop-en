<!-- generated-from: rules/engineering-philosophy.md sha256:932cf6bac9928e782677de036e17d4fdb0a12abceee42f7301e8f01f00f57437 -->
<!-- doc-lint:rule-definition -->
# Engineering Philosophy

> **AI-friendly to implement, human-friendly to review.**

This is not a compromise between two goals. **They are two axes that always
should have been separate.**

They used to be conflated — because the people writing the code and the people
reading it were the same people, so "easy to write" and "easy to read" had to be
the same thing. **They no longer are.**

## Why the old commandments need re-deriving

Nearly all of today's software engineering best practices grew on one implicit
premise: **the ceiling of engineering quality is set by human cognitive capacity.**

Keep functions short, keep nesting shallow, minimise concepts, unify the design,
eliminate duplication — on the surface these say different things. Underneath
they solve one problem: **make it fit in a limited human head.**
They are not wrong. They are **calibrated for that constraint.**

And that constraint is loosening. Tracing logic, maintaining consistency,
covering cases exhaustively — these happen to be machine strengths, and machine
capacity passed the human ceiling some time ago.

**A floor calibrated to the old ceiling is now a floor set too low.**

So this project's stance toward established best practice is to **follow it
critically, not by default.** Each rule gets re-asked: what problem did it
originally solve, and does that problem still exist? The procedure is in
`machine-first.md`.

## What each axis optimises for

| | **Implementation: AI-friendly** | **Review: human-friendly** |
|---|---|---|
| Who uses it | models write it, models change it, machines verify it | humans judge, humans carry responsibility |
| Optimise for | machine-checkable, exhaustible, forcibly reachable, revertible | judgeable, trustworthy, trade-offs visible |
| Concretely | explicit exhaustive branches beat a clever general path; types make invalid states unrepresentable; invariants become assertions; generate duplication rather than hand-maintain it | decisions carry their basis; gate failures carry a next step; numbers carry their measurement basis; anything unverified is marked as such |

## AI-friendly ≠ unreadable

**The direction that is friendly to a model is "more explicit", not "more obscure".**

A model reading code with no names, no types and seven levels of nesting is no
better off than a person — it gets less information, makes more mistakes, and its
mistakes are harder to catch. **Cutting readability does not make an
implementation more AI-friendly; it makes both sides worse.**

Exactly one thing is being relaxed: **compression done so it would fit in a human
head** — requirements like "one mechanism for everything" and "the fewer concepts
the better", i.e. uniformity at the design level.

**Not relaxed** (these look like readability; they are verifiability or
information content):

| Item | Why it stays |
|---|---|
| Naming | **A name is information, not decoration.** `fn f(a: u64, b: u64)` carries a whole layer of meaning less than `fn commit_txn(gen: Generation, root: Logical)`, and a model cannot recover it either |
| One responsibility per function | a function does one **independently verifiable** thing — that sets its length, not screen height |
| **Bounded path count** | how many cases does exhaustive coverage of this code need? If you can state it and it is bounded → verifiable. **Note: path count, not nesting depth**, see below |
| Consistent conventions | inconsistent conventions defeat mechanical checking — this is a different thing from "design uniformity"; do not conflate them |

**The criterion is still the same one**: does this make verification easier, or
does it only make things easier on the eye? All four above are the former, so
they stay — **with their reasons rewritten in terms of verification and
information, no longer hung on "readability".**

### Nesting depth itself is not capped

Rules like "no more than three levels of nesting" measure the wrong thing. What
actually determines verifiability is **path count**:

| | Paths | Testable? |
|---|---|---|
| 12 nested `for` loops over a 12-dimensional structure | **1** | fully testable; depth does not matter |
| 12 nested independent `if/else` | **2¹²** | not testable |
| 3 nested independent `if/else` | 8 | barely |

**Loop nesting adds almost no paths; conditional nesting multiplies them
exponentially.** The old rule capped both together because both are equally hard
on a human to read — that is a human constraint, not a verification constraint.

So: **any depth is fine, provided (1) the path count is statable and bounded, and
(2) the meaning of each level is explicit.** The second means each level's
iteration target has a name and a type, not `a[i][j][k][l]` — the problem with
that form is not that it is deep, it is that it **carries no information**, which
is the same rule as "a name is information".

### An example: where the meaning should live

Three ways to write the same thing, worst to best:

```rust
// 1. Meaning in a comment — the traditional form, and the worst
// dim0: ethnicity  dim1: gender  dim2: age band
pop[i][j][k]

// 2. Meaning in the names — long-winded by old taste, but machines like it
num_china_people[han][woman][teenager_18_to_24]

// 3. Meaning in the types — best; getting a dimension wrong will not compile
num_china_people[Ethnicity::Han][Gender::Woman][AgeBand::T18to24]
```

The first is **compression done so it would fit in a human head**: names cut to
the shortest, meaning moved into a comment. In the human era that was reasonable —
line width was limited, screens were limited. But **a comment can drift from the
code; a name cannot**, because the name *is* the code.

The second used to draw "too long — you can see it is a 3-D array, why spell it
out". That is human taste. To a machine it is pure gain: every dimension is
self-describing, and **you can judge whether an access is correct without any
context**.

The third pushes the information from "readable" to "checkable" — **swap two
dimensions and it will not compile**. This is the same rule as `machine-first.md`'s
"prefer types that make invalid states unrepresentable over comments explaining
constraints", and it shares a root with this project's newtype discipline
(logical address / physical address / generation are each their own type).

**Criterion**: does this piece of meaning live in a comment, in a name, or in a
type? **Later is better, because later is harder to drift from the implementation.**

## Corollary: where human attention should go

**Human attention is the most expensive thing in this system, so it should not be
spent reading code** — that job can go to a model.

It should be spent on the four things a machine cannot judge:

1. **Is this test testing the right thing?**
2. **Is this invariant itself correct?**
3. **Does the basis for this decision hold?**
4. **Do we accept this trade-off?**

None of the four requires reading the implementation end to end, but all four
require **the surface presented to humans to be human-friendly**.

So: gate output, kb conclusions, decision records, failure messages — **these must
be optimised for humans, and harder than before**, because they are now the only
things a human will look at. Code that is hard to read is an acceptable price;
those four being hard to judge is not.

## The line between the two axes

> **Gate proves evidence requirements, not semantic correctness.**

That sentence is the line: **machines prove evidence requirements; humans judge
semantic correctness.** Below it is the implementation axis; above it, review.

## Where this lands in this SOP

| Axis | Lands in |
|---|---|
| Implementation | `machine-first.md`, `kb-discipline.md`, plus each project's own design discipline |
| Review | the howto requirement in `show-me-test.md`, `design-doc-discipline.md`, the basis requirement for decision records |
| The line | `show-me-test.md`, "what the gate can and cannot prove" |
