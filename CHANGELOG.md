# CHANGELOG

Version history for the rules and the gate. `CLAUDE.md` and `rules/*.md` keep no
history sections (design-doc-discipline); history lives here. For per-change
detail see `git log` — commit messages are the change notes.

## 0.0.42 — 2026-09-11

**Pushing now checks and pushes every language repo together.** 0.0.41 was pushed for zh only, leaving en and ja
two commits behind: the remote carried three languages stating different versions of the rules, and nothing
complained, because the gate runs locally and cannot see the remote. New `scripts/push-all.sh` and the git hook
`scripts/githooks/pre-push`: set `core.hooksPath` once in each language repo, and from then on pushing any of
them first checks every language repo (on master, clean working tree, same VERSION, its own gate green); only
when all pass does it push the other language repos and then let this one through. If anything fails, nothing
is pushed; if a push is rejected halfway, it reports which repos already went out. When you push from a git worktree,
git gives the hook an absolute `GIT_DIR` for this repo, which overrides `git -C`, so the script clears it
first — otherwise checking and pushing the other language repos would land on this one (pushing from an
ordinary clone, the hook sees no such variable). `selftest.sh` gains 10 cases that go through a real
`git push` → hook path with local bare repos as remotes. `CLAUDE.md` says how to enable it.

**Four rule additions, from traps hit in singlefs this round.** `command-safety.md` gains "a script with a gate hands
over its output only after judging it": output first and judge later, and the caller's redirect file keeps an output
judged void that looks just like a valid one. `evidence-discipline.md` gains two: when an arm's definition says both how
it is done and what that achieves, the first must imply the second, or the legs judging by different sentences are
judging two different arms; and an extreme from a sweep that lands on the sweep's endpoint is not a measured number and
can only be written as "≤ endpoint". `session-wrapup.md` item 4 goes from three assumptions to four: creating a new file
may overwrite another session's freshly written uncommitted file, so create new files exclusively.

## 0.0.41 — 2026-09-10

**New `rules/code-discipline.md`: how machine-first lands in code.** There is one principle: past best
practice does not get to decide for us. `machine-first.md` said why the old rules need re-examining and
how to examine them, but not how to write code afterwards; the naming point (no length cap on names, no
abbreviations) lived in `engineering-philosophy.md` as a review criterion, not a rule. The new file turns
these into rules: names, branches, types, errors, functions and nesting, comments — plus a table grouped
by source: 85 popular practices in 10 groups from *Clean Code*, SOLID, *Refactoring*, *The Pragmatic
Programmer*, the Rust API Guidelines, the Linux kernel coding style and others, each with a disposition
and how we write it. The code-level conclusions moved over from the other two files, which keep only the
philosophy and the design- and process-level tables; the abbreviated names in the moved examples
(`dev`, `m`, `Lba`, `commit_txn`, `num_`) are spelled out.

The same version tightens six statements after review: path count measures control flow only and is not
the whole of verification difficulty — a loop's iteration bound, cross-iteration state and early exits
must be stated separately; the ban on wildcard arms applies to closed sets, and where the semantics allow
unknown values, "unknown" becomes an explicit variant and the `match` stays exhaustive; the unit of "one
concept, one name" is the semantic concept, not the word; a name that stands without its module path is
stated as a deliberate choice to carry namespace information in the name; `as_` / `to_` / `into_` are
conventions, not guarantees, with the C-CONV table as the authority; error variants are split by the
decision the caller must make, not expanded one per underlying cause. "A bounded path count means it can
be verified" in `engineering-philosophy.md` and `machine-first.md` becomes a necessary condition
accordingly.

**New gate stage "Naming discipline" (`scripts/naming-lint.sh`).** It scans the names we declare in
every `.rs` in the project and flags single letters and common abbreviations (a list of 144, each with
what to write instead). A project registers domain abbreviations in `.claude/abbreviations` — and may
register its own numbers as a class (`e<数字>`) — and declares directories or single files not to scan
in `.claude/naming-lint-exclude` (when sweeping old code, list files one by one and delete a line as
each is fixed); both need a reason. The in-line exemption is
`// naming-lint:external <reason>`. Method names in trait implementations, `extern` blocks and file
names fixed by Cargo are not judged. Names in shell scripts are not yet a check; `gate.sh` lists it
among the unimplemented stages.
12 fixture sets; 39 mutations applied to a copy (removing one exemption, one kind of declaration site,
one step of stripping comments or strings) were all caught by the fixtures.
A dry run on singlefs's research code (119 files, 16792 names, no abbreviations registered) reported
7246 hits; sampling each kind of hit found two kinds of false positive, both fixed: `lib.rs`, whose name
Cargo fixes, and the English article `a` in the middle of a name.

