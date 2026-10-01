<!-- generated-from: rules/kb-discipline.md sha256:fbc02aee6ce61e677bf2b4538c3912bba341d07369454023c4ef5ca500b68b47 -->
<!-- doc-lint:rule-definition -->
# Knowledge document discipline

**Applies to**: `kb/*.md` — **things models retrieve from.**

The kb is written for **being pulled out one item at a time**, not for being read through.
**Its one goal is to keep the model from making things up.** Being pleasant to read is not the objective.

## 1. Every fact stands on its own

A fact must still hold when retrieved alone, without the other paragraphs in its section.

**No positional references** — "as stated above", "same as above", "see above",
"mentioned earlier", "the aforementioned", "as described below";
positional references such as "see below", "see the section below", "see the table above", "that
table above"; the bare forms without "see": "the above-mentioned", "the text below", "the table below",
"the three items above", "below is the original text before it was voided", "the following is", "item by item, as follows:";
and phrasings that point at the adjacent paragraph or line, such as "see the next paragraph" and "same as the line above".

"the previous entry" / "the next entry" are **outside the gate** and checked by people: writers still
do not use them to point at another entry.

"Above" and "below" (上面 / 下面): **the gate checks only the forms that point at a position in the document**:
those preceded by the start of a sentence, punctuation, or a function word such as 以, 把, 按, 取, 在 or 是,
and followed by 的, 是, 保留, a numeral, 第 (an ordinal) or an item number, plus those that follow 列在, 照录在, 附在 or 放在
("listed / copied / attached / placed above or below"). Physical positions such as "the four points above it", "the units below a multi-level tree"
and "the line below each heading" are not judged. The exact patterns are the ones in `scripts/doc-lint.sh`.
"The above" and "the following" (以上 / 以下) are checked only in forms like "the following is" and "the several items above";
"earlier" / "later" (前面 / 后面) are **outside the gate**. "over" / "under" (上方 / 下方) are checked
only in the two forms that carry 见 ("see above", "see below"); the bare words
("below the field table", "the comment above `min`") are not judged.

**References to time are equally forbidden** — "this round", "the current round", "the previous round", "the round before", "last round".
Write the anchor: a date plus "that round", or the number of the experiment in question. A sentence that genuinely means any round should be rewritten without the reference
("the round that ran it" becomes "the run that produced this artefact").

The gate's criterion is **whether this line, or a heading at any level above it, carries a date**; if so it
passes. Three things are not checked: "the next round"; "this time" / "last time"; entries in a history section.
Compounds whose character to the left is 样, such as 样本轮 ("sample round"), are not judged.

**Self-references are equally forbidden** — "this entry", "this decision", "this
experiment", "that decision", "that experiment", "this invariant", "this section", "this chapter", "this table", "this
file", "this document".
"This file" is outside the gate and checked by people.

**Write the current location out as a name**:

| Referring to | Write it as |
|---|---|
| A numbered entry | write it as "number (short name)" — the same shape as rule 5 |
| A section | Its heading |
| A document | Its filename |

**Not covered**: "this project", "this repository", "this round", "this machine" refer to
the project and are not self-references. The gate judges a self-reference only at the start of a sentence or right after punctuation;
compounds such as 样本文档 ("sample document"), 成本表 ("cost table") and 三本文档 ("three documents") are not judged.

Enforced by `scripts/doc-lint.sh`, with the criteria chosen by this package's language: Chinese judges all three; Japanese judges positional references and self-references, recognising dedicated words such as 上記, 前述, 本節 and 本文書,
with temporal references not implemented; English implements none of the three. What is not implemented is reported explicitly as **unimplemented**, never passed silently.

## 2. Every entry carries its source and status

Source, measured or inferred, on what measurement basis. Dates go only with events: a measurement or a conclusion read elsewhere says which day it was done;
a sentence that states the current state carries no date (see "A dated snapshot of the current state is history too, and stays stuck on that day" under section 8).

