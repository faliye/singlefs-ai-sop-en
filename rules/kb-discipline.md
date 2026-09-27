<!-- generated-from: rules/kb-discipline.md sha256:00bc459a12f0ec5ece39151e9b467b2c8c6bcdd1868f7ccc1734c1dc54d5e98a -->
<!-- doc-lint:rule-definition -->
# Knowledge document discipline

**Applies to**: `kb/*.md` — **things models retrieve from.**

**The kb is not reading material.** With a model at hand, nobody reads the kb start
to finish; they have the model retrieve, extract and summarise. So the kb is
designed for **being pulled out one item at a time**, not for being read through.

**Its one goal is to keep the model from making things up.** Being pleasant to read
is incidental, not the objective.

## 1. Every fact stands on its own

A fact must still hold when retrieved alone, without the other paragraphs in its section.

**No dangling references** — "as stated above", "same as above", "see above",
"mentioned earlier", "the aforementioned", "as described below", and positional
references such as "see below", "see the section below", "see the table above", "that
table above", as well as the bare forms without "see": "the above-mentioned", "the text below", "the table below",
"the three items above", "below is the original text before it was voided", "the following is", "item by item, as follows:",
and phrasings that point at the adjacent paragraph or line, such as "see the next paragraph" and "same as the line above".
They are harmless when read through and break on the spot when retrieved;
and **a model will not say "I do not follow" — it will fill in something.**

"the previous entry" / "the next entry" are **outside the gate**: measured on a real
corpus, false reds outweigh true ones — one registered short name is literally "residue
of the previous timeline", and ordinary verb phrases like "note down one more item" hit
too. Writers still must not use them to point at another entry, but that half is checked
by people, not by the gate.

"Above" and "below" (上面 / 下面): **the gate checks only the forms that point at a position in the document**:
those preceded by the start of a sentence, punctuation, or a function word such as 以, 把, 按, 取, 在 or 是,
and followed by 的, 是, 保留, a numeral, 第 (an ordinal) or an item number, plus those that follow 列在, 照录在, 附在 or 放在
("listed / copied / attached / placed above or below"). "the four points above it", "the units below a multi-level tree"
and "the line below each heading" describe physical positions and are not judged. The exact patterns are the ones in `scripts/doc-lint.sh`.
"The above" and "the following" (以上 / 以下) are checked only in forms like "the following is" and "the several items above";
"earlier" / "later" (前面 / 后面) are **outside the gate**. "over" / "under" (上方 / 下方) are checked
only in the two forms that carry 见 ("see above", "see below"): with 见 in front they can only mean a
position in the document, while the bare words mostly point at a named physical position
("below the field table", "the comment above `min`") and are not judged.

**References to time are equally forbidden** — "this round", "the current round", "the previous round", "the round before", "last round".
A positional reference that points the wrong way can still be spotted; a temporal one cannot —
when "measured this round: green" comes back alone, which round it was is unknowable, and the model
fills one in anyway.
⇒ Write the anchor: a date plus "that round", or the number of the experiment in question. A sentence that genuinely means any round should be rewritten without the reference
("the round that ran it" becomes "the run that produced this artefact").

