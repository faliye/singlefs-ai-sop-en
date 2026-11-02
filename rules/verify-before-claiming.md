<!-- generated-from: rules/verify-before-claiming.md sha256:db37e18c415d03849bf43d1a3f1ebae1c41e6f65ade2029dc9989917db7ba0d6 -->
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
verbatim into your notes or experiment comments. If you cannot paste it, do not
derive from it.

## You checked the narrow claim and then stated the broad one

⇒ **The test: the sentence you are about to say — are its subject and scope exactly the
ones you just checked?**

- You checked "this path is blocked" ⇒ you may say "this path is blocked", **not "it
  cannot be done"**.
- You checked "this file does not say it" ⇒ you may say "this file does not say it",
  **not "nothing in the repository says it"**.

⇒ **What to do**: take the broad sentence as a proposition to be proved and ask
**"what cases would I have to rule out for this to be true"**, then rule them out one
by one. If you cannot finish, shrink the sentence back to the range you actually
finished.

⚠️ Before judging "is this object covered by a clause", **grep its name across the whole repository**
and see whether it has been placed in some existing class.

⚠️ **Truncated output is a narrow claim too**: `grep … | head -12` shows the first 12 lines, not every match.
⇒ **Before saying "only these places" or "these are all the matches", do not truncate the output**; if it is long,
count first (`grep -c`, `| wc -l`), and read the contents once the count matches.

## Relaying another place's convention carries its source or its original words

In a prompt sent to a subagent or a message sent to another session, a sentence that says "same convention as X", "do it the way X does", or "same as `X`" carries, in that same sentence, one of these:

- `path:line` (or a path, or "same file", together with "line N");
- a quotation of the original: at least 6 characters inside 「」, or the sentence ends in a colon and the next line starts with `>`;
- a path together with a function name in backticks, or a heading inside 「」.

When unsure what the original says, read it before dispatching. Writing only your own understanding of it ("same convention: take the qualified name") is not a source.

## Which half the gate handles

The hook `scripts/claude-hooks/borrowed-convention-guard.sh` (judgement in `scripts/borrowed-convention.py`) handles "Relaying another place's convention carries its source or its original words": it judges before dispatching a subagent (Agent, Task) and before SendMessage, and refuses when a borrowing sentence has neither a source nor the original words in that sentence.
The borrowing phrases it recognises are those in `BORROW_PHRASES` in `scripts/borrowed-convention.py`; phrases outside it are not judged. Whether the source points to the right place, or the original was copied correctly, it does not judge.
A project registers it in `.claude/settings.json`; how is in the hook's file header.

The remaining items are not checks. `scripts/doc-lint.sh` reaches only the written side (a kb entry's source and status, whether
a number carries its short name); it cannot check whether a sentence was just verified. "Check now, before you say it"
runs entirely on people (`show-me-test.md`: the gate proves that evidence requirements are met, not semantic correctness).

## Putting it into practice

- **Before rewriting any status line in the kb or a TODO, run that line's command.** Do not rewrite it unchecked.
- If you cannot say "this is the command I learnt it from", the sentence must be
  written as **conjecture**, not as fact.
- Factual corrections from someone else **also get checked** — neither accepted
  wholesale nor rushed to rebut; check, then state the result plainly.