- Numbers must carry their basis: blocks or bytes, metadata included or not, on what
  hardware under what workload.
- Conclusions read elsewhere carry their source and the note "not verified in this
  project".
- **When you cite someone else's measurement, write down what it does and does not prove.**
  Whether it extrapolates to here is judged and written down by the person writing it.
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
- **A number gets exactly one definition**; everywhere else links to it (item 4, "A
  contradiction is worse than a gap").

**The criterion**: substitute every number in the text with its defining sentence —
does the sentence still read? If it does not, the citation has drifted.

**A registration site takes exactly two forms**; a number in a table's first column is not a registration:

| Form | How it is written | Where the short name comes from |
|---|---|---|
| Registry table | a line of its own directly above the table it governs: `<!-- doc-lint:registry name-col=2 -->` | column N of each row (N ≥ 2) |
| Registry heading | `## D1 Data mobility —— settled`, **the dash is not optional** | after the number, before the dash |

A heading without the dash (`### D1 and its relation to parity`) is not a registration.

A short name may not be empty, may not equal the number itself, may not be shared by two
numbers, and its display width is capped at 48 (CJK counts 2, ASCII 1 — i.e. 24 CJK
characters). Domain terms (`RAID5`, `SHA256`) are shaped exactly like numbers; exempt them
explicitly: `<!-- doc-lint:not-numbers RAID5 SHA256 -->`.

**Which half the gate handles**: `scripts/doc-lint.sh` enforces the mechanically decidable
half —

| What it checks | The criterion |
|---|---|
| One registration site | exactly one per number |
| The short name is usable | non-empty, not the number itself, not shared with another number, within the width cap |
| The registry table is not broken | the marker sits directly above its table, every row reaches the name column, `name-col` ≥ 2 |
| Citations carry the name | every citation reads `number（short name）` and matches the registration site (ignoring `**`/`` ` `` and how much whitespace) |
| The converse | an id-shaped token recurring **≥3 times** with no registration site at all |

For citations missing their short name, first run `scripts/doc-lint-fix-names.py <file>` to add "（short name）" from the registration sites, then run `doc-lint.sh`; it only adds and never judges — whether registration sites and short names are right is still judged by `doc-lint.sh`.

**The half only a person can do**: the substitution criterion (does the sentence still
read once every number is replaced by its defining phrase). Lowercase numbers (`o1`), citation
forms other than putting the name in the link text, and whether a short name is a *good*
name are all out of the gate's reach.

**Known but not stopped**: put a `-`, `.` or `_` in front of a number (write `-D1`)
and the gate no longer recognises it as a citation.

## 6. Tables beat prose

Whatever can be written as a table is written as a table.

## 7. Do not optimise for reading order

No setup, no transitions, no conclusions to round things off.

## 8. Body text states only the current state; history goes to a "Revision history" at the end

Which kb files carry history is registered in `.claude/history-carriers` at the project root: one directory or file per line (relative to the project root), with the reason after `#`.
A registered file must close with a "## Revision history" section, kept even with no history yet; an unregistered file must not have that section — change its body directly, and the history lives in git.
Without that registry, every file under kb must close with the section. `INDEX.md` is never judged.
Enforced by `scripts/doc-lint.sh`.

### A dated snapshot of the current state is history too, and stays stuck on that day

A current-state sentence carries no date: when the state changes, change the current value, and write what it looked like on that day into "## Revision history" (not in a file that is not registered as carrying history; there the history lives in git).
For an existing dated current-state sentence ("Status on <date>: …" and the like), delete the date and keep the fact; check the fact itself once first (`verify-before-claiming.md`), and if it is stale, change it to the current value.
To tell whether a dated sentence is a current-state sentence, delete the date and read it again — if it reads as saying how things are today (whether something exists, how many, what is still owed), it is a current-state sentence;
only if it reads as saying what was done that day (changed to, settled, ran once) is it an event sentence.

The gate does not check this for now.
