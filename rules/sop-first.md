<!-- generated-from: rules/sop-first.md sha256:09b11d9c1c320674bd20ef1caef99c5ac26cc3f4c026d586e3acca2d5aab99bf -->
<!-- doc-lint:rule-definition -->
# SOP before code

**The spec and the gate outrank any line of implementation code.**

In any round of work, if "add one gate check" conflicts with "write one feature",
add the gate check first.

## Why

Because this project will be worked on by many people (and many models), and
**everyone's ceiling differs, but the floor can be constrained.** The thing that
constrains the floor is the SOP: rules that state what "done" means, and scripts
that turn "not done" red on the spot.

Wrong code can be torn up and rewritten next round; that cost is bounded.
**A missing piece of SOP leaves everyone without footing on that piece** — errors
accumulate unnoticed, and by the time they surface they have soaked into dozens of
commits. That cost is not linear.

## Corollaries

- **A script existing before the thing it tests is normal**, not backwards.
  A harness can be written and self-checked with no workload; plug the
  implementation in when it arrives.
- **The first reaction to a newly found trap is "how does this become a red line
  in the gate"**, not "which document does this go in". A reminder sentence stops
  nobody; a check that fails does.
- **Gate scripts need tests too**: change `scripts/` and you must construct an
  input that ought to be rejected, and confirm it really goes red. A gate that
  cannot check itself is decoration.
- SOP changes bump `VERSION`, so the project learns the rules moved the next
  time it runs the gate.

# The gate is a teaching instrument, not a sieve

**Assume by default that a submitter wants their patch to pass.**

When they are stopped, most of the time it is not that they did not want to test —
it is that they **did not know how to test.** On that premise, the gate's job is
not "keep bad things out" but **to state what "done" means clearly enough that
nobody has to guess.**

## Every rejection must carry a next step

A failure message that only says "not acceptable" leaves "what would have been
right" to guesswork — and people who can only guess will route around the gate, or
simply not submit. **Both outcomes are worse than letting the patch through.**

So: **when rejecting, use `howto` to state what to do next.**
Enforced by `scripts/gate-lint.sh` — every non-summary `bad` must be followed by a
`howto` within 4 lines, or the gate fails itself.

When you cannot write the `howto`, **do not add the check yet**: if you cannot
state the next step, the criterion behind the check is not clear to you either.

## Three properties of a good gate

1. **Runnable locally, and it agrees with the remote.** Submitters must be able
   to see the result before sending, not learn it from a rejection.
2. **Failure messages point at the rule, not just the symptom.** Let people learn
   the rule rather than paper over this one instance — otherwise the same problem
   comes back.
3. **Make the right thing easy.** Templates, skeletons, ready examples to copy are
   all part of the gate. The pair of litmus files in `litmus/` exists precisely so
   people can copy them.

## The compounding effect

State the rules clearly enough and newcomers pick up the rhythm within a few
rounds; development then goes *faster*, because **nobody has to guess whether
something counts as done.** Code quality rises across the board at the same time,
because nobody wants their work rejected — all they were missing was a clear path.

## Boundary

SOP-first does not mean unbounded SOP growth. **Before a rule gets in, ask "can
this become a check that fails?"** — if it cannot, it is most likely not thought
through yet; leave it in the kb as open, do not write it as a rule.