**`scripts/check.sh` runs clippy with seven more code-discipline lints**: `wildcard_enum_match_arm`,
`allow_attributes_without_reason`, `cast_possible_truncation`, `cast_sign_loss`, `cast_possible_wrap`,
`undocumented_unsafe_blocks`, `shadow_unrelated`. Measured on a sample crate: with one violation of each
planted, all seven went red; the version written per the rules passed all four steps — format, clippy,
build, unit tests.

**`doc-discipline.md`, `writing-economy.md` and `writing-style.md` are merged into
`rules/writing-discipline.md`.** All three govern text written for people (who it is for, how long, how
it is said). The merged text was compared line by line: no sentence of the three bodies was lost; only
the sentences pointing from one of the three files to another were dropped. Everything that referenced
them now points at the new file.

**doc-lint's rule-list check extends to the template and to projects.** It used to check only that the
SOP repository's `CLAUDE.md` and `rules/` match item for item; now `templates/CLAUDE.project.md`, and a
project's `CLAUDE.md` against its installed copy, are checked by the same criterion.
Two measured gaps forced this: the template has never referenced `pushback-discipline` since it was
added in 0.0.31; singlefs's `CLAUDE.md` has never `@`-referenced `engineering-philosophy`, `sop-first`,
`pushback-discipline` or `writing-style`, so those four were never loaded into context in singlefs
sessions. The template is now complete.

`GLOSSARY.md` revises the notes on "abbreviation" and "path count" and adds "abbreviation registry" and
"wildcard arm".

## 0.0.40 — 2026-09-10

**New gate stage "CHANGELOG continuity" (`scripts/changelog-lint.sh`): every version gets its
own section, and the newest section is `VERSION`.** Version discipline only asks whether the
spec proper changed without a `VERSION` bump; it never asked whether the CHANGELOG kept up.
Measured: in 0.0.39 `VERSION` went from 0.0.35 to 0.0.39 while the CHANGELOG in all three
language repositories skipped 0.0.36, so that version's two changes were recorded nowhere —
every gate was green, and it took a paragraph-by-paragraph read of the diffs to see it.

It judges the whole file, not a diff window: second-level headings may only be
`## x.y.z — YYYY-MM-DD`, and the last one may be an undated tail such as "x.y.z and earlier";
the newest section equals `VERSION`; each pair of adjacent sections must be immediate
successors (patch +1, minor +1 with patch reset, or major +1 with the rest reset) — a gap, a
duplicate or a reversed pair is red, and so is having no section at all. It runs only in the
SOP repository itself, and each language repository judges its own CHANGELOG. `CLAUDE.md` gains
one sentence on its first screen, and `skills/gate/SKILL.md` adds it to the stages that run
only in the SOP repository.

## 0.0.39 — 2026-09-10

**`rules/verify-before-claiming.md` gains a section: you checked the narrow claim and
then stated the broad one.**

The file previously covered two things: check external state before stating it, and
"is it settled" versus "what does it actually say". Both assume the failure mode is
**not having checked**. This section covers the other one — **you checked and it still
failed**: you did run the command, you did read the file, you are holding a verified
proposition, and **the error is that the sentence you then said is wider than it**.

The test: the sentence you are about to say — are its subject and scope exactly the
ones you just checked? You checked "this path is blocked", so you may say "this path is
blocked", not "it cannot be done". You checked "this file does not say it", so you may
say that, not "nothing in the repo says it". What to do: take the broad sentence as a
proposition to be proved, ask "what cases would I have to rule out for this to be
true", and rule them out one by one; if you cannot finish, shrink the sentence back to
the range you actually finished.

Measured in singlefs (2026-09-10, twice on the same day by the same person):
(1) checked "`sudo` is blocked by the sandbox" ⇒ wrote "root is unavailable, so this
observation cannot be made", while another path in the same repo **does not need that
privilege** and the docs say word for word that it was measured working ⇒ a whole round
with zero observations, and the recorded reason was false.
(2) checked "this object is not listed in that class rule's enumeration" ⇒ wrote
"nothing in the repo covers this cell" and framed an entire experiment on it, while
**the class membership is stated in two other files** and the class rule covered it all
along ⇒ the framing was void and the same owed-check entry got written wrong twice.

⇒ The section closes with one more note: "is this object covered by a clause" is
especially prone to this, because **a clause can live elsewhere and cover it by class**
while the object's own section says nothing; before judging, grep its name across the
whole repo and see whether it has been placed in some existing class.

## 0.0.38 — 2026-09-10

**Two additions, each folded into an existing section; both belong to the family
"the clause is still there but has stopped doing anything".**

