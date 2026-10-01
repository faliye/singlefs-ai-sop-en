<!-- generated-from: rules/test-discipline.md sha256:ed6319c55a61cd8cd3e9088bb6364ecfbe4baa96960686b59303b58c3da91430 -->
<!-- doc-lint:rule-definition -->
# Testing discipline

## A single observation does not count

N≥5 rounds. To call it "pass" every round must pass; to call it "fail" every round
must fail; if neither holds, report "unstable" honestly and draw no conclusion.

The precondition is that the observation itself varies: real I/O, concurrency, timing,
randomness — at least one must be present. With none of them the thing under test is a
deterministic model, and the only phrasing allowed is "running N rounds proves there is no
hidden state, not statistical stability"; its evidence strength comes from mutation testing
and from assertions that pin down absolute values, not from the round count.

## Could not read ≠ read zero

If a metric comes back empty, retry and **void the whole round**. Never let it enter
the judgement as a zero.

## Time phases inside the process under test, not in the loop that relays its output

When a wrapper starts a child process, reads its output lines and relays each one onward, **do not use the moment each line was read as a phase clock**.

**What to do**: time inside the process under test and write the number into its result line. If arrival times must be used, read the child's output to the end first and relay afterwards.
When both an outer and an inner number exist, report a containment self-check: the outer phase must contain the inner one, and rounds where it does not are left out of the statistics.

## Before an experiment runs, the answer must not already exist

**Do not presuppose a conclusion, and do not consult any existing conclusion while
designing one** — this project's conclusion from last round, a settled result from
elsewhere, your own intuitive expectation: none of them.

**What you must read is the definition, not the answer.** The basis, parameters and
semantics of the thing under test have to be read verbatim (`verify-before-claiming.md`,
"whether it is settled and what it actually says are two different questions");
"last round measured X" and "project Y says this way is faster" are answers, and are not read.

**What to do**: pin down criteria, thresholds and discard clauses before the run, then go
look at the existing conclusions. Where the two disagree, record both as they stand —
never go back and edit the criterion. **Edit it and it is a new experiment: re-run.**

**A positive control's answer must not sit anywhere the side under test can read it.**
When what is under test is an agent that reads the repository, the answer does not go into the script it is going to run, nor into records in the same repository;
storing only a hash is not enough either: an unsalted short hash over a small candidate space counts as plaintext.
An answer used for a blind test stays out of any repository the side under test can read, or is replaced with a stretch of history it has never seen; a control left in the repository serves only as a regression check (guarding against the tool being narrowed), not as a blind test.

## An experiment's failure clause must not make its conclusion unfalsifiable

**Do not write "the effect did not show up" wholesale as an implementation bug.** Keep the two
apart, and **run both controls**:

| Control | What it is for | Verdict when no difference shows |
|---|---|---|
| **Positive control**: a baseline where the effect is firmly established in the literature | Proves the measurement has discriminating power | **The implementation is broken, discard the round** |
| **Real baseline**: the design this project actually intends to build | Answers what the experiment is really asking | **A legitimate result, record it as such** |

**Every time you pin a failure clause, write the next sentence: "what observation would make
this fire".** If you cannot write that sentence, the clause's antecedent is either backwards or
the clause is vacuous; either way it counts as having no clause.

### Do not write a criterion as a conjunction; a threshold must not be a tautology of the arm's definition

- **Do not fold two quantities into one true / false**: one line per quantity, each with its own verdict; leave the conjunction to people.
- **A threshold must not be the definition of the arm under test**: before writing a threshold, put the definition of the arm
  under test beside it; a cell whose threshold follows straight from the definition does not count as a criterion.

## The positive control must run against **every** arm under test

The control loop must iterate over the full set of arms, not "just run it against the first one".
When adding an arm, change the control loop along with it.

## Comparing the arms only against each other cannot detect "every arm is wrong together"

**Next to every cross-arm assertion there must be an assertion that pins down an
absolute value.** "The three arms have equal overhead" is not enough; you also need
"the overhead is exactly N", with N derived by independent arithmetic.

