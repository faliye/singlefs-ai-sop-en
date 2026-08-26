<!-- generated-from: rules/session-wrapup.md sha256:caf236a582896179c8a72828f743c3b9a61565a2ace3e7ea875acae7f76a9684 -->
<!-- doc-lint:rule-definition -->
# Wrap-up: required before the end of every round of work

## 0. Report progress — first, and never skipped

One line, three numbers: **which milestone we are at / whether the gate passes / what
evidence is still missing.** Status is always checked now (run `gate.sh`), never
copied from the previous round's notes.

**State at the same time how much this round added to that number.** Often it is 0: a
null result, a decision overturned, or merely establishing that some piece of evidence
is not yet obtainable. **Then write 0.** Counting other rounds' results into this one
turns the progress table into something that only ever goes up, and it stops being
useful.

## 1. Any script or command typed this round that should move into `scripts/`?

The criterion is "**will it get copied a second time**", not "is it well written".

**Turn traps you have hit into checks that fail, not into reminder sentences.**
Writing "careful not to do Y while X" stops nobody typing commands by hand; a check
that **refuses to run** at that moment does.

Look the other way too: is anything in there no longer used? Delete it.

## 2. Any skill or rule used this round that should be changed?

Was any step held together by memory? Was a criterion added only after the fact? Was
the same boilerplate copied a third time?
**If so, change that file now**, not "next time".

Where to change it — ask "**would another project need this too?**":

- Yes → change singlefs-ai-sop (`rules/` / `skills/` / `scripts/`) and bump `VERSION`
- No → change the project's own `kb/` or the project's `CLAUDE.md`

## 3. Did any decision change?

If this round overturned or settled any design decision, **write it into
`kb/decisions.md` right away**, with what overturned it.
One missing decision record means that in three months "why did we decide this?" needs
archaeology to answer.