**`rules/evidence-discipline.md`, under "After a re-run, check the prose back against it",
gains a second form: it is not the number that drifted, it is the qualifier that went
missing.** That section used to cover numeric drift only — the prose says 1.475x while
the artifact says 1.625x, and the interval assertion cannot catch it. But prose and
artifact can disagree another way: **every number is true, and the conclusion is wider
than the artifact supports**, because the table in the prose dropped a whole parameter
dimension. When a number drifts you still have two numbers to lay side by side; when
the qualifier is gone **there is nothing to compare against**. The test: when a
conclusion says "only A buys you this", go read the artifact for **the non-A arm at
every parameter point**; if the table in the prose has no column for that parameter,
the word "only" does not hold.

Measured in singlefs (2026-09-09): an experiment's prose said "the one thing only
clustering buys is the reclaim cell … neither other arm can free a single segment",
while in the same stored artifact another arm was **identical cell for cell** at the
non-interleaved point; the table in the prose had no interleave-step column. **The
harness's assertion scope had been right all along** — the unit test pinned that
parameter and said so in the assertion message — what shed the scope was the prose.
The rerun was byte-identical and **the gate was green**. That conclusion had already
been inherited in three downstream places, one of them the very decision item it settled.

**`rules/test-discipline.md`, under "a failure clause must not make the conclusion
unfalsifiable", gains the converse: the antecedent can be written backwards, and once
it is, it never fires.** The main rule covers "the clause makes the conclusion
impossible to overturn"; this one covers "the clause itself can never be triggered".
Written backwards does not look like **wrong**, it looks like **did not fire** — and in
a round report, "this clause did not fire" is indistinguishable from "this clause was
checked and the conclusion is fine". What to do: **after writing each failure clause,
immediately write one sentence saying what observation would trigger it**; if you
cannot, you wrote it backwards or you wrote it empty.

Measured in singlefs (2026-09-09): a pre-registered clause said "if some arm's benefit
appears only on the **non-interleaved** workload ⇒ record the condition as unknown",
while that round's finding was that the benefit appears only on the **interleaved**
one ⇒ the antecedent was identically false, it never fired once, and it should have.
The person who wrote the clause self-reviewed twice without seeing it; another leg
caught it by checking the antecedent word for word.

## 0.0.37 — 2026-09-09

**`rules/evidence-discipline.md`, under "Never pick the conclusion first and then build a model
for it", gains a section: an arm's definition is nailed down before the run too, and there is
exactly one admissible path when a failure clause fires.**

The existing text gave only prohibitions — criteria, thresholds and void clauses are fixed
before the run, "do not go back and change the criteria", "do not loosen the rule afterwards
and then declare victory" — but **no admissible path**. So when a failure clause actually
fires, only two options remain: pretend it did not, or throw the whole round away. And when
an arm is loosely worded, the attack hits **its weakest reading**; "clarifying" the arm into
the strong reading and declaring it the winner changes no criterion on paper and is post-hoc
modelling in substance.

The three steps added: **record the loss** under the weakest reading, **tighten only** (the
new form must be judged at least as harshly by the original criteria), and **state where it
tightened** (if you cannot say what it demands more of, it is a loosening). It also points out
that the first step is the one people skip, and that once skipped, an honest tightening and a
"loosen it, then declare victory" read identically on the page.

Measured in singlefs (2026-09-09): in a three-way round, a backward-reasoning leg ruled an arm
out under the failure clause written before the run, on the grounds that it did not cover field
order. On checking, the arm as registered had never said it projected scalars only — what was
hit was its weakest reading. The verdict followed the three steps, and the tightened form went
into that decision's write-up.

## 0.0.36 — 2026-09-09

**`rules/evidence-discipline.md` gains the converse of "Withdrawing a number or a conclusion
also means sweeping for who cites it": a withdrawal's rationale collapsing does not bring the
withdrawn conclusion back.** A withdrawal is a verdict; its rationale collapsing only means that
verdict lost its grounds, not that the opposite holds, and bringing the original back takes a
fresh argument. And after a withdrawal the slot usually already has another rationale holding it
up, which you miss entirely if you stare only at the withdrawn one. What to do: ask three
questions in order — which rationale holds the slot today, does it stand up on its own, and has
the re-argument for reviving the withdrawn one actually been done.

**`skills/decide/SKILL.md` gains item 7: an open item's question must have exactly one
reading.** Write out each reading separately; if they get different answers, the question is
not finished — settle the question before arguing the rationale.

