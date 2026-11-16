---
name: gate
description: Run this project's acceptance gate. Use it before submitting code, or when judging whether a change can be accepted — covers what each stage means, how to read the result, and which "failures" are environment problems rather than code problems.
---
<!-- generated-from: skills/gate/SKILL.md sha256:cf7f1671031b37c6a6c77c274963db006aeaf153bfeda19013fc893525a39131 -->

# The acceptance gate

The rule lives in `rules/show-me-test.md`. **This page is about running it, reading it,
and which failures are not real.**

## Running

```bash
bash .claude/scripts/gate.sh          # everything; mandatory before submitting
bash .claude/scripts/check.sh         # only format/lint/build/unit tests, for fast feedback
bash .claude/scripts/env.sh           # only the environment check
GATE_BASE=<commit> bash .claude/scripts/gate.sh   # pick the diff base
bash .claude/scripts/gate.sh --staged # only HEAD + the index: for several sessions writing one repository; what goes red is what this commit brings in
```

## Stages and how to read them

When a stage goes red it prints what blocked and what to do next; do that. What a red in each stage means, and where the rule is:

| Stage | A failure means | Rule |
|---|---|---|
| Spec version | The project's `.singlefs-ai-sop-version` and singlefs-ai-sop's `VERSION` disagree. **Read the rule changes first**, then run `install.sh` to refresh the stamp | |
| Copy matches upstream version (`副本与上游同版本`) | What ran is not the project's own copy (run `bash .claude/scripts/gate.sh` instead), or the copy's `VERSION` differs from the upstream repository in the sibling directory: if the copy is behind, copy it fresh from upstream and run `install.sh`; if upstream is older than the copy, update or commit that version upstream first | |
| Gate self-check | Some rejection gives no way out, or a check that scans a set of objects reported no count on success | `rules/sop-first.md` "Every rejection must carry a next step", `rules/show-me-test.md` "The gate must not pretend to pass" |
| Admission and run conditions (`准入与运行条件`) | A script's `admission:` / `run-condition:` declaration, `preflight` call or `preflight_record_success` does not comply, or an exclusion or registration table points at a path that does not exist. Register libraries and fixtures in `.claude/preflight-exclude` | `rules/preflight-discipline.md` |
| Gate discriminating power | A fixture was judged against expectation — **the gate itself is broken**; fix that before anything else | |
| Shell discipline | A script has a form that S1 to S10 of `shell-lint.sh` forbid | `rules/command-safety.md` |
| Script executable bits (`脚本执行位`) | A `.sh` in the index is not `100755`, or a mode in the index disagrees with the executable bit in the working tree: fix it with `git update-index --chmod=+x <path>` (or `-x`). An empty directory in scope is red as well: put a `.keep` in any you need to keep and `git add` it | |
| Local stage discriminating power (`本地阶段判别力`) | A fixture under `.claude/gate.d/fixtures/<stage>/` was judged wrong: the exit code is off, or the `want=` line is missing from the output. Decide whether the stage or the fixture is wrong, then fix that side; a stage with no fixtures is recorded as not run this time | |
| Link targets (`链接指向`) | A relative link in a document points at a path that does not exist (relative paths resolve from the linking file's directory), or a "section <number>" reference is past the target document's count of `##` sections, or the target cannot be read. Point at a section title instead of "section <number>". Directories of evidence kept verbatim go into `.claude/doc-lint-exclude` | |
| History entry numbers (`历史条目编号`) | A history entry added this time reuses an ordinal "(其 <ordinal>)" under the same "`##` section + date + named item". Give it the next free number; do not renumber an entry someone else committed | `rules/kb-discipline.md` item 8 "Body text states only the current state; history goes to a "Revision history" at the end" |
| Tool-layer gates (`工具层的闸`) | A hook in `.claude/hooks/` or in the copy's `scripts/claude-hooks/` is not registered in `.claude/settings.json`, fails its `--selftest`, is not attached to every event in its `hook-events`, or, written with a tool name, sits on a matcher that does not recognise that tool. Register it as its file header says | `rules/sop-first.md` "Before adding a gate or hook, look for an existing one" |
| New root entries (`项目根的新条目`) | A file or directory newly added at the project root is not registered in `.claude/root-entries`, or the registry is malformed. Move it into an existing directory first; if it really must sit at the root, ask the user, and register it once they agree | `rules/command-safety.md` "Do not create files or directories at the project root" |
| Soft links (`软链接`) | The repository has a symbolic link not registered in `.claude/soft-link-exempt` at the project root, or the exempt table is malformed. Replace it with a copy or a written-out real path and delete the link; if it really has to stay, register it | `rules/command-safety.md` "No symbolic links in the repository" |
| Gate overlap check (`门禁查重`) | A gate or hook added or changed this time lacks `gate-similar` / `hook-events`, does not name every existing gate or hook it must, gives too short a reason, or adds lines identical to a whole block of an existing one. First run `python3 .claude/singlefs-ai-sop/scripts/gate-overlap.py --list` to find the one that handles the same thing, and merge where you can | `rules/sop-first.md` "Before adding a gate or hook, look for an existing one" |
| Relay timing (`转发计时`) | In `.rs` and `.py` files, a loop reading a child process's output both timestamps lines and relays them. Fix it per the rule first; where the timestamp genuinely feeds no timing conclusion, write `// relay-timing-lint:allow <reason>` on the loop head (`#` in Python) | `rules/show-me-test.md` "Time phases inside the process under test, not in the loop that relays its output" |
| Duplicate definitions (`同名定义`) | In a `.py` file, one scope, or in a `.sh` file, the column-0 definitions of one file, define the same name twice. Rename or delete one; where overriding is intended, write `# duplicate-definitions:allow <reason>` on the later definition's line | `rules/code-discipline.md` "One definition per name per scope" |
| Code names (`代号`) | A `.md` line added this time refers to something by a code name (`5a`, `10g`, `SC02` and the like). Write what it is; put commands and file names in backticks; a project's own registered numbering systems go into `.claude/code-names` | `rules/writing-discipline.md` "No abbreviations, nothing referred to by a code name" |
| Numbers and short names (`编号与简称`) | A "number (short name)" cited in a source comment, script or record disagrees with the short name at the kb registration. Copy the short name from the registration (the first line of each entry, `## D<ordinal> short name —— status`) | `rules/kb-discipline.md` item 5 "A number can serve as an index, never as a name" |
| Document discipline | History statements in body text, a kb number cited without its short name, or a CLAUDE.md that does not `@` every rule | `rules/design-doc-discipline.md` item 1, `rules/kb-discipline.md` item 5, `rules/writing-discipline.md` |
| Document discipline's unimplemented list (`文档铁律的未实现清单`) | `doc-lint.sh --not-impl` failed, so the unimplemented list at the end of the summary is missing its entries. Run `bash .claude/singlefs-ai-sop/scripts/doc-lint.sh --not-impl` alone to see why (exit 78 means its admission and run conditions are unmet) | |
| Rule discipline (project-local) (`规则纪律（项目本地）`) | The project's `.claude/rules/` (or `.claude/agents/` when that directory does not exist), the project `CLAUDE.md`, an agent definition or a skill body does not comply. Register files not yet swept one by one in `.claude/rules-lint-exclude` | `rules/rules-discipline.md` |
| Naming discipline | A name we declare in a `.rs` file is a single letter or a common abbreviation, or `.claude/abbreviations` / `.claude/naming-lint-exclude` is malformed | `rules/code-discipline.md` |
| Show me test | `crates/*/src` or `crates/*/build.rs` changed with no test. **This one is not to be bypassed** | `rules/show-me-test.md` |
| Build and unit tests | Genuinely broken, or cargo is missing. clippy runs with `-D warnings`, and a `_ =>` arm on an enum that is a closed set is refused as well | |
| Rule manifest (`规则清单`) | In a project: the installed copy was modified or copied incompletely; copy it fresh from upstream and run `install.sh`. In the SOP repository: the manifest is out of step with the rules, or some human-facing text is neither in the manifest nor exempted; fix it, then run `bash scripts/manifest.sh --update`. When the installed copy is the English or Japanese edition, this item exits 77 and is recorded as not run this time (`本次未跑`): the manifest is maintained only in the reference repository (the Chinese one), and whether a translated edition has kept up is judged by the Chinese repository's "Cross-language sync" (`各语言同步`) | |
| Project-registered unimplemented methods (`项目登记的未实现手段`) | A line in `.claude/gate-not-implemented.tsv` lacks its key or description, or its key repeats a shared key or another line. Write one line as `key<TAB>what is missing<TAB>reminder that still applies once covered`, each key registered once | |
| Project-local stages | Some local check in `.claude/gate.d/` failed, or could not be read; a red "覆盖声明（…）" (coverage declaration) means a `# gate-covers:` line names an item that is not on the list | |
| Working tree unchanged during the run | Files in the working tree changed while the gate ran, so the stages did not all read the same version. Wait until the edits stop and rerun, or use `--staged` | |
| No temporary files left behind | Some stage (or a test or harness it started) created something in this run's `TMPDIR` and did not delete it when it finished. Have whoever creates it delete it when it finishes. When another stage went red this item is recorded as not judged this run, and the temporary directory is kept whole for you to look at the scene (only the latest 3 are kept) | `rules/command-safety.md` "Test images always go in a temporary directory" |

**Four stages run only in the SOP repository itself** (invisible to consuming projects):

| Stage | A failure means |
|---|---|
| Rule discipline (`规则纪律`) | This package's `rules/`, `CLAUDE.md`, `agents/*.md` or `skills/*/SKILL.md` breaks one of the parts of `rules/rules-discipline.md` that can be judged literally |
| Cross-language sync (`各语言同步`) | A translated repository cannot be found, its shared part or manifest has not kept up, or a translation's provenance hash on its first line is not the current source. For the shared part run `bash scripts/i18n-sync.sh --update`; after retranslating against the current source run `bash scripts/i18n-sync.sh --stamp <language> <files>…` |
| Version discipline (`版本纪律`) | A path governed by `GOVERNED` in `scripts/version-discipline.sh` changed without a `VERSION` bump, or `VERSION` went down |
| CHANGELOG continuity (`CHANGELOG 连续`) | The newest section of `CHANGELOG.md` is not `VERSION`, two adjacent sections skip, repeat or go backwards, or a level-two heading is not a version section. Give every version its section |

## Unimplemented stages

Every run, `gate.sh` lists the verification methods the shared gate **does not implement**: the final criterion, naming discipline for shell scripts, and whatever the project registered in `.claude/gate-not-implemented.tsv` at its root.
Only the project can wire the final criterion and its own registered items in, under `.claude/gate.d/`: a stage that does declares `# gate-covers: <item>` in its header (keys copied literally from `gate.sh`); only when it ran and passed this round does the item move under "covered by project-local stages".

**Read the result together with this list**: an all-green gate says only "documents comply + tests exist + unit tests pass", plus whatever the stages listed under "covered by project-local stages" each verified.

- While no stage covers a registered item, do not claim "that one is verified".
- When a stage does cover it, the claim reaches only as far as that stage itself verified (the registry's "reminder that still applies once covered" column, which `gate.sh` prints at the end).

## Common false failures

| Symptom | Real cause |
|---|---|
| Show me test says "nothing to judge" | The working tree matches the base. **Neither a pass nor a failure**; make a change and rerun, or set `GATE_BASE=<ref>` |
| "Upstream freshness not checked" | The upstream repository is not a sibling directory. This item **could not run** and is listed separately in the summary — do not read it as a pass |
| You wired in a stage, yet the unimplemented list still shows the item | That stage's header lacks `# gate-covers:`, or this round it exited 77 or went red. Only a stage that ran and passed moves the item |
| Build stage reports cargo missing | Environment problem. Run `env.sh` for the full picture, install the toolchain, rerun |
| doc-lint flags an example quoted in a rule document | That file is missing `<!-- doc-lint:rule-definition -->`, or the example is not in backticks / 「」. With the marker the body is still checked; only examples in those two forms are exempt |
| Show me test says no tests, but you wrote some | The tests sit in `crates/*/src/` without `#[cfg(test)]`/`#[test]`, so the script cannot see them |

## The gate must be able to fail, too

After changing a check in `scripts/`, **build an input that ought to be stopped and
confirm it really goes red**:

```bash
# Build a sample that should be rejected, feed it to doc-lint, confirm it goes red: a kb document missing its closing history section (a structural check, judged in a repository of any language)
sample_directory=$(mktemp -d); mkdir -p "$sample_directory/kb"
printf '# Decisions\n\nNode size is 16K.\n' > "$sample_directory/kb/decisions.md"
bash .claude/singlefs-ai-sop/scripts/doc-lint.sh "$sample_directory"; echo "exit code $? — expected 1"
rm -rf "$sample_directory"
```

- **Build the sample in a separate directory**; do not `>>` onto a real kb file.
- After changing a check, also run `bash .claude/singlefs-ai-sop/scripts/selftest.sh`: it uses the fixtures under `scripts/fixtures/` to prove every check can still go red.
- **A new check comes with a new fixture**, and its `want=` must name that check's own message, not a fragment shared by several checks.
