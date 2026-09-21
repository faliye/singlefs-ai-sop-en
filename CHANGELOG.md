# CHANGELOG

Version history for the rules and the gate. `CLAUDE.md` and `rules/*.md` keep no
history sections (design-doc-discipline); history lives here. For per-change
detail see `git log` — commit messages are the change notes.

## 0.0.56 — 2026-09-21

**The consumer's name is now gone from script comments too; twelve of its local checks are taken upstream; and a use-before-definition in 0.0.55 that cut downstream gates in half is fixed.**

Regression fix (`scripts/gate.sh`):
- The "agent definition discipline" stage added in 0.0.55 called `run_rules_lint` dozens of lines before it
  was defined. The SOP repository has no `.claude/agents/`, so that `if` was never entered — all three gates
  were green while any project with agent definitions died on that line: 121 items run, no summary, and not
  one of the later stages executed. That stage was redundant anyway, so it is gone; the project-local rules
  stage now also runs when only `.claude/agents/` exists.
- **New exit guard**: the summary line must be printed; exiting early now says so on stderr and returns the
  original exit code. "Did not run" and "ran and passed" used to look identical.

De-branding (clause 8 of `rules/rules-discipline.md`, carried through):
- 28 measurement provenance comments in scripts, 4 in gate fixtures, plus `README.md`, `install.sh` and
  `templates/` — all rewritten to name no one. The one occurrence in *code*: the git ref
  `refs/singlefs/gate-ok` is now `refs/sop/gate-ok`.

Twelve consumer-local stages taken upstream:
- `doc-lint.sh`: twelve history-tone patterns, each with its own reason and its own fixture line.
- `shell-lint.sh` gains **S7**: a pipefail pipeline ending in an early-exiting `grep -q` reads a hit as a miss.
  Upstream's own `lib.sh` had two of these; both fixed.
- New `script-modes.sh`: **upstream's own changelog-lint, show-me-test and version-discipline had lost their
  executable bit in the index**; fixed here.
- New `stage-selftest.sh`, `link-targets.py`, `history-ordinal.sh`, `hooks-registered.sh`,
  `relay-timing-lint.py`, `number-name-sync.sh`.
- `rules-lint.sh` gains one lexical pattern; `gate.sh` gains `run_stage_may_skip` (exit 77 = not run this round).
- The corresponding 10 local stages and 2 library scripts are deleted downstream; no criterion was lost —
  each was measured against the consumer project first.

Selftest grows from 373 to 395 cases.

## 0.0.55 — 2026-09-21

**0.0.54 widened the reference checks to normative texts and caught the work objects along with them; singlefs's temporal-reference check is taken upstream.**

Rules (`rules/`):
- `kb-discipline.md` clause 1 gains **temporal references**: "this round", "that round", "the previous
  round" are forbidden; write the anchor instead (a date plus "that round", or the experiment's number). The gate's
  criterion is whether this line, or the nearest heading above it, carries a date. Three are not checked:
  "the next round" is a generic; "this time" / "last time" hang on technical objects; history entries all
  hang under dated headings.
- "over" / "under" in clause 1 move from "outside the gate" to **checked in the two forms carrying 见**
  ("see above", "see below"). The bare 下一节 ("the next section") is added too, narrowed symmetrically
  with 上一节 so that "the next node" and "the next tick" are excluded.
- `rules-discipline.md` clause 7 now states how normative texts differ from kb: self-reference is judged
  only for words naming a document structure; domain objects ("this experiment", "this decision") are not,
  because a normative text is about how to handle those objects; temporal references are not judged either,
  since a rule holds for every round.
- `rules-discipline.md` gains clause 8, "The body carries nothing specific to one consumer": the body may
  not name a project that uses these rules, nor its paths or file names, nor use its decision or experiment
  numbers as examples. The consumer list is registered in `I18N` under `consumers=`. All three languages were
  swept against it: 3 named references rewritten, 5 downstream numbers replaced by the generic "number (short
  name)" form, and the project name in skill descriptions replaced by "this project".

Scripts (`scripts/`):
- The self-reference patterns in `doc-lint.sh` are split in two. 0.0.54 applied the kb set verbatim to
  normative texts and misjudged 5 places in singlefs's `.claude/agents/` and `.claude/rules/`.
- Two forms added to the contextual-reference patterns: "see above" / "see below" and the bare 下一节.
  Both were judged only by singlefs's local stage `20-kb-shape.sh`; upstream had missed them.
- **The temporal-reference check is taken upstream** from singlefs's local stage
  `.claude/gate.d/25-kb-deictic.sh`: the criterion is general (it governs how documents are written and does
  not depend on the filesystem), so it now lives in `doc-lint.sh`, for `kb/*.md` only. Compounds like 样本轮
  are excluded on the left boundary — measured against singlefs's gate fixtures.
- `rules-lint.sh` gains an eighth criterion: a consumer project's name in a rule body goes red. The list is
  read from **the scanned repository's** `I18N` (`consumers=`), with the package's own `family=` string
  stripped first — an upstream script does not hard-code a downstream name. A project root has no `I18N`,
  so the criterion has nothing to judge when a project scans its own rules, and the success line says so.
  New fixtures `rules-lint/consumer` (red) and `consumerok` (green).
- New fixtures `doc-lint/kbround` (red) and `kbroundok` (green); the `selfok` fixture gained a date anchor.

How these were shown to fail: run against singlefs's 443 files, every reference-class red disappeared with
no false positives; the temporal check was cross-checked against its predecessor on the same kb set. 371
selftest cases pass.

## 0.0.54 — 2026-09-21

**Freezing evidence binds this round only; a change that did not reach the whole repository must register every file it skipped; and the body of a normative text may not point at a position.**

Rules (`rules/`):
- Three paragraphs added under "Evidence kept verbatim may not be edited afterwards" in
  `evidence-discipline.md`: the freeze holds within one round — once this round is committed, the next
  commit deletes the batch the previous one left behind, this round's stays, and this round's verification
  still works; to find what was deleted, go to the version history. Old evidence is a reference, not a basis:
  when an earlier conclusion is in doubt, verify it again rather than digging through old evidence; to
  overturn it, run another round under the project's own inference discipline. The check that body text
  matches its artefacts is judged against **this change**, not against the whole repository.
- New section in `show-me-test.md`, "When a change does not reach the whole repository, every file left out
  must be registered": when a norm, an operation or a change covers only some files, each file not covered
  goes into an exclusion table, one per line, with the reason after `#`. **No exclusion table means the whole
  repository must be changed.** The criterion is "does this sentence become false, or does some gate stop
  working, if it is changed". An exclusion pointing at a path that does not exist goes red, and the table
  must be readable by a single command.
