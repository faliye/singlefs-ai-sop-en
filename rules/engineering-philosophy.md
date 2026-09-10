<!-- generated-from: rules/engineering-philosophy.md sha256:0be2426b15bd92d9b142e18142e0abb13d0154de08fc9725202faa29df1fc588 -->
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

**"Machine capacity passed the human ceiling" rests on two falsifiable premises**
— the order of magnitude of the context window, and that concurrent interleavings
can be decided exhaustively. The premises themselves, their calibration, and what
you would have to observe to overturn them are in `machine-first.md`, under "What
this stance assumes, and how to overturn it". They are not repeated here: write
one premise in two places and the two places will eventually disagree.

So this project's stance toward established best practice is to **follow it
critically, not by default.** Each rule gets re-asked: what problem did it
originally solve, and does that problem still exist? The procedure is in
`machine-first.md`; what it concluded is in `code-discipline.md`.

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
| Naming | **A name is information, not decoration**: meaning left out of a name is meaning a model cannot recover, even by reading the implementation |
| One responsibility per function | a function does one **independently verifiable** thing — that sets its length, not screen height |
| **Bounded path count** | how many cases does exhaustive coverage of this code's control flow need? If you cannot state it, or it is unbounded, it cannot be verified. **It is path count, not nesting depth** — and it is a necessary condition for verifiability, not a sufficient one |
| Consistent conventions | inconsistent conventions defeat mechanical checking — this is a different thing from "design uniformity"; do not conflate them |

**The criterion is still the same one**: does this make verification easier, or
does it only make things easier on the eye? All four in the table make verification
easier, so they stay — **with their reasons rewritten in terms of verification and
information, no longer hung on "readability".**
**How this lands in code is spelled out in `code-discipline.md`**: no length cap on names,
no abbreviations, no single letters; no cap on nesting depth, but a bounded path count;
meaning goes into a type when it can, into a name when it cannot, and never into a comment.

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
| Implementation | `machine-first.md`, `code-discipline.md`, `kb-discipline.md`, plus each project's own design discipline |
| Review | the howto requirement in `show-me-test.md`, `design-doc-discipline.md`, the basis requirement for decision records |
| The line | `show-me-test.md`, "what the gate can and cannot prove" |
