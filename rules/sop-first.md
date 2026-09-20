<!-- generated-from: rules/sop-first.md sha256:9a597f82725a7c8f253445168dfb6a2df8f2c7d5c3baff57ed99b0b7831bc8f1 -->
<!-- doc-lint:rule-definition -->
# SOP before code

**The spec and the gate outrank any line of implementation code.**

In any round of work, if "add one gate check" conflicts with "write one feature",
add the gate check first.

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

So: **when rejecting, state what to do next.**
Enforced by `scripts/gate-lint.sh`, which covers every shape of rejection in the table
below:

| Shape | Where the remedy goes | Criterion |
|---|---|---|
| `bad` | the `howto` right after it | a `howto` within 5 lines counting the `bad` itself (comment lines do not count) |
| `die` | **its own second argument** (`die "what blocked" "what to do"`) | a `die` carrying only one argument fails |
| a printed `✗` (an `echo`, or a `print` in embedded python) | the `→` line before the next rejection | no `→` in between fails |

The `die` row is not an afterthought: `die` is `bad` + `exit`, so it is a rejection
too — and a rejection as remedy-free as `die "unit tests failed"` is one that checking
only `bad` cannot find.

The third row covers **a project's own local stages**. Most of them do not source
`lib.sh`; they `echo "  ✗ …"` directly, or put the criterion inside embedded python:
`print('  ✗ …')` — neither of the first two rows reaches them. Measured on singlefs's
local stages: of 46 such rejections, 14 had no next step at all.

The window is not 5 lines here: python often prints one `✗`, then loops through the
offending items, and only then gives the remedy — 5 lines would misjudge the whole
batch. So the criterion is "the remedy appears **before the next rejection**". Both
exemptions must be **written out explicitly**; nothing gets guessed. A per-item line
inside a loop carries `# gate-lint:detail` (its remedy lives on the summary), and a
summary line carries `# gate-lint:summary`.

When you cannot write the `howto`, **do not add the check yet**: if you cannot
state the next step, the criterion behind the check is not clear to you either.

## Three properties of a good gate

1. **Runnable locally, and it agrees with the remote.** Submitters must be able
   to see the result before sending, not learn it from a rejection.
2. **Failure messages point at the rule, not just the symptom.** Let people learn
   the rule rather than paper over this one instance — otherwise the same problem
   comes back.
3. **Make the right thing easy.** Templates, skeletons, ready examples to copy are
   all part of the gate.

## The compounding effect

State the rules clearly enough and newcomers pick up the rhythm within a few
rounds; development then goes *faster*, because **nobody has to guess whether
something counts as done.** Code quality rises across the board at the same time,
because nobody wants their work rejected — all they were missing was a clear path.

## Boundary

SOP-first does not mean unbounded SOP growth. **Before a rule gets in, ask "can
this become a check that fails?"** — if it cannot, it is most likely not thought
through yet; leave it in the kb as open, do not write it as a rule.
