<!-- generated-from: rules/verify-before-claiming.md sha256:78555e22c3feb89b0db8ed13d84c3a1eecb31a04033782e30888674447a5221a -->
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

## "Is it settled" and "what does it actually say" are two different questions

Checking the status column and confirming a decision is "settled" does not mean you
know what it settled on. Before using a decision to build a model, write a check, or
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

## Putting it into practice

- **Before rewriting any status line in the kb or a TODO, run that line's command.**
  Do not rewrite it unchecked — the old line at least carries a date; rewriting it
  gives a wrong statement a fresh timestamp.
- If you cannot say "this is the command I learnt it from", the sentence must be
  written as **conjecture**, not as fact.
- Factual corrections offered by the user **also get checked** — neither accepted
  wholesale nor argued with; check, then state the result plainly.
