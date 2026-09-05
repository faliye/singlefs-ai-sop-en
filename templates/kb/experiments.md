<!-- generated-from: templates/kb/experiments.md sha256:6e50711dc72aae68335f519cb53285dc772218ae92f14c07bd53ab57c8f3116a -->
# Experiment record

Each experiment answers one question. **Criteria, thresholds and voiding clauses are
fixed before the run** (`rules/test-discipline.md`, "the answer must not already exist
before the experiment runs"); only after they are written do you go and look at existing
conclusions. Changing a criterion after the run makes it a new experiment — rerun.

---

## E1 <name, 24 characters or fewer> —— not yet run

<!-- This heading is the registration site for the number: the segment between the
     number and the "——" is its short name. Every citation elsewhere is written
     `<number> (<short name>)`; a bare number is not allowed. Enforced by doc-lint.sh. -->

**Question**: <one sentence. Name which item of which decision it would establish or
overturn.>

**Prerequisites**: <which **concrete input** is missing — which number, which field
table, which settled decision. Not "module X is owed": the criterion is often pure
arithmetic and needs no such module, and writing it coarsely shelves an experiment that
could have run today.>

**Criteria** (fixed first, before looking at any existing conclusion):

| # | What is judged | Threshold | Voiding clause |
|---|---|---|---|

**Controls**:

- Positive control: <a baseline where the effect is firmly established, run against
  **every** arm. No difference measured → the implementation is broken, void the round.>
- Real baseline: <the scheme this project actually intends to implement. No difference
  measured → a legitimate result, record it as it is.>
- Absolute-value assertion: <at least one assertion pinning an absolute number, to catch
  "every arm is wrong together".>

**Mutations**: <the list of ways the code under test was broken, one line each:
what was changed → which assertion went red.>

**Rerun**: <one command plus the output path.>

**Conclusion**: <written after the run. Numbers carry their measurement basis; "what
this number shows" is an inference and owes the argument it owes — it is not an
observation.>

---

## Revision history

### YYYY-MM-DD
- Created.
