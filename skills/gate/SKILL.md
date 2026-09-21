---
name: gate
description: Run singlefs's acceptance gate. Use it before submitting code, or when judging whether a change can be accepted — covers what each stage means, how to read the result, and which "failures" are environment problems rather than code problems.
---
<!-- generated-from: skills/gate/SKILL.md sha256:e71deec8f716d300a5c2209711931977bb2540d363c7d9f439d035307ba07e0e -->

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

| Stage | A failure means |
|---|---|
| Spec version | The project's `.singlefs-ai-sop-version` and singlefs-ai-sop's `VERSION` disagree. **Read the rule changes first**, then run `install.sh` to refresh the stamp |
| Gate self-check | Some rejection gives no way out (a `bad` with no `howto`, a `die` carrying one argument, a printed `✗` with no `→` after it), or a check that scans a set of objects reported no count on success. Shapes in `rules/sop-first.md` |
| Gate discriminating power | A fixture was judged against expectation — **the gate itself is broken**; fix that before anything else |
| Shell discipline | A script has `pkill -f` / `killall` / `pgrep -f`, carries a value out of a subshell through a variable, uses git's undo commands, or runs `rm -rf` on an unguarded variable path. See `rules/command-safety.md` |
| Document discipline | History statements in body text, a kb number cited without its short name, or a CLAUDE.md that does not `@` every rule. See `rules/writing-discipline.md` |
| Naming discipline | A name we declare in a `.rs` file is a single letter or a common abbreviation, or `.claude/abbreviations` / `.claude/naming-lint-exclude` is malformed. See `rules/code-discipline.md` |
| Show me test | `crates/*/src` changed with no test. **This one is not to be bypassed**; see `rules/show-me-test.md` |
| Build and unit tests | Genuinely broken, or cargo is missing. clippy runs with `-D warnings`, and a `_ =>` arm on an enum that is a closed set is refused as well |
| Project-local stages | Some local check in `.claude/gate.d/` failed, or could not be read; a red "覆盖声明（…）" (coverage declaration) means a `# gate-covers:` line names an item that is not on the list |
| Working tree unchanged during the run | Files in the working tree changed while the gate ran (you were still editing, or another session was), so the stages did not all read the same version. Wait until the edits stop and rerun, or use `--staged` |

**Three stages run only in the SOP repository itself** (invisible to consuming projects):
cross-language sync, version discipline, and CHANGELOG continuity.

"Rule manifest" runs on both sides but asks different questions: inside the SOP repo it
asks whether the manifest is in step with the rules; inside a project it compares
**the copy you installed** — a modified or partial copy turns this red.
When the installed copy is the en / ja edition, this stage reports "not applicable": the manifest is
maintained only in the reference repository (zh), and whether a translated edition has kept up is judged
by the zh repository's cross-language sync stage.

## Unimplemented stages

Every run, `gate.sh` lists the verification methods the shared gate **does not implement**:
model-based differential testing, crash-point replay, the final criterion, and naming discipline for
shell scripts. Only the project can wire the first three in, under `.claude/gate.d/`: model-based differential
testing needs an ideal model of the thing under test, crash-point replay needs its own write stream and checker,
and the final criterion is set by the project.
A stage that does declares `# gate-covers: <item>` in its header (keys copied literally from
`gate.sh`); only when it ran and passed this round does the item move under "covered by
project-local stages".

**This is not noise; it is a precondition for reading the result**: an all-green gate says only
"documents comply + tests exist + unit tests pass", plus whatever the stages listed under
"covered by project-local stages" each verified. While no stage covers crash-point replay, any
claim that "the write path is verified" is false; when a stage does cover it, the claim reaches
only as far as the write paths that stage enumerated.

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

After changing `gate.sh` or `doc-lint.sh`, **build an input that ought to be stopped and
confirm it really goes red**:

```bash
# Build a sample that should be rejected, feed it to doc-lint, confirm it goes red
d=$(mktemp -d); mkdir -p "$d/kb"
printf '# Decisions\n\nNode size is 16K (was 4K).\n\n## Revision history\n\n### %s\n- Created.\n' "$(date +%F)" \
  > "$d/kb/decisions.md"
bash .claude/singlefs-ai-sop/scripts/doc-lint.sh "$d"; echo "exit code $? — expected 1"
rm -rf "$d"
```

⚠️ **Build the sample in a separate directory; do not `>>` onto a real kb file.**
Appended text lands after the "Revision history" heading, where the body scan has
already stopped — the exit code is 0, which looks like "the check does nothing" when in
fact the sample was built in the wrong place (measured on this very skill's own example
during an audit).

After changing a check, also run
`bash .claude/singlefs-ai-sop/scripts/selftest.sh`: it uses the fixtures under
`scripts/fixtures/` to prove every check can still go red.
**A new check comes with a new fixture**, and its `want=` must name that check's own
message — a fragment shared by several checks watches nothing
(`rules/show-me-test.md`).

Per `rules/show-me-test.md`, a check that cannot be shown to go red is not written.