Measured in singlefs (2026-09-09), both from the same slot: a rationale was withdrawn on
2026-09-06 for being "mutually exclusive with X", X was cancelled the next day, and the asker
asked for a re-judging, pointing toward revival. The three things the backward-reasoning leg of
a three-way argument hit have nothing to do with whether the exclusivity still holds: another
document had left a replacement rationale that does not depend on it that same day; each of that
replacement's two supports is broken; and the slot is not even asking what the withdrawn
rationale answered — its question, "does it need equal integrity", had two readings (the thing's
own bytes, or the target it points at), the withdrawn rationale answered the first and the
wording asked the second. The slot sat stuck for three days across two rounds of argument, and
closed on the spot once the question was pinned down.

## 0.0.35 — 2026-09-06

**`rules/show-me-test.md`, in "turn traps you have hit into checks that fail", gains a
subsection on reach: a check standing inside one apparatus does not govern the next one.**
Assertions and mutation testing only take effect inside the apparatus they live in; stand
up a second apparatus, model the same thing again from scratch, and that check will not say
a word.

Measured in singlefs (2026-09-06): a counting model took the "capacity × fill rate" budget
for the number of objects of one kind and added a second kind on top of it, which is 126%
of a disk's worth of objects; fixed the same day, with an assertion left behind to go red.
Hours later another model made the same mistake (135%), and that assertion, living in
another apparatus, said nothing — the new apparatus had its own unit tests green, every
mutation caught, the gate green, while the two models reported numbers 1.5× apart for the
same physical quantity with nothing comparing them, and the wrong number went into a
settled clause.

The criterion therefore changes from "was this trap turned into a check that goes red" to
"**which layer is this trap on**": a trap in the behaviour of one piece of code takes a
check next to that code; a trap in the **measurement basis** (how the same quantity is to
be computed) takes a **cross-apparatus** check that forces the two onto the same number.
The quick criterion is "stand up a second apparatus and do it again from scratch — would
you step into it twice?" The section also names where whoever fixes the trap stops most
easily: they really did turn it into a check that goes red, the evidence is complete and
the gate is green, so they never ask again how far that check reaches.

**`rules/evidence-discipline.md`, in "quote an artifact by copying the line whole", gains a
subsection: after a re-run, check the prose back against it — a green replay does not mean
the prose is right.** The replay pins which range the conclusion lands in, and the wider
that range, the further the prose can drift inside it. Measured: an experiment's prose said
1.475× / 22.33× / 5 123 506 ns while its own kept artifact says, verbatim, 1.625× / 30.36×
/ 5 199 857 ns; all three sit inside the range assertions and the gate was green. The same
subsection governs claims a single command could count — "N unit tests", "M mutations", "K
lines of artifact" — five of which were found disagreeing with the source or the artifact
in one repository on one day.

Both are rule text only, with no new gate stage: a cross-apparatus check first needs a
machine-readable annotation on artifacts saying which quantity a number is, and this
package has none; the machine-checkable place for counting claims is each project's own
local stages.

## 0.0.34 — 2026-09-06

**`rules/evidence-discipline.md`, in the section on sweeping a new criterion back over the
entries already on the books, gains a subsection: withdrawing a number or a conclusion
also means sweeping for who cites it.** Same discipline, different object, and it hides
better — whoever withdrew it usually did sweep a few places, so they have every reason to
believe they finished, and the one they missed surfaces days later.

Measured in singlefs (2026-09-06): when the width of a location entry was rewritten, that
decision's own text said "this item settles a number that never went through the decision
process and had already been consumed", and it named and swept two downstream derived
numbers. Three days later a third citation in another document still carried the old
value, was copied into the background material of a three-way argument, all three legs
inherited it, and every argument resting on that number was voided for the whole round.
Two places swept, one missed, and the one who swept did not know it.

The criterion is written as: you have finished withdrawing only when you can produce the
list of everything in the repository still using that number. The scope is stated as the
whole repository rather than memory, because memory hands you exactly the easy ones. The
section also states that this and the existing "background material fed to a multi-party
argument must itself be checked first" are two ends of one hole, and that both ends have
to be plugged.

Rule text only, no new gate stage: there is no machine-checkable form today, since
deciding that a number was withdrawn needs a withdrawal registry and this package has
none. A consumer that wants the check should put it in its own local stages.

## 0.0.33 — 2026-09-06

**`install.sh` learns to tell "the project has taken this file over" from "the project is
behind upstream".** It used to judge only "differs from the upstream template", and the two
look identical under that test. But the kb skeleton and the skill stubs exist to be edited
by the project — so **any project that touches its own kb can never refresh its version
stamp again** after the first install, and gate stage 0 stays red forever. Measured on
singlefs: all 12 "behind" files were the project's own edits (`.claude/kb/INDEX.md` carries
36 lines of the project's own wording).