The gate's criterion is **whether this line, or the nearest heading above it, carries a date**; if so it
passes. Three things are not checked: "the next round" means any round yet to happen and is a generic by
nature; "this time" / "last time" hang mostly on technical objects (this diff, last scrub's watermark) and
are self-sufficient; and every entry in a history section hangs under a dated heading, which body scanning
stops at anyway. Compounds such as 样本轮 ("sample round") do not count: when the character to the left is 样, it is not judged.

**Self-references are equally forbidden** — "this entry", "this decision", "this
experiment", "that decision", "that experiment", "this invariant", "this section", "this chapter", "this table", "this
file", "this document".
Of these, "this file" is outside the gate: doc-lint's self-reference word list has no "file"
ending, and a genuine self-reference almost always says "this document". Writers still must
not use it to point at the document; that half is checked by people.

A self-reference is sneakier than a dangling reference: a dangling reference at least
points in a direction, while a self-reference points at **"here"** — and the item
retrieval hands back **has no "here"**. It has already been lifted out of the file; the
`## <number> <short name>` heading that item sits under does not travel with it. So when
"this decision also rules that K may not be a format constant" comes back alone, not one
word remains about whose decision that was.

⇒ **Write the current location out as a name**:

| Referring to | Write it as |
|---|---|
| A numbered entry | write it as "number (short name)" — the same shape as rule 5 |
| A section | Its heading |
| A document | Its filename |

**This is the dual of rule 5, "A number can index a thing, it cannot name it".** Rule 5
governs "citing another number without writing its short name"; this one governs "talking
about yourself without even writing the number". Only together do they close — without
this one, rewriting "number (short name)" as "this decision" would *pass*
the gate.

**Not covered**: "this project", "this repository", "this round", "this machine" refer to
the project, not to a location in a document. The gate judges a self-reference only at the start of a sentence or right after punctuation;
compounds such as 样本文档 ("sample document"), 成本表 ("cost table") and 三本文档 ("three documents") do not count —
without that restriction, false reds would outnumber real violations

Enforced by `scripts/doc-lint.sh`, with the criteria chosen by this package's language: Chinese judges all three; Japanese judges positional references and self-references, recognising dedicated words such as 上記, 前述, 本節 and 本文書,
with temporal references not implemented; English implements none of the three. What is not implemented is reported explicitly as **unimplemented**, never passed silently.

## 2. Every entry carries its source and status

Source, measured or inferred, on what measurement basis. Dates go only with events: a measurement or a conclusion read elsewhere says which day it was done;
a sentence that states the current state carries no date (see "A dated snapshot of the current state is history too, and stays stuck on that day" under section 8).

This is not pedantry: **it is the only cue a model has for telling "this is
established" from "this was a guess at the time".** Without the cue, the two look
identical in a retrieval result.

- Numbers must carry their basis: blocks or bytes, metadata included or not, on what
  hardware under what workload.
- Conclusions read elsewhere carry their source and the note "not verified in this
  project".
- **When you cite someone else's measurement, write down what it does and does not prove.**
  Provenance is not enough: provenance answers "where did this number come from",
  while whoever retrieves it actually needs to know "does this number apply to me".
  There is a layer of extrapolation between the two, and **the person writing it down
  must do that extrapolation and record it** — leave it to the next reader and they
  will most likely skip it and use the number as is.
- **All historical data is reference only.** Every measurement is bound to the build
  it was taken on; to use it in support of a new conclusion, re-run it first and
  confirm it still holds today.

## 3. Record "we do not know" explicitly

Anything investigated without a conclusion, anything unverified, anything
deliberately set aside — write it down and mark it as such.

**The way a model fills a gap is by inventing.**
A gap is more dangerous than an error — an error can be argued with; a gap gives you
nothing to argue with.

## 4. A contradiction is worse than a gap

A given fact gets **exactly one** authoritative record; everywhere else links to it.

A person reading through notices when two passages disagree. **Retrieval will not
serve up both — it picks one, and does not tell you it picked.**

## 5. A number can serve as an index, never as a name

**Giving a concept a number is not the same as giving it a name.** In retrieval a
number **does not stand on its own** — it leaves the definition somewhere else and
leaves nothing but a symbol at the point of use. So **whoever cites it can change its
meaning without noticing**, because not one word will look out of place.

⇒ **Carry the name at every citation**: write it as "number (short name)", never the
number alone.
⇒ **A number gets exactly one definition**; everywhere else links to it (a direct
corollary of item 4, "A contradiction is worse than a gap").

The same number meaning something else in two other documents is the failure this rule
guards against: nowhere does anything look wrong, and a criterion written on that basis
turns into a tautology nothing can falsify.

⇒ **The criterion**: substitute every number in the text with its defining sentence —
does the sentence still read? If it does not, the citation drifted long ago, and the
number hid the drift.

**A registration site takes exactly two forms** — nothing is inferred from "the first
column holds a number", because first columns are also used as row labels
(`| D2 | node-internal parity overlaps with D2's variable width… |`); inferring would make
every one of those a false alarm:

| Form | How it is written | Where the short name comes from |
|---|---|---|
| Registry table | a line of its own directly above the table it governs: `<!-- doc-lint:registry name-col=2 -->` | column N of each row (N ≥ 2) |
| Registry heading | `## D1 Data mobility —— settled`, **the dash is not optional** | after the number, before the dash |

Why the heading form demands the dash: if `### D1 and its relation to parity` also counted
as a registration, it would become a second registration site and override the short name —
**turning every correct citation red at once**.

A short name may not be empty, may not equal the number itself, may not be shared by two
numbers, and its display width is capped at 48 (CJK counts 2, ASCII 1 — i.e. 24 CJK
characters): **every citation has to carry it, and a name too heavy to carry is no name at
all**. Domain terms (`RAID5`, `SHA256`) are shaped exactly like numbers and a machine cannot
tell them apart; exempt them explicitly: `<!-- doc-lint:not-numbers RAID5 SHA256 -->`.

**Which half the gate handles**: `scripts/doc-lint.sh` enforces the mechanically decidable
half —

| What it checks | The criterion |
|---|---|
| One registration site | exactly one per number |
| The short name is usable | non-empty, not the number itself, not shared with another number, within the width cap |
| The registry table is not broken | the marker sits directly above its table, every row reaches the name column, `name-col` ≥ 2 |
| Citations carry the name | every citation reads `number（short name）` and matches the registration site (ignoring `**`/`` ` `` and how much whitespace) |
| The converse | an id-shaped token recurring **≥3 times** with no registration site at all |

That last one cannot be dropped: without it a kb that never registered anything comes out
all green — no registration sites means the other checks have nothing to do — and the kb
where this went wrong was exactly that kind.

**The half only a person can do**: the substitution criterion (does the sentence still
read once every number is replaced by its defining phrase) is a semantic judgement;
no machine decides that. Lowercase numbers (`o1`), citation
forms other than putting the name in the link text, and whether a short name is a *good*
name are all out of the gate's reach.

One more is **known but not stopped**: put a `-`, `.` or `_` in front of a number (write `-D1`)
and the gate no longer recognises it as a citation. It could be stopped, but at the cost of falsely
rejecting a batch of normal citations in real kbs, so it is not stopped

## 6. Tables beat prose

Filterable, alignable, and every row stands alone.

## 7. Do not optimise for reading order

The kb has no "read it from the top" use case. No setup, no transitions, no
conclusions to round things off.

## 8. Body text states only the current state; history goes to a "Revision history" at the end

The rule is the same as for design documents, but the **reason differs, and is
harder**: retrieval serves the stale entry up **on its own**, with no context and
nothing to compare against; a model has no "I remember this changed last week" and
takes it all at face value.

Every kb document must close with a "## Revision history" section — keep the section
even with no history yet, for later. `INDEX.md` is the exception: it is a signpost
table, it carries no facts of its own, so it has no old conclusions to keep.
Enforced by `scripts/doc-lint.sh`.

### A dated snapshot of the current state is history too, and stays stuck on that day

Sentences like "Status on 2026-09-14: … still not in layer 0's workload" or "(all four already present as of 2026-09-14 …; layer 0 has only the first transaction)" written in the body text
state the present on the day they are written, and from the next day on they are a snapshot of that day: later changes do not touch them, so they stay stuck on that day.
When retrieval serves one up, the model reads the leading date as "that is in the past", while the sentence actually says "the current state as of that day".

⇒ A current-state sentence carries no date: when the state changes, change the current value, and write what it looked like on that day into "## Revision history".
For an existing dated current-state sentence, delete the date and keep the fact; check the fact itself once first (`verify-before-claiming.md`), and if it is stale, change it to the current value.
To tell whether a dated sentence is a current-state sentence, delete the date and read it again — if it reads as saying how things are today (whether something exists, how many, what is still owed), it is a current-state sentence;
only if it reads as saying what was done that day (changed to, settled, ran once) is it an event sentence. A date in the past does not mean the sentence talks about the past.

The gate does not check this for now: the existing kb uses this pattern a lot ("Status (date):" and the like), and a check would turn whole areas red at once; sweep first, then add the check.
