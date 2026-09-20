<!-- generated-from: rules/verify-before-claiming.md sha256:26d01dd4ba95a7aaad3e1d8af23618cec11c599456feca1164a561e94f580562 -->
<!-- doc-lint:rule-definition -->
# Check now, before stating external state

**Anything you are about to say about "what the outside world looks like right now"
must be confirmed by running a command on the spot.**
Memory, the kb, the previous turn of conversation, a TODO table — all are leads, none
are evidence.

"The outside world" = everything not in this conversation that may have been changed
without my knowing:

| What you are about to say | The command to run now |
|---|---|
| whether the gate passes right now | run `scripts/gate.sh`; do not infer it from the last run |
| whether some invariant is implemented | read the status column in `kb/invariants.md`, then grep the checker source to confirm it is really there |
| whether some decision is settled | read `kb/decisions.md` and see which of settled / partly settled / open it is |
| whether the toolchain and environment are complete | run `scripts/env.sh` |
| how another filesystem does something | check its documentation or source now, and note in the kb both the source and "not verified in this project" |
| the current state of a test image | run the checker now; do not rely on "it was fine last time" |

**What does not need checking now**: files read earlier in this same conversation,
pure code-logic derivation, arithmetic.

## Whether it is settled and what it actually says are two different questions

Checking the status column and confirming a decision is "settled" **does not mean you
know what it settled on**. Before using a decision to build a model, write a check, or
overturn some other conclusion, **you must read its definition word for word** — not
model it from memory.

Observed: an experiment set out to test whether some decision would break under a new
scenario. The model was built from two possible readings of the decision's wording,
both readings showed a failure, and the decision was overturned on that basis. **The
decision's own text already spelled out a third reading** — under it the error rate is
exactly zero, so the conclusion pointed the wrong way entirely. In hindsight: the
status column got checked, the definition never got read.

**What to do**: before using a decision in any derivation, paste its defining sentence
verbatim into your notes or experiment comments. If you cannot paste it, you were
working from memory.

## You checked the narrow claim and then stated the broad one

This one is the hardest to catch yourself, because **the "check now" step really did
happen**: you ran the command, you read the file, you are holding a proposition you
genuinely verified. **The error is that the sentence you then said is wider than it.**

Both have the same shape: **what was verified is one path / one file; what was asserted
is all paths / the whole repo.**

⇒ **The test: the sentence you are about to say — are its subject and scope exactly the
ones you just checked?**

- You checked "this path is blocked" ⇒ you may say "this path is blocked", **not "it
  cannot be done"**.
- You checked "this file does not say it" ⇒ you may say "this file does not say it",
  **not "nothing in the repo says it"**.

⇒ **What to do**: take the broad sentence as a proposition to be proved and ask
**"what cases would I have to rule out for this to be true"**, then rule them out one
by one. If you cannot finish, shrink the sentence back to the range you actually
finished — **a narrowed sentence is still useful; a wide false one becomes the
foundation of the whole round.**

⚠️ "Is this object covered by a clause" is especially prone to it: **a clause can live
elsewhere and cover it by class**, while its own section says nothing at all. ⇒ Before
judging, **grep its name across the whole repo** and see whether it has been placed in
some existing class — **the class rule answers for it.**

⚠️ **Truncated output is a narrow claim too**: `grep … | head -12` shows the first 12 lines, not every match.
⇒ **Before saying "only these places" or "these are all the matches", do not truncate the output**; if it is long,
count first (`grep -c`, `| wc -l`), and read the contents once the count matches.

## Which half the gate handles

**Not one item in this rule is a check.** "Check now, before you say it" is a behaviour,
not a textual form: the same sentence looks identical in the file whether it was checked on
the spot or written from memory, and no machine can tell the two apart.
`scripts/doc-lint.sh` reaches only the written side (a kb entry's source and status, whether
a number carries its short name); it cannot check whether a sentence was just verified.

So this rule runs entirely on people. It is written here not as an excuse but so that "the
gate is all green" is not read as "this rule was kept" (`show-me-test.md`: the gate proves
that evidence requirements are met, not semantic correctness).

## Putting it into practice

- **Before rewriting any status line in the kb or a TODO, run that line's command.**
  Do not rewrite it unchecked: rewriting it vouches afresh for a statement nobody has checked,
  and readers will take it as just checked.
- If you cannot say "this is the command I learnt it from", the sentence must be
  written as **conjecture**, not as fact.
- Factual corrections from someone else **also get checked** — neither accepted
  wholesale nor argued with; check, then state the result plainly.