A project lists the files it has taken over in `$ROOT/.claude/install-owned`, one entry per
line, `<relative path>  # why`. **The reason is mandatory**: taking a file over means
upstream changes to it will never reach you again, and the reason column is where that gets
faced head-on. The list is always reported in the output — quietly skipping a few
comparisons looks exactly like this guard never having been implemented.

Two things go red: no reason given; and a path `install.sh` does not lay down at all (such
an entry does nothing, while leaving people believing that file is already exempt). **A
broken list blocks on the spot**; the version stamp is not refreshed past it.

The `README.md` line claiming the version stamp "refreshes every time" was wrong and is
fixed here — it had long since parted ways with what `install.sh` actually does.

Five mutations run one at a time. One of them exposed a blind spot: deleting the "a broken
list blocks" branch left both existing fixtures **green together** — they also had content
behind upstream, and that was the path making them red. It took a fixture with nothing else
behind to isolate it. Self-test 182 → 188.

## 0.0.32 — 2026-09-06

**gate-lint gains the third rejection shape: a printed `✗`.** Until now it recognized only
`lib.sh`'s `bad` and `die`, while **a project's own local stages mostly do not source
`lib.sh`** — they `echo "  ✗ …"` directly, or put the criterion inside embedded python:
`print('  ✗ …')`. That whole class of rejection had never been checked.

