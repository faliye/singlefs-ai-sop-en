<!-- generated-from: templates/kb/decisions.md sha256:c4c4e686926487e6c1623c3757b890ea955bccf89f21df97fdffcbcbdd23b3b7 -->
# Design decision record

Each decision has exactly three states: **settled** / **half-settled** (direction fixed,
details open) / **undecided**.
The format and the hard requirements are in the `decide` skill. When overturning a
decision, edit the body directly and put the basis into the closing "Revision history".

---

## D1 <name, 24 characters or fewer> —— undecided

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
