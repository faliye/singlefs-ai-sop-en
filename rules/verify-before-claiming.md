<!-- generated-from: rules/verify-before-claiming.md sha256:d608d7d06438fed349270152484cacaf54f0acaa031e23cf99529c090c8edc03 -->
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
| whether the gate passes right now | cite the summary of the most recent `gate.sh --staged` run (with its date and the hash of that tree); if anything changed after it, do not say "passes" — say "green last time; these later changes have not been run". The gate runs only at commit time; do not rerun it just to say this |
| whether some invariant is implemented | read the status column in `kb/invariants.md`, then grep the source of the check that implements it to confirm it is really there |
| whether some decision is settled | read `kb/decisions.md` and see which of settled / partly settled / open it is |
| whether the toolchain and environment are complete | run `scripts/env.sh` |
| how another implementation does something | check its documentation or source now, and note in the kb both the source and "not verified in this project" |
| the current state of the system under test or its artefacts | run the project's checks now; do not rely on "it was fine last time" |

**What does not need checking now**: files read earlier in this same conversation,
pure code-logic derivation, arithmetic.

## Whether it is settled and what it actually says are two different questions

know what it settled on**. Before using a decision in any derivation (building a model,
writing a check, overturning some other conclusion), paste its defining sentence
verbatim into your notes or experiment comments. If you cannot paste it, you were
working from memory.

## You checked the narrow claim and then stated the broad one

⇒ **The test: the sentence you are about to say — are its subject and scope exactly the
ones you just checked?**

- You checked "this path is blocked" ⇒ you may say "this path is blocked", **not "it
  cannot be done"**.
- You checked "this file does not say it" ⇒ you may say "this file does not say it",
  **not "nothing in the repo says it"**.

⇒ **What to do**: take the broad sentence as a proposition to be proved and ask
**"what cases would I have to rule out for this to be true"**, then rule them out one
by one. If you cannot finish, shrink the sentence back to the range you actually
finished.

⚠️ Before judging "is this object covered by a clause", **grep its name across the whole repo**
and see whether it has been placed in some existing class.

⚠️ **Truncated output is a narrow claim too**: `grep … | head -12` shows the first 12 lines, not every match.
⇒ **Before saying "only these places" or "these are all the matches", do not truncate the output**; if it is long,
count first (`grep -c`, `| wc -l`), and read the contents once the count matches.

## Which half the gate handles

**Not one item is a check.** `scripts/doc-lint.sh` reaches only the written side (a kb entry's source and status, whether
a number carries its short name); it cannot check whether a sentence was just verified. "Check now, before you say it"
runs entirely on people (`show-me-test.md`: the gate proves that evidence requirements are met, not semantic correctness).

## Putting it into practice

- **Before rewriting any status line in the kb or a TODO, run that line's command.** Do not rewrite it unchecked.
- If you cannot say "this is the command I learnt it from", the sentence must be
  written as **conjecture**, not as fact.
- Factual corrections from someone else **also get checked** — neither accepted
  wholesale nor rushed to rebut; check, then state the result plainly.