Measured on singlefs's local stages: 46 such rejections, all exempt; with the check in
place, **14 of them have no next step at all** (`15-research-build.sh`'s "no cargo",
`70-citations.sh`'s "cannot find $S" — both `exit 1` without telling anyone what to do).
Rejections covered by gate-lint: 125 → 170.

Two parts of the criterion were calibrated on the real corpus:

- **The window is not 5 lines; it runs to the next rejection.** Python often prints one
  `✗`, then loops through the offending items, and only then gives the remedy — a 5-line
  window misjudges that whole batch (fixture `printrejok` watches this).
- **Heredoc bodies are not read as code, except interpreter heredocs.** The `print` inside
  `python3 - <<'PY'` really is shown to the submitter; skipping it as before would leave
  python-implemented stages blind as a group (fixture `printrejpy` watches this).

Both exemptions must be written out explicitly; nothing gets guessed. A per-item line
inside a loop carries `# gate-lint:detail`, a summary line carries `# gate-lint:summary`.

`rules/sop-first.md` gains a third row in its table of rejection shapes. Four fixtures,
four mutations run one at a time, each caught by exactly one case. Self-test 178 → 182.

⚠️ **This turns singlefs's gate red in 14 places**, all inside its own `.claude/gate.d/`.
The three upstream repos are unaffected — the shared package has no printed rejections at
all; everything goes through `bad`.

## 0.0.31 — 2026-09-06

**New rule `rules/pushback-discipline.md`: a proposal is not exempt because of who made
it.** A plan the user proposes goes through the same gate as a plan from anywhere else.
When it contradicts measured data, or a fact already checked, three things get said
*before* anything is touched: which item it contradicts (with its source), what goes
wrong if you follow it (and what observation would show that happening), and whether a
third path exists. If they restate it, do it and stop arguing — but **the warning gets
recorded**. **Finding the mismatch only after the work is done changes nothing: say so, and
withdraw the whole thing if that is what it takes** — cost is the user's to carry, code
answers for correctness, and "we already built this much" is not a reason to continue.
Among code changes nothing cannot be taken back: reverting used to cost human time, that
part is the machine's work now, and tens of thousands of lines is not a different order of
magnitude from a few hundred. What genuinely cannot be taken back — `git checkout`,
`rm -rf`, permanent outward commitments — is `command-safety.md`'s business.

**Warnings live in `.claude/warnings/<date>.md`**, one file per date, one `##` section per
warning, all four items present: Proposal / Objection / Known risk / Outcome. Everywhere
else links here and copies nothing (`kb-discipline.md` §4). One file per date exists so
that **no old file ever has to be edited** — a warning, once written, is the record of
that day. `install.sh` lays the directory out in the project.

**Two new doc-lint checks**:

- Files under `.claude/warnings/` must be named `YYYY-MM-DD.md`, and every `##` section
  must carry all four items. The item names are taken per language and all three run:
  the check recognizes fixed item names rather than guessing semantics from a
  character blacklist, so it does not fall into the "no word list, not implemented" tier.
- `rules/*.md` and the `@rules/` references in `CLAUDE.md` must match item for item. Both
  directions fail silently: an unreferenced rule never enters the context yet looks
  exactly like one in force, and a reference to a missing file simply does not expand,
  leaving CLAUDE.md reading as complete. This one is what this round itself needed —
  adding a rule file and forgetting to list it in CLAUDE.md went red nowhere.

Seven fixtures; five mutations run one at a time, each caught by exactly one case.
Self-test 171 → 178 cases.

## 0.0.30 — 2026-09-06

**The gate in the en and ja repos goes green again.** Two holes, both left by copying one
set of rules into several language repos:

- `doc-lint`'s fixtures are written in Chinese, while `scripts/` is copied byte-for-byte
  into every language repo. In the en repo those fixtures were then judged against
  `## Revision history` — 44 cases red at once, with nothing wrong in doc-lint itself.
  Language is a property **of the fixtures**, not of the repo: `selftest` now pins it with
  `DOC_LINT_LANG=zh`. That knob can swap the criteria out, so setting it **prints a line**,
  and a self-test case watches that the line is still there. The hole dates from 0.0.25,
  when the criteria became language-dependent; en and ja have been red ever since.
- The en repo translated the placeholder short names of both D1 and E1 in `templates/kb/`
  as `<name, 24 characters or fewer>`, colliding with "no short name may be shared by two
  numbers". Each now carries its own noun; zh and ja already had them apart.

**doc-lint gains an exclusion list: evidence kept verbatim must not be edited afterwards.**
Directories like `research/prompts/` hold the prompts as they were sent to the model; they
correspond one-to-one with the artifacts, and changing one character means the artifact no
longer corresponds to its input. A project declares them in `.claude/doc-lint-exclude`, one
entry per line, **each with its reason written out**. An entry pointing at a directory that
does not exist, or one that excludes no file at all, goes red — an exclusion that does
nothing leaves people believing those files are already steered around. Every exclusion is
reported in the output: quietly skipping two hundred files looks exactly like the check
never having been implemented. Rule in `rules/evidence-discipline.md`; seven fixtures,
including a control that carries no exclusion file.

## 0.0.29 — 2026-09-06

**The "superseded by X" pattern now fires only on the numbering schemes this SOP governs**
(`D` decisions / `E` experiments / `C` owed checks / `I` invariants / `A` premises / `O` oracles).
Measured false positive: a sentence about on-disk bytes — "to rebuild U1 you need U2's original
bytes — already overwritten by U9" — was reported as an in-place "superseded by" annotation.
There the word means bytes being written over, not a clause being overturned. The two senses
collide in one word, and the only thing that separates them is whether the thing doing the
overwriting is a numbered clause.

## 0.0.28 — 2026-09-05

**The item-count criterion now recognizes python f-strings.** Recognizing only shell's
`$n` made every python-implemented stage a false positive — their success lines are
formatted inside python (measured on two singlefs stages). Fixture `countpy`; `nocount`
still goes red, so the criterion is not hollowed out.

## 0.0.27 — 2026-09-05

**Fixes an infinite loop 0.0.26 introduced, and adds a timeout to selftest.**

- When `gate-lint` strips arithmetic expansion, the closing `))` is now sought only
  **after** the `$((`. Searching the whole line for the first `))` hangs: on a line like
  `if ((okc)); then …; pass=$((pass+1)); fi` the first `))` sits before the `$((`, so the
  remainder still contains `$((` and never gets shorter. Real prose triggered it at once —
  nine minutes without finishing, neither red nor green. Fixture `arith2` must run to
  completion and be green.
- Both fixture-execution points in `selftest` gain a `timeout` (60s default, override with
  `SELFTEST_TIMEOUT`). **A hung check is more dangerous than a red one**: in the gate's
  output it is nothing at all, it simply never comes back.

## 0.0.26 — 2026-09-05

**Project-local stages come under the gate's own governance, plus three disciplines
extracted from three days of singlefs work.**

**1. `.claude/gate.d/` is now scanned by both lints**

`gate.sh` hands the project-local stage directory to `gate-lint` and `shell-lint`.
Until now it was in neither lint's scan — those scripts reject submitters exactly like
shared stages do, yet had never been checked: the first run over them produced 7
rejections with no way out (measured in singlefs).

**2. gate-lint gains a third check: the success line must report how many items were checked**

"Scanned zero items" is not passing. Write a criterion's search scope a little too
narrowly and every object is skipped at the first step — neither pass nor fail, and the
tail still reports green (singlefs C114: one stage's third check sat green exactly like
that). Only scripts that **report success** are judged; scripts that genuinely scan
nothing write `# gate-lint:nocount <reason>`. Fixtures `nocount` / `countok`; this
package's own `show-me-test.sh` was the first thing it bit.

**3. gate-lint false positive fixed**: in `bad=$((bad + 1))` the `bad ` sits right after
`(`, which CMD_POS read as a rejection. Arithmetic expansion is now stripped before the
match; fixture `arith`.

**4. Rule additions** (each with its measured cost)

- `evidence-discipline`: **quote an artifact by copying the line whole** (the same
  difference misstated four rounds running); **sweep a new criterion back over entries
  already on the books** (the ruler was swung only at the newcomer); one row added to the
  sampling table ("does the same artifact hold a counterexample"); the straw-man section
  gains "carry an arm's definition along with its number".
- `show-me-test`: proving a check goes red **also governs design argument** — build the
  must-report-non-zero world first, then change the rule.
- `skills/decide` hard requirement 6: **touch a clause a person settled and you owe an
  open entry in `checks-owed.md`**.

## 0.0.25 — 2026-09-02

**The last of the Chinese left in the translation repositories is gone, and the
`agents/` layer is stood up.** Both landed as failing checks, not as notes.

**1. skills / templates move from "copied verbatim" to "translated per file"**

They used to sit in `i18n-sync`'s `SHARED` and were copied byte for byte, so this
repository's skill bodies, project skeleton and litmus comments were all in Chinese —
and `templates/CLAUDE.project.md` is what `install.sh` writes as the user's project
`CLAUDE.md`. Directly at odds with "someone working in English never has to read
Chinese".

- The manifest grows from 14 to 28 entries: `CLAUDE.md`, `rules/`, `agents/`,
  `skills/`, `templates/` (including `*.litmus`). One criterion: is there prose
  written for people in it?
- **New coverage check**: any text in neither list turns red on the spot. This is the
  machine-checkable form of "a blank" — add a `.md` and forget to decide whether it is
  translated, and the gate remembers for you.
- **The provenance stamp's position and comment syntax now depend on the file type**:
  a `SKILL.md`'s YAML frontmatter must start on line 1 (put the stamp above it and the
  skill silently fails to install); a `.litmus` must start with `C <name>`, or herd7
  reports `splitter error in sublexer first line` (measured by running herd7).
  The stamp is read back after writing, and both traps are now failing checks.