- New clause 7 in `rules-discipline.md`, "No positional references in the body, and no self-reference":
  the scope is `rules/*.md`, `CLAUDE.md`, `agents/*.md` and `skills/*/SKILL.md`; the criterion is the same
  one `kb/*.md` is held to, with the per-word scope in `kb-discipline.md` clause 1. Write the position as a
  name — a section becomes its heading, a table becomes the thing it judges, a fact elsewhere becomes a link
  to the file it lives in. In a rule file carrying the `<!-- doc-lint:rule-definition -->` marker, only the
  examples inside backticks and 「」 are exempt; the body itself is still judged.
- The closing line of `show-me-test.md` changed from "the quality of the gate is this project's ceiling" to
  "the quality of the gate is how high this project's floor sits": inference 3 in the same file says the gate
  is **a floor, not a ceiling**, and the closing line used the same word for a different quantity.
- All three languages swept against clause 7: `CLAUDE.md`, `command-safety.md`, `engineering-philosophy.md`,
  `evidence-discipline.md`, `kb-discipline.md`, `machine-first.md`, `show-me-test.md`, `sop-first.md`,
  `writing-discipline.md` and `skills/gate/SKILL.md` now name what they used to point at. Forms the gate's
  patterns cannot reach ("the second line", "the paragraph above", "this table", "this file") were fixed by hand.

Scripts (`scripts/`):
- `sb superblock` dropped from the abbreviation list in `naming-lint.sh`: an abbreviation exists for one
  concept; when the concept goes, the line goes — it is not repointed at a different expansion.
- The reference checks in `doc-lint.sh` (contextual references, self-reference) now cover normative texts as
  well as `kb/*.md`: `rules/*.md`, `CLAUDE.md`, `agents/*.md`, `skills/*/SKILL.md`. They scan the body with
  examples carved out, so the tables that enumerate these very words do not judge themselves, and the failure
  message now distinguishes "kb body" from "normative body". New fixtures: `fixtures/doc-lint/rulesref`
  (red — all 6 positional references across `rules/` and `CLAUDE.md` must be reported) and `rulesrefok`
  (green — in a marked rule file, examples inside backticks and 「」 are not judged).

Cross-language (`I18N`):
- The Japanese `show-me-test.md` regained the paragraph it was missing (evidence is the same measure for
  everyone, and the submitter can apply it in advance).
- The English and Japanese `CLAUDE.md` wiring tables gained the agents row, which only the Chinese one had.

## 0.0.53 — 2026-09-20

**Rule files carry the how only: a fourth document discipline, a check that fails, and history cleared out of the rules in all three languages.**

Rules (`rules/`):
- New `rules-discipline.md`, covering `rules/*.md` and a project's `.claude/rules/*.md`. The body carries four things only (how, what to do and not do, what to do by default, which rule lives where); the criterion stays, the argument moves out; measurements, arguments and decision records lifted out of the body are deleted outright, with no new home — a shared rule's history goes where it always did, `CHANGELOG.md`, one section per version; a link to history must carry the deterrent "do not read it unless you are tracing where this came from", with not one word dropped; rule files keep no history section.
- The third row of the three-document table in `writing-discipline.md` now points at it; the authoritative statement of "`CLAUDE.md` and `rules/*.md` keep no history section" moves from `design-doc-discipline.md` into the new rule.
- All 14 rules, `CLAUDE.md`, `agents/INDEX.md`, `skills/crash-test` and `skills/decide` swept against the new rule: measurements, argument sections, decision dates and gate-discrimination write-ups deleted from the body; the body keeps the how. Section structure aligned file by file across the three languages.

Gate (`scripts/`, one copy across the three repos):
- New `rules-lint.sh`, judging seven things: record sections, argument sections, dated lines, explanatory paragraphs and half-sentences, lexical explanations, and history links with no deterrent. Four exemptions: dates inside 「」, threshold dates inside backticks, table rows (exempt from the last three), and files registered in the exclusion list. Its patterns are Chinese, so a repository of another language reports not implemented (exit 77) rather than passing silently.
- Two `gate.sh` stages: "rule discipline" scans this package's `rules/`, "rule discipline (project-local)" scans the project's `.claude/rules/`. Both also scan `CLAUDE.md`, `agents/*.md` and `skills/*/SKILL.md` — they are executed the same way rules are. A project needs no stage of its own in `.claude/gate.d/`.
- `selftest.sh`: 15 new rules-lint fixtures (each red fixture breaks exactly one thing; the green ones sit on the edge of all four exemptions); `gate.sh`'s stubs and wants updated for the new stages.

## 0.0.52 — 2026-09-19

**The ban on finding processes by pattern now also covers commands typed in a session; a new `scripts/proc.py` does the same work by process ID; checks within one script go parallel where they can, while "a bare `wait` swallowing the failures" becomes a failing check; shell-lint's S1 loses two false reds.**