**What to do**: once you have written a cross-arm assertion, ask yourself "if all three
arms were wrong **together**, who would notice". If you cannot answer, add one that
pins down an absolute value.

## An endpoint is not a trajectory: a quantity a clause feeds into a predicate must be reported as a trajectory

When some clause uses a quantity as the input of a stop / start / admission predicate, report at least three numbers:
**the peak after the initial supply is used up, the number of rounds it was positive, and the end value**. A report that
gives only the end value may not use trajectory words such as "never" or "always" in its body text.

## An experiment must state which decision it measures for, and when enough is enough

A preregistration that only says "what to measure" is not enough. For each quantity it must also say **which decision the quantity serves,
which value of it would flip that decision, and at what point that decision becomes decidable**. A quantity that cannot answer those three is not registered.

- **Write the choices separately from the conclusions.** When handing over choices, also write a list that contains only the questions: the candidates for each choice,
  what observation would flip the choice, and when enough has been measured. It carries no leaning and no numbers, so it can go unchanged to the side designing the experiment.
- **Each quantity in the registration maps to one row of that list**; a quantity that maps to no row is not registered.
- **Every hand-back carries a table**, one row per choice: "decidable / what is still missing / can the remaining quantities still flip it". Before dispatching again, name the rows still missing; when no row is missing, stop.
- **"Not run because the decision was already decidable" is a legitimate ending, not debt**, and the experiment status must be able to say so.

## Mutation testing proves the assertions can go red, not coverage

Mutation only proves "every piece of code with an assertion has been mutation-tested", **not**
"every piece of code the conclusion rests on has been tested".

Never write "zero blind spots"; write "all N mutations were caught" and nothing more.
And once the conclusion is written, go back and ask: **is the arithmetic this conclusion
comes from covered by an assertion?**

Two pitfalls silently disarm mutation testing:

1. **Write constant assertions as addition, not subtraction**: write `assert_eq!(A, B + 240)`,
   not `assert_eq!(A - B, 240)`.
2. **An equivalent mutant is not a blind spot; account for it separately.** For a mutation
   that agrees with the original on every input and is judged equivalent, pin the equivalence
   down as a test for the record, then substitute a mutation that really changes behaviour.

## The check itself can also be wrong

Before asserting, confirm the check **has discriminating power**: when verifying
"there are no invalid blocks", also confirm the scan actually read some blocks.

After writing a check, ask: if the thing under test really were broken, would this go
red? If you cannot answer, it is not finished.

## Which half the gate handles

**Two items of the testing discipline are checks**: change `crates/*/src/` or `crates/*/build.rs` and you must bring
tests, judged by `scripts/show-me-test.sh`; and a loop in the project's `.rs` and `.py` files that reads a child process's output (which directories are scanned follows `scripts/relay-timing-lint.py`) must not
both timestamp lines and relay them, judged by the stage "Relay timing" (`转发计时`, `scripts/relay-timing-lint.py`). One more sits on the unimplemented list that
`gate.sh` prints every time: the final criterion. It is set by the project (`show-me-test.md`), so only the project can wire it up in `.claude/gate.d/` and
declare `# gate-covers:`.
The project's own verification methods are registered in `.claude/gate-not-implemented.tsv` at the project root (one per line: key, what is missing, the reminder that still applies once covered); `gate.sh` merges them into the same list, and the project wires them up and declares them the same way.

**Everything else runs on people, and there is no plan to make it a check for now**:
whether the round count is enough, how a deterministic experiment is written, whether the
criteria were pinned before the run, whether both controls ran, whether a failure clause can
ever fire, whether an assertion pinning an absolute value sits beside the cross-arm one,
whether a quantity a clause feeds into a predicate was reported as a trajectory, whether a
mutant is equivalent, whether a check has discriminating power, whether a negative
result proves the path really executed, whether phases outside a relay loop were timed inside the process under test,
whether the side under test can read a positive control's answer, whether each registered quantity
maps to a decision it is meant to settle.

## A negative result must be separable from "the code never ran"

"Did not reproduce" does not mean "no problem". To claim "this path is correct", you
need a reading proving that path **actually executed**.
