<!-- generated-from: rules/engineering-philosophy.md sha256:8317634233439a8267e28bd8ff87a194bed30eca09763ac065ef6a1be18a5a7d -->
<!-- doc-lint:rule-definition -->
# Engineering Philosophy

> **AI-friendly to implement, human-friendly to review.**

They are two separate axes, each optimised on its own, with no compromise struck between them.

## What each axis optimises for

| | **Implementation: AI-friendly** | **Review: human-friendly** |
|---|---|---|
| Who uses it | models write it, models change it, machines verify it | humans judge, humans carry responsibility |
| Optimise for | machine-checkable, exhaustible, forcibly reachable, revertible | judgeable, trustworthy, trade-offs visible |
| Concretely | explicit exhaustive branches beat a clever general path; types make invalid states unrepresentable; invariants become assertions; generate duplication rather than hand-maintain it | decisions carry their basis; gate failures carry a next step; numbers carry their measurement basis; anything unverified is marked as such |

## AI-friendly ≠ unreadable

**The direction that is friendly to a model is "more explicit", not "more obscure".**

Exactly one thing is being relaxed: **compression done so it would fit in a human
head** — requirements like "one mechanism for everything" and "the fewer concepts
the better", i.e. uniformity at the design level.

**Naming, one responsibility per function, bounded path count and consistent conventions are not relaxed**;
they are kept for verifiability and information content:

| Item | How far it is kept |
|---|---|
| Naming | meaning goes into the name, none left out |
| One responsibility per function | a function does one **independently verifiable** thing — that sets its length, not screen height |
| **Bounded path count** | you can state how many cases exhaustive coverage of this code's control flow needs, and the number is finite. **It is path count, not nesting depth** — and it is a necessary condition for verifiability, not a sufficient one |
| Consistent conventions | conventions stay consistent. This is a different thing from "design uniformity"; do not conflate them |

**The criterion is the one in `machine-first.md`**: a rule that makes the code easier to verify mechanically
stays; one that only makes it easier on the human eye can go.
**How this lands in code is spelled out in `code-discipline.md`**: no length cap on names,
no abbreviations, no single letters; no cap on nesting depth, but a bounded path count;
meaning that can go into a type does not go into a name, and meaning that can go into a name does not go into a comment.

## Corollary: where human attention should go

Human attention is not spent reading code; reading code goes to a model. Human attention
is spent on the four things a machine cannot judge:

1. **Is this test testing the right thing?**
2. **Is this invariant itself correct?**
3. **Does the basis for this decision hold?**
4. **Do we accept this trade-off?**

Gate output, kb conclusions, decision records and failure messages are optimised for humans,
so that these four can be judged without reading the implementation end to end.
Code that is hard to read is an acceptable price; those four being hard to judge is not.

## The line between the two axes

> **Gate proves evidence requirements, not semantic correctness.**

**Machines prove evidence requirements; humans judge semantic correctness.**
Below the line is the implementation axis; above it, review.

## Where this lands in this SOP

| Axis | Lands in |
|---|---|
| Implementation | `machine-first.md`, `code-discipline.md`, `kb-discipline.md`, plus each project's own design discipline |
| Review | the howto requirement in `sop-first.md`, `design-doc-discipline.md`, the basis requirement for decision records |
| The line | `show-me-test.md`, "what the gate can and cannot prove" |