Gate (`scripts/`, one copy across the three repositories):
- The criteria for `pkill -f` / `killall` / `pgrep -f` move from shell-lint into `lib.sh` (`PATTERN_KILL_RE`, `PATTERN_PGREP_RE`) so they are defined in one place; expanded, they are byte-for-byte the old S2 and S3.
- New Claude Code PreToolUse hook `scripts/claude-hooks/pattern-process-guard.sh`: judges commands typed in a session with the same criteria, command positions only. Heredoc bodies fed to non-shell commands are not judged; heredocs fed to a shell and the strings of `-c` and `<<<` are. A hit exits 2 and gives the process-ID alternatives. 19 samples (`fixtures/pattern-process-guard/`), each shown to flip under a breakage aimed at it. What it cannot reach is written in its header: prefixes outside the command-position table (`timeout 5 …`, `nohup …`, `ssh host '…'`) slip through, and quoted text that happens to follow `( | ; !` is refused.
- New `scripts/proc.py`: `find` (exact executable-name match, never lists the branch that issued the command), `wait` (by process ID, timeout required), `stop` (TERM then KILL, refuses itself and its ancestors), with `--selftest` and three breakage switches; selftest gains four cases. The selftest's sleep argument now carries the process ID: with a fixed number, the three language repositories running selftest in parallel saw each other's processes.
- shell-lint's S1 (an assignment inside a subshell does not reach the parent) loses two false reds. For a function whose definition and closing brace sit on one line, searching downward for the closing brace took the next function's line-initial `}` and counted the top-level assignments in between as its body (all 14 false reds in singlefs's research scripts had this shape); it is now judged on that one line. A direct call indented inside another function body was not recognized; leading indentation now counts as a command position. Samples onelinefunc (green), onelinebody (red), indentcall (green).
- New shell-lint check S6: a `wait` with no arguments in command position turns red. Its exit code is always 0, so however many of the parallel checks went red the parent process never sees it — a gate that can no longer go red after being made parallel is worse than a slow serial one. Where the exit code really is collected elsewhere, `# shell-lint:exit-collected <how it is collected>` on that line lets it through, and the reason may not be left out (a marker with no reason is its own kind of red). Samples: parallelwait (red, one of each form) and a correct parallel block with the marker added to good (green); three mutations each flip the one spot they aim at.
- selftest's six batches of fixture cases (doc-lint, gate-lint, shell-lint, pattern-process-guard, changelog-lint, naming-lint) now run in parallel: when handing work out each item writes its exit code to its own `.rc` file, and collection judges them one at a time in the order they were handed out — no part of the judging is parallel. However many items go out, that many must come back; a missing `.rc` turns red. All 350 verdicts are case-for-case identical before and after, and the stage went from 24.8 s to 21.4 s, 66% CPU to 105%. Measured while writing it: with `set -e` in force inside the subshell, a sample going red exited on that very line and never wrote the `.rc` — 119 cases all missing their exit code; taking the exit code inside an `if` fixed it.
- doc-lint's archaic-phrasing word list loses one false red: `是故` was hitting the ordinary word 是故意的 ("deliberately"), now `是故[^意]`; sample stylefalse guards this exclusion and two others.
- selftest cases 323 → 351.

Rules:
- command-safety: the "`pkill -f` / `killall` are banned outright" section adds the `proc.py stop`, `find` and `wait` forms; "typed command lines are out of its reach" becomes: the hook refuses them before they run, a project registers it once in `.claude/settings.json` (the snippet is given), and in a session without it only this text applies.
- command-safety gains two sections: "Within one script, run the checks in parallel when they can be" (the criterion, the three cases that do not go parallel, taking the degree of parallelism from `nproc`; once a gate is slow nobody runs it locally and what `sop-first.md` asks for — "runs locally, judges the same as the remote" — fails on the spot) and "Parallelism must not swallow the failures" (the measured table of which four forms do and do not collect the exit code, output must not go straight to stdout, however many items go out that many come back, and after making something parallel prove again that it can go red). The opening "five failing checks" becomes six.

## 0.0.51 — 2026-09-19

**The kb reference check gains the bare positional forms; several lessons measured on singlefs on 2026-09-17 and 09-18 go into the rules; current-state sentences carry no date, without exception.**

Gate (`scripts/`, one copy across the three repositories):
- doc-lint's dangling-reference check gains the bare forms without "see" (上述, 下文, 下表, 以下是, 逐条如下：, 上面三条 and the like).
  The criteria were measured over 357 kb files in singlefs: 103 new hits outside the old criteria, and read one by one, every one was a genuine reference;
  上面 / 下面 ("above" / "below") match only the forms that point at a position in the document, and 前面 / 后面 and 上方 / 下方 are not checked. Fixtures dirref (red) and dirrefok (green).
- The dangling-reference and self-reference patterns are first compiled once by grep on their own, and a pattern that does not compile turns red on the spot:
  that grep is wrapped in `|| true`, so a compile failure (exit 2) looks exactly like "no match", and writing a range of circled numbers turned the whole check silently green for every kb file.
  New selftest case "a pattern that does not compile turns red on the spot"; cases 320 → 323.
- Downstream: scanning singlefs with this version's doc-lint turns two genuine references red (`.claude/kb/experiments/154-…` line 8, "下文 H5…",
  and `.claude/kb/milestone/02-second-txn.md` line 266, "下面的数" and "下面引的六家"); singlefs fixes them.

Rules:
- kb-discipline: item 1 gains the bare forms and the gate's reach for "above" / "below"; under item 8, new "A dated snapshot of the current state is history too, and stays stuck on that day":
  a current-state sentence carries no date, and an existing one has its date deleted and its fact kept. Item 2 now says "dates go only with events",
  and the sentence "the old line at least carries a date" in verify-before-claiming's "Putting it into practice" is rewritten to match — the three used to contradict one another, and are reconciled as the user decided: current-state sentences carry no date.
- evidence-discipline: "An independent count that matches only the total does not confirm a breakdown"; "Sweep by the old wording, not only by the new names".
- test-discipline: new sections "Time phases inside the process under test, not in the loop that relays its output" and "An experiment must state which decision it measures for, and when enough is enough";
  "a positive control's answer must not sit anywhere the side under test can read it"; the list of what runs on people under "Which half the gate handles" gains these three.
