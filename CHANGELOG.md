# CHANGELOG

Version history for the rules and the gate. `CLAUDE.md` and `rules/*.md` keep no
history sections (design-doc-discipline); history lives here. For per-change
detail see `git log` — commit messages are the change notes.

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
