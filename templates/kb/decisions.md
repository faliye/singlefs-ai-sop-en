<!-- generated-from: templates/kb/decisions.md sha256:4c39fa5207446b386f96ad8a5e86f81aab51a707abfb6f565d1bae5a9bae7fe7 -->
# Design decision record

Each decision has exactly three states: **settled** / **half-settled** (direction fixed,
details open) / **undecided**.
The format and the hard requirements are in the `decide` skill. When overturning a
decision, edit the body directly; if this file is registered in `.claude/history-carriers` at the project root (without that registry, every file counts),
put the basis into the closing "## Revision history", and if it is not registered, the history lives in git.

---

## D1 <decision name, display width ≤ 48, CJK = 2> —— undecided

<!-- This heading is the registration site for the number: the segment between the
     number and the "——" is its short name. Every citation elsewhere is written
     `<number> (<short name>)`; a bare number is not allowed. Enforced by doc-lint.sh. -->

<The conclusion. If half-settled, say which detail is open.>

Basis: <why. Numbers from elsewhere carry their source and measurement basis, and are
marked as not verified in this project.>

---

## Revision history

### YYYY-MM-DD
- Created.