- show-me-test: "The reach stops at `.claude/gate.d/`": research scripts and hooks elsewhere in the project need their own local stage handing them to both gate-lint and shell-lint
  (singlefs's research scripts: 84 gate-lint findings, 18 shell-lint findings); corollary 5, "A handed-back list that a machine confirms is complete is not thereby judged right".
- command-safety: new section "Once you start a background task or a subagent, re-check it on a schedule, and do not force it to end". Record-and-report-only applies to tools that detect stuck or spinning work;
  a hook that refuses one specific dangerous form is not covered (read broadly, the original sentence clashed with session-wrapup item 4).
- session-wrapup: "Do not write it into a session's private memory".

## 0.0.50 — 2026-09-16

**One full audit: three real bugs on the `--staged` path, doc-lint exempting the rules proper wholesale, and a
batch of leftovers from after 0.0.48, fixed together.**

Gate (`scripts/`, one copy across the three repositories):
- The `--staged` handshake variable `GATE_STAGED_FROM` leaked into the child processes the inner run started: the
  `gate.sh` that selftest runs nested took it as its own project root when looking for sibling directories, and the
  "upstream older than copy" / "copy behind upstream" cases went red by mistake — on 0.0.49, `--staged` in singlefs
  could not pass. A user-set `GATE_BASE` leaked into selftest the same way, and two show-me-test cases exited 3.
  `gate.sh` now unsets `GATE_STAGED_FROM` as soon as it has read it, and selftest starts every child with both
  stripped (`env -u`).
- `--staged` could not run on the SOP repository itself: the inner run used the source repository's scripts while
  `ROOT` was the temporary tree, so it was treated as a consuming project and went red with "version stamp missing",
  and the three stages that run only in the SOP repository silently did not run. When the source repository is this
  package, the `gate.sh` inside the temporary tree runs, and i18n-sync finds the sibling language repositories from
  the source repository's parent directory.
- `--staged` used a diff base one notch narrower than a direct run: the temporary tree is a detached HEAD,
  `@{upstream}` does not resolve, and it fell back to `HEAD~1` — so the "split it into two commits and it slips
  through" hole was open on that side (measured: direct run `origin/master`, `--staged` `HEAD~1`). The base is now
  computed in the source repository and passed in.
- shell-lint's S3 went red on `pgrep -f` only when the same line also contained one of the substrings
  `xargs|kill|until|while|if`: `pids=$(pgrep -f X)` with `kill $pids` on the next line, or `while` and `pgrep -f`
  written on separate lines, were all green, while `pgrep -f notify` was judged a wait loop because it contains `if`.
  Now any `pgrep -f` in command position goes red, with the fixture `pgrepsplit`.
- "Version stamp missing" recorded a FAIL but printed it through `warn` with no remedy; gate-lint only recognises
  `bad` / `die` / `✗`, so it could not see it. Now `bad` + `howto`.
- doc-lint's `rule-definition` marker exempted the whole file: put padding phrases or "originally X" into `rules/`
  and doc-lint stayed green, while the same sentence in the README went red at once. The body is now checked; only
  the examples quoted in backticks and 「」 are cut out before the word list is applied. The 14 rule files and
  `CLAUDE.md` move from "skipped" to "checked" (checked 14 → 29). Fixtures `rulesprose` / `ruleshiststate` (red),
  `rulesexample` (green).
- doc-lint excluded the installed copy with the wildcard `*/singlefs-ai-sop/*` plus a "don't exclude when ROOT is inside the package"
  exception, so the scan set depended on where the package lives: the same fixture `projok` reported "checked 1" in the zh repo and
  "checked 2" inside the singlefs copy, and once synced there, selftest went red on one case at once. Now `$ROOT/.claude/<family>/*`,
  relative to ROOT; a new case copies the scripts under a path containing `/singlefs-ai-sop/` and runs the same fixture.
- **`--staged` was silently swallowed when invoked through the project wrapper**: the wrapper puts the project root in `$1`
  and appends the user's arguments, while gate.sh hard-coded `$1 == --staged`. The command the rules and the skill recommend
  never entered that branch; the gate ran against the working tree and said nothing. Arguments are now order-free, and an
  unrecognised argument is refused.
- **Heredoc detection treated three non-heredocs as an opening** (`<<<` here-strings, `# … <<PY` in a comment,
  `$((1<<SHIFT))`), and from that line on the whole file went unchecked — the bigger the script, the likelier the hit.
  The delimiter rule now lives once in `lib.sh` as `HEREDOC_RE` (the `CMD_POS` precedent), and comment lines are skipped first.
- **`… | grep -q` at the end of a pipe**: grep exits on the first match, the upstream takes SIGPIPE, and under `pipefail`
  the pipeline returns 141, so the condition reads false. Past the pipe buffer (64 KiB) the check silently stops running
  (measured: the same uncounted-summary script is red at 4 lines and green at 338 KB; a 108 KB new file carrying tests was
  judged to carry none). gate-lint's G3, both places in show-me-test, and doc-lint's invariant-id lookup now use `grep -c`.
- **gate-ok recorded the HEAD at the end of the run**: a commit another session lands mid-run gets stamped as verified and
  then sits outside every later diff window. It now records the HEAD from the start of the run, and records nothing when
  `GATE_BASE` was set explicitly (which includes the inner layer of `--staged`).
- **The diff base took the upstream tip instead of the merge base**: after a fetch without a merge, the upstream tip carries
  commits the local branch does not have, so a clean working tree was judged "crates code changed with no test changes".
  A false red pushes people into working around the gate.
- **`GATE_BASE` was not validated**: one wrong letter (`orgin/master`) made `changed_files` fall back to the empty-repo path,
  show-me-test report "nothing to judge", the gate go green, and gate-ok advance. It is now refused with a remedy.
- **"Is the thing under the gate the SOP repo itself" compared `pwd`**: run through a symlink it was judged a consuming
  project — red for a missing version stamp, with the three SOP-only stages silently skipped. Now `pwd -P`.
- **Running gate-lint / shell-lint standalone inside a project reported the copy's fixtures** as the project's own
  violations. Both lints now exclude `$ROOT/.claude/<family>/`, matching what doc-lint already does.
- **One trailing space after a kb `## Revision history` heading voided the whole rule**: body scanning no longer stopped
  there, and the file was still reported as missing the section. Trailing whitespace is now trimmed before comparing.
- **`x="$(sed -n … I18N)"` exits 2 when I18N is absent**, and `set -e` takes the script down with no output (six sites: gate.sh, gate-lint, shell-lint, two in doc-lint, push-all; round four found two more).
- **pre-push ignored which ref was being pushed**: a feature branch, a tag, even `git push --dry-run` pushed the other two
  language repos **for real**. Only a master push triggers it now; `CLAUDE.md` states the hook's reach and the `--dry-run` trap it cannot stop.
- **Files seeded by install.sh were 0600** (`mktemp` permissions preserved by `mv`); now 644.
- `env.sh` now checks the tools the gate itself depends on and never checked: gawk, sha256sum, timeout, plus
  git ≥ 2.28, bash ≥ 4.3 and `sort -V`. `manifest.sh` pins its sort to `LC_ALL=C`; selftest isolates
  the user's global and system git config.

Round three (the items the first two audits left open, folded in as well):
- **shell-lint S5 knew one spelling**: `rm -rf "$d"/x`, `rm -rf -- "$d/x"`, and an unguarded second argument on the same line were all green. Now two steps: pick out `rm -r`, then check each argument for an unguarded variable path.
- **`--staged` failed when run from a git hook**: the hook's GIT_DIR is relative, points elsewhere once gate.sh changes directory, and the remedy said "run git worktree prune". The GIT_* variables are cleared up front; a failed worktree add prints git's own words and points at `worktree remove --force`.
- **`--staged` cleanup lives in the EXIT trap only**, so every exit path removes the temporary worktree.
- **The version-mismatch rejection hit an unbound variable**: the family name was defined dozens of lines later. Moved above its first use; its remedy no longer tells people to run git log inside a copy that has no .git.
- **Local stages each chose their own diff base**: gate.sh computes it once and exports `GATE_DIFF_BASE` to every stage.
- **"Nothing to judge" is kept apart from "judged"**: naming-lint exits 3 when there is no .rs and the gate records "not run"; doc-lint gains `--not-impl`, and a language without a word list lists those checks under not implemented instead of PASS.
- **gate-lint scans .py too**, applying only "a directly printed ✗ needs a remedy" (13 such lines in singlefs's lib-*.py had never been checked).
- **doc-lint's two "deprecated" patterns shared one message**, so deleting either stayed green; split, each with its own fixture.
- **install.sh could write the version stamp downward** when the copy was older than the stamp, and the downgrade travelled with the commit. Now refused. The wrappers say what to do when the copy is missing instead of bash's bare "No such file".
- **i18n-sync accepted only a `.git` directory** in three places, refusing worktrees. This release was built on worktrees, which exercised all three.
- **Dates may not be invented**: three fixtures and one skill example said 2026-01-01, more than half a year before this repository's first commit, and the gate stayed green. All replaced with real dates: fixtures use their own creation dates, the skill example uses `date +%F`. doc-lint checks history entries and measured-on dates, changelog-lint checks section dates, selftest checks fixture dates. The lower bound is the checked project's own first commit minus 7 days of grace (git is asked only when the directory is itself a repository top level and not a shallow clone, never an enclosing one), falling back to this package's start, 2026-08-26; the upper bound is today in the latest time zone (UTC+14). Why both ends are set this way: see round four.
- Cases added for gate.sh's judgment branches (local stages handed to both lints, show-me-test exit 3, not running the installed copy, .rs without Cargo.toml, `--staged` copying the ignored copy), all of which could be flipped to PASS with no case going red; bump.sh and the i18n-sync `--stamp` rejections had no coverage and now do.
- lib.sh drops the unused `added_lines`.

Round four (three models audited this release, one each on forward derivation, backward derivation and cross-check, over two rounds; the problems they found are fixed here too):
- **The date check judged real dates impossible.** The upper bound was the date on the machine's clock, and that clock is UTC: a same-day date written in Tokyo between 00:00 and 09:00 was judged "later than today",
  and 8 of singlefs's 194 commits wrote history entries in that window dated later than the commit's UTC date. The lower bound was pinned to the first commit, while work done before git init enters the first commit carrying the dates it was done on:
  singlefs's initial commit already holds 5 history entries from the day before, and against 0.0.50 as it stood before the handover its gate was red at those 5 places (measured during the audit).
  The upper bound now follows the latest time zone and the lower bound allows 7 days of grace (measured: only one day early); something like 2026-01-01 is still stopped. Four cases pin both ends, with the clock fixed by a fake `date`.
- **The machinery that names retired wrappers was deleted along with QEMU**, while the same release produced a new retired wrapper: singlefs's `.claude/scripts/lkmm.sh` forwards to a deleted script,
  install.sh refreshed the stamp as usual, and running the wrapper gave the remedy "the copy is not installed". The machinery is back, made generic; see the handover section below.
- **The unimplemented list stopped mentioning the acceptance criterion**: once the two QEMU keys were deleted, a project with no final-criterion stage wired in got not a word about it in the summary. Replaced by `最终判据` (final criterion), which names no apparatus; see below.
- **Two zero-output sites remained for a missing I18N**: doc-lint reading `this=`, and manifest reading `this=` and `reference=`. Both get `|| true`, with one case each.
- **selftest threw the stamping output into `/dev/null` while building translation fixtures**: once the stamp and its read-back disagreed in form, lib.sh's `set -e` took the whole self-test down at case 245,
  the remaining 58 cases never ran, and all that was left was exit code 1. On failure it now prints the original message before stopping.
- **Content unrelated to VMs had been deleted by mistake; it is back**: the "Test images always go in a temporary directory" and "Look before running anything destructive" sections of `command-safety.md`, and `mkfs`, `dd` and `lsblk` in its table.
  Crash-point replay needs images and mkfs just as much, and singlefs's code and scripts cite "Test images always go in a temporary directory" by its heading. The dmsetup check in `env.sh`, which is for the block-layer write recording used by crash-point replay, is back too.
- Wording: the "skip list" paragraph brought into `show-me-test.md` said gate-lint already checks `noskip`; no such check exists, so it now says this is not done yet;
  doc-lint's file-header comment still said `rule-definition` skips the whole file; the pre-push comment put the `--dry-run` trap in the README, when it is in `CLAUDE.md`;
  of the two "irrefutable" in `test-discipline.md`, the previous round had fixed only one; the date item said "five fixtures" where there are three, and the one place in the `fence` fixture that said 2026-01-01 becomes its creation date;
  the "sort does not understand -V" example in env.sh is rewritten so it can be checked (0.0.9 → 0.0.10 would be judged a downgrade, measured during the audit).
- The second audit round then found:
  - **When the machine's date does not understand `-d`, the date lower bound silently disappears**, and 2026-01-01 passes: the date check is called inside an `if` condition, which `set -e` does not cover.
    doc-lint and changelog-lint now try `date -d` once at the start and stop if it fails; env.sh checks this too.
  - **In a shallow clone the "first commit" is where history was cut off**; used as the lower bound, it misjudges more the older the project is. In a shallow clone git is not asked, and the bound falls back to this package's start.
  - **Not one `Measured (<date>` or `実測（<date>` in the en / ja repositories was checked**: only the Chinese `实测（` was recognised, and the unchecked half was not on the unimplemented list either. All three forms are recognised now.
  - **Retired wrappers were recognised only in the current form**: the three-line wrappers laid by 0.0.49 and earlier went unrecognised, and the stamp was refreshed as usual. Both forms are recognised now; a wrapper a project added from the template around a shared script that still exists does not count as retired.
  - **30 days of grace let through too many invented dates**: measured: only one day early; narrowed to 7 days.
  - **No selftest case covered "the working-tree fingerprint does not touch the real index"**: with the fingerprint changed to run `git add -A` directly on the real index, the computed tree was byte-for-byte identical and not one existing case went red,
    while every gate run would quietly stage the user's unstaged changes. One case added, comparing `git status` before and after the call directly.
  - Wording: the merged-in "Write conjunctions as conjunctions" section forbids `⇒`, yet the rule lines newly written in this release had 7 `⇒`; they become conjunctions, and the section now states that the gate does not check this and older text has not been swept;
    the "skip list" sentence in `show-me-test.md` no longer describes an exemption syntax that is not implemented; the `最终判据` passage now says to write the acceptance criterion into the project's kb first, and a stage that does only part of it does not write that key;
    three sentences that said model-based differential testing also needs a recorded write stream are corrected; the measured record in `session-wrapup.md` states that what it hit was the kind that commits without paths;
    the gate skill's stage table gains "Working tree unchanged during the run"; two section headings stacked together in gate.sh are separated; several comments in doc-lint and changelog-lint follow the grace change.
- Evidence that they go red: the 8 fixes the first audit round found — date upper bound, grace period, not looking up to an enclosing repository, retired wrappers, a template wrapper around a shared script that still exists not counting as retired, the two I18N sites, final criterion —
  plus 6 from the second round — `date -d`, shallow clones, the en / ja measured-date forms, old-form wrappers, 7 days of grace, the fingerprint not touching the real index — were each reverted to the old form, and each time the case aimed at it went red.
  Printing the original message when stamping fails has no dedicated case: it was verified by breaking the stamp read-back and watching the self-test stop at the first fixture build and print the original message.

Changes from another session over the same period, folded into this release as well:
- **The gate goes red when the working tree changes while it runs**: gate.sh fingerprints the working tree at the start and at the end (tracked files plus untracked files that are not ignored, computed with a copied temporary index, never touching the real one),
  and goes red when they differ; the remedy is to wait until the edits stop, or use `--staged`. Two selftest cases: editing a file mid-run goes red, staging alone mid-run does not count as a change;
  the "gate-ok records the HEAD from the start of the run" case now writes its content before the run and only moves HEAD during it.
- `show-me-test.md`, two paragraphs: reporting how many were checked is not enough — what was not checked must be listed one by one, with the list computed on the spot; the number in a success line needs a fixture pinning it too.
- `evidence-discipline.md`, two paragraphs: every number that enters a conclusion — first ask whether it was measured or guessed; the list a withdrawal sweep turns up must not be replaced wholesale — first tell whether each hit states the present or what happened on one occasion.
- `writing-discipline.md`, one section: "Write conjunctions as conjunctions; unpack noun strings into sentences".
- `session-wrapup.md`, two items: running without `--staged` goes red if the working tree changes; "commit only these paths" is not `git commit -- <path>`.

selftest 268 → 320 cases: the three rounds of fixes added 58, round four added 12, another session's changes brought in 2, and handing QEMU and herd7 over removed 20 (see below). Evidence that they go red: across three rounds, each fix was reverted to its old form, and each time the case aimed at it went red.
Four of those cases were fake on the first try and stayed silent under mutation: filler lines written as comments in the large-file fixture, an absolute GIT_DIR, a manual cleanup before die, and a local stage failing on its own and masking the lint verdict. Two mutations had broken the case statement's syntax and were redone as whole-line replacements. All were fixed until they went red.

Rules and documents (every language together):
- `command-safety.md`: five failing checks, not four — `pgrep -f` added; the S3 description now matches the script
  (any command position goes red); one padding phrase.
- `sop-first.md`: rejection shapes "two" → three; the two historical sentences ("17 `die` sites were exempt",
  "all exempt until now") deleted or turned into measured records.
- `show-me-test.md`: the local-stages sentence "until now it was in neither lint's scan" becomes a measured record.
- `test-discipline.md`: two "irrefutable" aligned with the section heading's "unfalsifiable"; "this section's main
  point" now names the rule.
- `verify-before-claiming.md`: the heading "whether it is settled and what it actually says are two different
  questions" loses its inner quotes, matching its two citations.
- `kb-discipline.md`: states that "this file" is outside the gate and rests on people. `design-doc-discipline.md`:
  doc-lint scans only `.md`; the code-comment clause rests on review.
- `engineering-philosophy.md`: the howto requirement is in `sop-first.md`, not `show-me-test.md`; the criterion
  sentence now quotes `machine-first.md` verbatim. `machine-first.md`: the paragraph duplicated word for word from
  `engineering-philosophy.md` shrinks to one sentence plus a reference.
- `code-discipline.md`: three disposition labels ("this reason dropped", "kept, and stronger", "left to types")
  folded into the six declared ones.
- `writing-discipline.md`: "Which half the gate handles" adds that the rules proper are checked too, with only the quoted
  examples exempt.
- skills: gate adds the `--staged` usage, lists every failure cause for gate self-check and shell discipline, says
  the rule manifest does not apply under the en / ja copy, and carries the new `rule-definition` semantics;
  crash-test's `gate-covers` keys gain `命名纪律（shell）`; decide: "本工程" → "本项目".
- templates: the three kb templates' relative paths to the rules become `../singlefs-ai-sop/rules/…`, which
  resolves from `.claude/kb/`; the project template's shell-lint comment lists everything.
- README: "clone" → "copy"; the three-gate passage said both "enforced by gate-lint" and "judged by people" — now
  the gate covers only the form of gate 3.
- en / ja: wording inconsistencies in the 0.0.48 / 0.0.49 translations fixed along the way (English "remedy" had
  been used for both 出路 and 改法; Japanese had two spellings each for "literal pid" and "VM").
- Round three: `test-discipline.md` and `verify-before-claiming.md` gain a "Which half the gate handles" section stating what is a check and what rests on people; `command-safety.md` gains "Three silent failures at a process boundary"; `code-discipline.md` states that shell naming discipline has not been back-applied to existing scripts; `sop-first.md` drops a countable claim; in the English and Japanese repositories the "Which half the gate handles" section title had two renderings each, now unified into one.

QEMU and herd7 are handed over to singlefs entirely. Only singlefs uses them, so how to test them, how to verify them and whether to gate on them are its own decisions; this package no longer tests or verifies either:
- `scripts/lkmm.sh`, `scripts/fetch-deps.sh`, `scripts/fixtures/lkmm/` and `templates/litmus/` are deleted. The gate has no LKMM stage.
- The unimplemented list drops the two keys `QEMU 真实负载` (QEMU real workload) and `QEMU 崩溃注入` (QEMU crash injection), replaced by `最终判据` (final criterion), which names no apparatus: the acceptance criterion is set by the project,
  and the item becomes "covered by whom" only when a project stage meets it, declares `# gate-covers: 最终判据`, and ran and passed this round.
- `install.sh` no longer lays `.claude/scripts/lkmm.sh` or seeds `litmus/` at the project root. Naming retired wrappers is now generic: any file in `.claude/scripts/` that matches the wrapper template (its current form or the form from 0.0.49 and earlier) character for character
  and whose shared script is absent from this release is named, and the version stamp is held back, instead of recognising only a hard-coded `qemu.sh`. The full wrapper text is written only in `wrapper_text`, so laying a new wrapper and recognising a retired one compare against the same text.
- `env.sh` no longer checks qemu-system-x86_64, fio or `/dev/kvm`.
- `i18n-sync.sh` drops the `.litmus` form of the provenance marker, and `install.sh` no longer recognises it when stripping markers; `manifest.sh` no longer counts `.litmus` in the translated list or the coverage check;
  the remedy in `show-me-test.sh` loses its two lines on concurrency and memory ordering.
- Rules: in `show-me-test.md`, "The final criterion is QEMU/KVM stress testing" becomes "The final criterion is set by the project". `machine-first.md` premise two no longer names herd7 / LKMM,
  and states that the tool, its discriminating power and its binding to code are the project's own decisions, with the basis pointing at singlefs's kb; the two kinds of "model" in its table are written apart, and the one that gets exhausted is called the "formal model".
  `command-safety.md` drops the "QEMU virtual machines must write their pid to a file" section; in the sentence on waiting for a process to end, "things like VMs" becomes a process you did not start yourself;
  the result-collection sentence's "BIOS escape sequences, kernel logs, serial-port noise" is rewritten without naming a source; the measured record on subshell assignments no longer names a QEMU harness.
  `test-discipline.md`, `code-discipline.md` and `sop-first.md` follow.
- Skills: crash-test drops its LKMM and QEMU sections, and its `gate-covers` keys are `模型对拍` (model-based differential testing), `崩溃点重放` (crash-point replay), `最终判据` (final criterion) and `命名纪律（shell）` (naming discipline for shell); gate drops its LKMM row.
  The project template and README follow; GLOSSARY's "control case", previously defined in terms of litmus, becomes the umbrella term consistent with `test-discipline.md`: a positive control or a real baseline.
- selftest drops the lkmm fixtures and the litmus provenance markers, two groups, 20 cases in all; the 3 retired-wrapper cases are now built from lkmm.sh. Names like `stop_vm`, `HAS_KVM` and `KERNEL_PATH` in the shell-lint fixtures become neutral ones.
- The removed text as it stood is kept in singlefs under `.claude/handover/qemu-herd7/`.
- **This release does not bump the version, so the gate gives no sign that the rules changed**: singlefs's version stamp is already 0.0.50, the `规范版本` (spec version) and `副本与上游同版本` (copy and upstream at the same version) stages stay green, and syncing rests entirely on people.
  When syncing, install.sh stops at two places: the two litmus lines in `.claude/install-owned` (this package no longer lays those paths) and the `.claude/scripts/lkmm.sh` wrapper (its target is deleted);
  it stops at the first one first, and the second shows up only when you rerun after fixing it. The gate stops at one: the `# gate-covers: QEMU 真实负载` header of `gate.d/55`, a key no longer on the list.
  By what stage 55's header says, it runs a real workload plus two controls that must go red, with no crash injection, so the line should be deleted; once the acceptance criterion is written into the kb and the stage does all of it, write `最终判据`.
  Nothing shared judges `litmus/` any more; to keep judging it, wire the handed-over `lkmm.sh` in as the project's own stage. Other places that will not go red but no longer tell the truth after syncing are listed in the README of the handover directory.

Not touched: in `evidence-discipline.md`, "five such claims were found…" — the parenthesis counts only four; the
original record is in singlefs and cannot be verified here. Seven GLOSSARY terms unused anywhere in the repository,
pending verification; a batch of dates in the changelog-lint and doc-lint fixtures are earlier than the day the fixture was added (for example, a fixture added on 2026-09-10 says 2026-09-01), all within the bounds.
They are invented version sections and history entries inside fixtures, not placeholder dates like 2026-01-01; this release leaves them, and whether to change them is undecided.

Two more were deliberately left. The "Which half the gate handles" section for `evidence-discipline.md` was not added: this release takes in another session's two paragraphs in that file, and adding the section would need another round of audit. Trimming the case paragraphs and merging duplicated criteria in the rules was not done: what would be removed is evidence, and when in doubt it stays.

## 0.0.49 — 2026-09-14

**Four gate gaps found while wrapping up 0.0.48, fixed together.**

- `gate.sh --staged` leaked its temporary worktree whenever the inner gate went red: the outer script ran
  "inner; rc=$?" under the `set -e` from lib.sh, so a red inner gate made the outer one exit on that line and
  cleanup never ran — and red is exactly when you want the result. Measured on fresh repos: worktree
  registrations went 1 → 1 on a pass and 1 → 2 on a failure. The exit code is now taken inside an `if`.
- `project_root` only recognised a `.git` directory, while a git worktree's `.git` is a file; nine scripts use
  `${1:-$(project_root)}`, so run without arguments inside a worktree they walked up to `/` and exited 1 with no
  output under `set -e`. A `.git` file now counts, and when no root is found it says so and gives a way out.
- "Copy matches upstream" reported "copy behind upstream" even when the copy was newer, which pointed the wrong
  way (re-copy the copy, and get the old version back). It now splits by direction: an older upstream is reported
  as such, and the way out is to update upstream.
- `install.sh` re-seeded templates a project had deleted (`put` only refuses to overwrite existing files). A file
  listed in install-owned and deleted by the project is no longer laid down again.

selftest: 267 cases (+7: --staged cleanup on red, finding the root inside a worktree, saying so when there is no
root, the two version directions, and not re-seeding an owned-and-deleted file plus its read-back). Each of the five
fixes, reverted one at a time, turns red on the case written for it.

Two measured lessons another session had left uncommitted in the zh working tree are folded into this version as well
(they had no version bump, no manifest refresh and no translation):
- `command-safety.md`: `pgrep -f` inside a wait loop matches the loop's own command line and never exits; wait on the
  literal pid (`kill -0`) or with `wait`. The S3 check in `shell-lint.sh` extends to command position after `if` /
  `while` / `until` / `!` (`CMD_POS` in `lib.sh` is the one definition), with the fixture `pgrepwait`.
- `evidence-discipline.md`, "A criterion can be written wrong too", gains a fourth form: the remedies a pre-run clause
  offers do not reach the cells that were hit; the three questions to ask after a hit become four.

selftest gains one more case for this: 268 in total.

## 0.0.48 — 2026-09-14

**singlefs's `crates/` now has code, so the LKMM and QEMU gates change with it.**

LKMM (`scripts/lkmm.sh`):
- **Controls are recognised by content**, no longer by filename alone: with comments and the first line
  stripped, a control may only drop barrier lines from its Never, or relax `smp_store_release` /
  `smp_load_acquire` to `WRITE_ONCE` / `READ_ONCE`, at least once. By filename alone, a "control" with a
  different `exists` or reader passed; with colliding prefixes (`a` and `a-b`), `a-b`'s control was also
  counted as `a`'s.
- **Every Never must be bound to code**: its header states `singlefs-models: <path>::<function>`; the gate
  checks the file exists, the `fn` exists, and some `.rs` under `crates/` spells out the litmus filename.
  One that models no code writes `none — <reason>`. herd7 judges only the shape in the litmus, so if the code
  changed its order and the litmus did not, the verdict stayed Never. The template `commit-publish.litmus`
  is marked `none`.
- New `--static-only`: runs only the checks that need no herd7 and exits 3 even when they all pass. selftest
  feeds its fixtures through it, so their verdicts no longer depend on whether the machine has herd7.

QEMU:
- **Removed the shared `scripts/qemu/run.sh` and the gate stage "QEMU harness self-test" (`GATE_QEMU`).**
  It attached no disk, took only shell scripts and captured no results; singlefs wrote its own VM harness for
  that reason, and gate stage 55 uses that one — the shared harness's self-test proved something nobody used.
  The VM harness belongs to the project; the rules it must keep remain in `rules/command-safety.md`.
- `install.sh` no longer lays down `.claude/scripts/qemu.sh`; an untouched old wrapper left in a project is
  named, and the version stamp is not refreshed until it is removed.

`gate.sh`:
- **Projects declare coverage of the unimplemented list**: a local stage's header states
  `# gate-covers: <item>`; only when it ran and passed this round does the item become "covered by which stage".
  A key not on the list is red. The list splits "QEMU crash injection" (the final acceptance criterion) from
  "QEMU real workload". Before, the list was hard-coded: singlefs ran crash-point replay and the real-device
  stage on every gate run, and the summary still printed "nothing under test" and "crash consistency is not
  yet in the gate".
- A local stage exiting 77 is recorded as "not run this time" — not a pass, and not coverage.

Rules follow: `show-me-test.md` (the VM harness belongs to the project, coverage declarations, exit 77) and
`machine-first.md` (controls by content, litmus bound to code); the `crash-test` and `gate` skills and the
project template follow as well. The `gate` skill also gains the "Unimplemented stages" section the zh source
has carried since the first version and this translation had been missing.

## 0.0.47 — 2026-09-13

**Three-way consistency check on 0.0.46, fixing 4 translation issues and 1 drift in the zh source itself.**
Four agents each did a paragraph-by-paragraph semantic check of the zh / en / ja versions of
`evidence-discipline.md`, `show-me-test.md`, `test-discipline.md`, and `verify-before-claiming.md`
(show-me-test came back clean). Fixes: `verify-before-claiming.md`'s English had dropped a bold emphasis
("does not mean you know what it settled on"); the same file's "factual corrections from someone else"
had been narrowed to "the user" / "利用者" in both English and Japanese, inconsistent with
`pushback-discipline.md`'s rendering of the identical zh sentence ("someone else" / "他人") — restored to
match; `test-discipline.md`'s English had conflated "作废条款" (discard clause) with "失败条款" (failure
clause) at one site — restored to "discard clause"; `evidence-discipline.md`'s Japanese rendered "门禁" as
both "門番" and "ゲート" in different places — unified to "ゲート"; the English "判据" had drifted to
"Test" at one site — restored to "Criterion". Also found a drift in the zh source itself: the self-check
table had grown to 6 rows over past additions while the body text still said "这四条" (these four) —
corrected to "这六条" (these six).

## 0.0.46 — 2026-09-13

**Seven lessons measured on 2026-09-13 added to four rules; the gate is unchanged.** `evidence-discipline.md`: the
self-check table under "Never pick the conclusion first and then build a model for it" gains a row (is this number a
function of some parameter that was never swept); "An arm's definition is nailed down before the run too" gains a
paragraph — revising an arm or a criterion before the artifact runs is legitimate, but the pre-run registration must state
what changed, which unit-test reading it rests on, and that the moment was before the artifact; a new section "A criterion
can be written wrong too: when it fires, first decide which kind it is" lists three forms (the hit does not tell the arms
apart, the discriminator cannot be observed, the hit is filed under the wrong criterion) and the three questions to ask
first. `test-discipline.md`: a new subsection under the failure-clause section, "Do not write a criterion as a
conjunction; a threshold must not be a tautology of the arm's definition", and a new section "An endpoint is not a
trajectory: a quantity a clause feeds into a predicate must be reported as a trajectory". `show-me-test.md`: a
cross-apparatus check may pin only the value, not the quantity — two quantities get two registered names, each pinned to
its own value. `verify-before-claiming.md`: truncated output is a narrow claim too; before saying "only these places",
do not truncate — count first.