- **`install.sh` strips the stamp when laying files into a project**: it is the
  distribution layer's bookkeeping, and copied into a user's project it becomes a
  stale annotation that will never be updated, sitting on a file they are about to
  edit (found by actually running an install).
- 28 translated files (14 each for en and ja) rewritten. The litmus templates in all
  three languages were run through herd7: identical verdicts.

**2. `agents/` is governed**

`agents/INDEX.md` states what belongs there (the line against `skills/`: read, versus
delegated), how to write one, three disciplines, and how it is wired in. It is in the
manifest and translated per file, in `GOVERNED`, and `install.sh` lays down agent
stubs. Empty is a state and gets said out loud: while the directory is empty,
`manifest.sh` reports "governed, currently empty" rather than passing silently.
doc-lint now also requires an agent definition to keep no history section and to carry
`name` (matching the filename) and `description` — neither of which errors when
missing; they just silently do not take effect.

**3. Fixed along the way** — every one surfaced by feeding the self-test a fixture

- `manifest.sh`'s `gen()` lost its `cd` inside a pipeline; run from the repository root
  the cwd happened to be the package root, so it never showed.
- `find` against a non-existent directory failed the whole of `translated_paths`:
  `2>/dev/null` hides the message, not the exit code, so `--update` exited 1 printing
  nothing at all.
- `stamp_read` used sed, where BRE's `\|` collides with the delimiter; it silently
  matched nothing and was caught only by the read-back assertion after stamping.
  Now awk.

Discriminating power: cases 111 → 120. Every new check was mutation-verified — green
before, red after. Five of them were blind spots this round created itself (the
coverage check, "line 1 pushed down", stripping stamps on install, the agent `name`
check, `agents/` in `GOVERNED`); they only bite once their fixtures exist.

One note on process: both instances of "copied the manifest without retranslating"
this round were mine. A batch edit script aborted partway, one language never got its
edit, and both were stamped anyway — with the gate fully green. A provenance stamp can
only show that someone claims a file was retranslated, never that it was. Comparing the
content word by word is what caught it.

Still open: the `Note` column in GLOSSARY is still in the reference language — the last
language gap in this package, recorded explicitly at the top of that file.

## 0.0.24 — 2026-09-02

**This version was never released on its own; it ships in the same commit as 0.0.25.**
Downstream sees 0.0.23 jump straight to 0.0.25 — recorded here so nobody goes looking
for a 0.0.24.

Second adversarial audit, driven by mutation testing. **The theme is discriminating
power**: of the gate checks added last round, six could be deleted outright and
`selftest.sh` still reported 54/54 green — the check was there, the thing watching the
check was not.

