<!-- generated-from: rules/kb-discipline.md sha256:57af63ffe2ca4e1ff4717c2453318c0f6af21ba630c647ead9c098ecece4ffff -->
<!-- doc-lint:rule-definition -->
# Knowledge document discipline

**Applies to**: `kb/*.md` — **things models retrieve from.** The kb is written for being pulled out one item at a time; its goal is to keep the model from making things up, and being pleasant to read is not the objective.

The kb lives in `.claude/kb/` by default; when a project moves it elsewhere, it registers the project root's `.claude/kb-directories` (one per line, relative to
the project root, with a `#` reason; multiple lines are fine).

Enforced by `scripts/doc-lint.sh`, and the exact patterns are the ones in it; the forms it does not check, writers keep to all the same.
The criteria are chosen by this package's language; what is not implemented it reports explicitly as unimplemented.

## 1. Every fact stands on its own

A fact must still hold when retrieved alone. Three kinds of reference are not written:

| Don't | Examples | Write it as |
|---|---|---|
| **Positional references** | "as stated above", "same as above", "see above", "mentioned earlier", "the aforementioned", "as described below", "see below", "see the section below", "see the table above", "that table above", "the above-mentioned", "the text below", "the table below", "the three items above", "the following is", "item by item, as follows:", "see the next paragraph", "same as the line above", "the previous entry", "the next entry" | The name of the thing it points at |
| **References to time** | "this round", "the current round", "the previous round", "the round before", "last round" | A date plus "that round", or the number of that experiment; one that means any round is rewritten without the reference |
| **Self-references** | "this entry", "this decision", "this experiment", "that decision", "that experiment", "this invariant", "this section", "this chapter", "this table", "this file", "this document" | The name of the current location |

**Write the current location out as a name**:

| Referring to | Write it as |
|---|---|
| A numbered entry | "number (short name)", the same as item 5 "A number can serve as an index, never as a name" |
| A section | Its heading |
| A table | The thing it judges |
| A document | Its filename |

**Not covered**: "above" and "below" pointing at a physical position ("the line below each heading"); a reference to time where this line, or a heading at any level above it, carries a date; "the next round", "this time", "last time"; entries in a history section; "this project", "this repository", "this machine".

## 2. Every entry carries its source and status

Source, measured or inferred, on what measurement basis. Dates go only with events: a measurement or a conclusion read elsewhere says which day it was done; a sentence that states the current state carries no date.

- Numbers must carry their basis: blocks or bytes, metadata included or not, on what
  hardware under what workload.
- Conclusions read elsewhere carry their source and the note "not verified in this
  project".
- When you cite someone else's measurement, write down what it does and does not prove; whether it extrapolates to here is judged and written down by the person writing it.
- **All historical data is reference only.** To use it in support of a new conclusion,
  re-run it first and confirm it still holds today.

## 3. Record "we do not know" explicitly

Anything investigated without a conclusion, anything unverified, anything
deliberately set aside — write it down and mark it as such.

## 4. A contradiction is worse than a gap

A given fact gets **exactly one** authoritative record; everywhere else links to it.

## 5. A number can serve as an index, never as a name

- **Carry the name at every citation**: write it as "number (short name)", never the
  number alone.
- **A number gets exactly one definition**; everywhere else links to it.

**The criterion**: substitute every number in the text with its defining sentence —
does the sentence still read?

**A registration site takes exactly two forms**; a number in a table's first column is not a registration:

| Form | How it is written | Where the short name comes from |
|---|---|---|
| Registry table | a line of its own directly above the table it governs: `<!-- doc-lint:registry name-col=2 -->` | column `name-col` of each row (`name-col` ≥ 2) |
| Registry heading | `## D1 Data mobility —— settled`, **the dash is not optional** | after the number, before the dash |

A heading without the dash (`### D1 and its relation to parity`) is not a registration.

- A short name may not be empty, may not equal the number itself, may not be shared by two numbers, and its display width is capped at 48 (a Chinese, Japanese or Korean character counts 2).
- Domain terms (`RAID5`, `SHA256`) are shaped exactly like numbers; exempt them explicitly: `<!-- doc-lint:not-numbers RAID5 SHA256 -->`.
- When the kb carries translations, each language is judged on its own: files are grouped by language suffix (`.zh.md`, `.en.md`, `.ja.md`; a file with no suffix belongs to this package's language), a number has one registration site in each language, and a citation is compared against the registration site of its own language.
- For citations missing their short name, first run `scripts/doc-lint-fix-names.py <file>` to add "（short name）", then run `doc-lint.sh`.

`scripts/doc-lint.sh` judges: one registration site per number, the short name is usable, the registry table is not broken, citations carry the short name and match the registration site, and an id-shaped token recurring 3 or more times with no registration.
Lowercase numbers (`o1`), forms with a `-`, `.` or `_` in front of the number (`-D1`), citation forms other than putting the name in the link text, and whether a short name is a good name, it does not judge.

## 6. Tables beat prose

Whatever can be written as a table is written as a table.

## 7. Do not optimise for reading order

No setup, no transitions, no conclusions to round things off.

## 8. Body text states only the current state; history goes to a "Revision history" at the end

Which kb files carry history is registered in `.claude/history-carriers` at the project root: one directory or file per line (relative to the project root), with the reason after `#`.
A registered file must close with a "## Revision history" section, kept even with no history yet; an unregistered file must not have that section, and the history lives in git.
Without that registry, every file under kb must close with the section. `INDEX.md` is not judged.

For history entries numbered "（其 <ordinal>）", look up the highest existing number under the same `##` section, the same `### <date>` heading and the same named item before taking one; a collision is judged by the gate stage "History entry numbers" (`历史条目编号`, `scripts/history-ordinal.sh`).

### A dated snapshot of the current state is history too, and stays stuck on that day

A current-state sentence carries no date: when the state changes, change the current value, and write what it looked like on that day into "## Revision history" (not in a file that is not registered as carrying history).
For an existing dated current-state sentence, delete the date and keep the fact; check the fact once first, and if it is stale, change it to the current value.

**The criterion**: delete the date and read it again. If it reads as saying how things are today (whether something exists, how many, what is still owed), it is a current-state sentence; if it reads as saying what was done that day (changed to, settled, ran once), it is an event sentence.

The gate stage "代号" (`scripts/code-name-lint.py`) judges one shape in kb lines added relative to the diff base: a date followed within 4 characters by 现状, 目前状态 or 当前状态 ("current state") and then a colon.