## 0.0.45 — 2026-09-11

**`gate.sh --staged` cleans up its temporary worktree when interrupted.** A Ctrl-C used to leave the worktree registered
in the repository, needing a manual `git worktree prune`; the worktree is now removed by an INT / TERM trap before exiting.
selftest gains a case: a project stage that sleeps 20 seconds gets INT sent to its whole group mid-run, and afterwards the
repository must hold exactly one worktree registration; with the trap removed the case goes red.

## 0.0.44 — 2026-09-11

**`gate.sh --staged`: run the whole gate on HEAD plus the index only.** When several sessions share a repository,
the working tree mixes in other sessions' unfinished changes and untracked files, and a red gate on the working tree
cannot say whose it is — `session-wrapup.md` item 4 could only ask you to check file by file. The gate now applies
`git diff --cached` onto HEAD in a temporary worktree and runs the whole gate there: other sessions' uncommitted changes
and untracked files stay out, and whatever goes red is what this commit brings in. An untracked SOP copy is copied into
the worktree as-is, and sibling upstream repositories are still looked up from the original project root. It came from a
singlefs wrap-up on 2026-09-11: the working-tree gate went red in five stages, three of them from another session, and
only a hand-built worktree told them apart. selftest gains three cases: an unstaged violation does not count under
`--staged`; without `--staged` the same violation must count (proving the stage would go red — without this case a green
`--staged` cannot tell "left out" from "never red"); a staged violation counts. `session-wrapup.md` item 4, second
bullet, gains a sentence on how to use it.

## 0.0.43 — 2026-09-11

**`i18n-sync.sh --update` refreshes each translation repo's `SOURCE-MANIFEST.sha256` itself; nobody copies it by
hand any more.** When 0.0.42 went out, `--update` was rejected as "behind" because en / ja still held the old
manifest, so none of the shared files were synced — that step used to rely on someone remembering to copy zh's
`MANIFEST.sha256` across. Now, when `--update` finds SOURCE-MANIFEST behind or missing, it first checks every
translation's provenance stamp: if all of them match the current source it refreshes SOURCE-MANIFEST from this
repository's manifest and reads it back; if any file is missing, unstamped or translated from an older source, it
does not copy and names those files. The per-file check is now a single function, `stale_articles`, shared by this
step and the existing check 5. The two howto lines that told people to copy by hand now say to run `--update`, and
the three READMEs say the same. `selftest.sh` gains 5 cases: after retranslating and stamping it copies and the copy
matches the manifest byte for byte; without retranslation it does not copy and SOURCE-MANIFEST is untouched; a
missing one gets created.

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