- **Discriminating power**: single-purpose fixtures for doc-lint's history-statement
  pattern set / `CLAUDE.md` history section / kb history section, gate-lint's `howto`
  window value, lkmm's two static checks, and show-me-test's set of test annotations.
  The `blind` fixture's `want` was `"body must not"` — shared by six different checks —
  and is now split per check. All eight mutations now go red. Cases 54 → 69.
- **A dead pattern in doc-lint**: `'[〔【\[]已废弃[〕】\]]'` can never match under GNU
  grep — POSIX makes a backslash literal inside a bracket expression. Fixed.
- **`die` is no longer exempt**: `die` is `bad` + `exit`, so it is a rejection.
  `lib.sh`'s `die` now takes "message + remedy"; gate-lint fails a `die` carrying only
  one argument; 17 call sites got their remedy.
- **New `shell-lint.sh`**: turns two mechanically checkable rules from
  command-safety into failing checks — killing processes by pattern match, and
  carrying a value out of a subshell through a variable.
- **`$QEMU_LOG` in `qemu/run.sh`**: assigned only inside a function that always runs
  in `$( )`, so all six parent-scope references were `unbound variable` under `set -u`
  — **five failure branches died before printing their `howto`**. The caller now owns
  the work directory. This is the exact pitfall command-safety itself documents.
- **The gate skill's "prove it goes red" example measured green**: `>>` appends past
  the "Revision history" heading, where the body scan stops. Replaced.
- **The spec proper is defined once**: `GOVERNED` in `version-discipline.sh` is
  authoritative; `CLAUDE.md`, README and session-wrapup each carried a different list
  and now link to it. `README.md` and `I18N` are now governed.
- Deduplication and rot: the multilingual paragraph duplicated between `CLAUDE.md`
  and README (already drifted) now lives only in README; the installer does six
  things, not three (including writing `litmus/` at the project root); the gate
  skill's stage table went from 4 rows to 9; 15 places hard-coding "three languages"
  now follow `languages=` in `I18N`.
- `gate.sh`: "upstream freshness not checked" now appears in the summary's
  "not run this time" list — it used to warn once at the top while the summary
  reported "all N stages passed".
- Rule additions: sop-first's remedy requirement now covers `die`;
  design-doc-discipline carves out pitfall comments in checking code;
  kb-discipline states that `INDEX.md` keeps no history section;
  command-safety marks which two of its rules are now checks.
- Post-review round: the English `show-me-test.md` had lost a negation
  ("it guarantees nobody reaches correctness"); gate-lint and shell-lint now scan
  the package root, so `install.sh`'s four remedy-free `die` calls were caught and
  fixed; the `howto` window is 5 lines counting the `bad` itself, not 4.

## 0.0.23 — 2026-09-02

Batch of gate fixes following an adversarial audit (report in that session's log):

- doc-lint: unclosed code fences go red; a `##` section after "history" goes red;
  invariant definitions inside history tables no longer count; history sections in
  rules files go red; the rule-definition marker is restricted to CLAUDE.md /
  rules / skills; exempting a registered number via not-numbers goes red;
  compound words like 脚本文件 / 同上游 no longer false-positive as references.
- gate-lint: `bad` is recognized in all forms (single/double quotes, variables,
  after `;{|&` / `then` / `else`); `howto` inside comments doesn't count as a
  remedy; the summary exemption is tightened to "失败/未通过/未过：$counter".
- Show me test extracted into show-me-test.sh: `#[test]` in comments doesn't
  count; tests/ only counts `.rs` files; build.rs is code too; on the default
  branch the diff base retreats to HEAD~1, so commit-first no longer yields
  "nothing to judge".
- lkmm: every Never litmus needs its own `-nofence` paired control; one global
  Sometimes no longer covers all; static checks run before the herd7 probe;
  every rejection gained a howto.
- i18n-sync / manifest: the diff diagnostic pipelines in failure branches used to
  kill the script under set -e + pipefail (losing the howto and remaining
  languages) — guarded with `|| true`; i18n-sync now verifies manifest freshness
  first; GLOSSARY.md joined the shared (verbatim-copied) set.
- New version-discipline.sh: changing governed content without bumping VERSION
  goes red (was a reminder sentence).
- selftest generalized to all gate scripts: fixture dirs for doc-lint /
  gate-lint / lkmm plus scripted cases for show-me-test / version-discipline /
  manifest / i18n-sync — 54 cases.

## 0.0.22 — 2026-09-02

- session-wrapup gains "are other sessions flying in this repo"; show-me-test
  gains the mutation list; test-discipline gains "deterministic model × N runs"
  and "mutation testing proves assertions can go red"; GLOSSARY gains
  mutation-testing terms.

## 0.0.21 and earlier

See `git log --oneline` — each commit message is written as "version: what changed".
