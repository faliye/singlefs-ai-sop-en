---
name: gate
description: Run singlefs's acceptance gate. Use it before submitting code, or when judging whether a change can be accepted — covers what each stage means, how to read the result, and which "failures" are environment problems rather than code problems.
---
<!-- generated-from: skills/gate/SKILL.md sha256:ddb124d130221b00203a98a87694cc171b7a11b8bc2293ed1afc007bc8e579f7 -->

# The acceptance gate

The rule lives in `rules/show-me-test.md`. **This page is about running it, reading it,
and which failures are not real.**

## Running

```bash
bash .claude/scripts/gate.sh          # everything; mandatory before submitting
bash .claude/scripts/check.sh         # only format/lint/build/unit tests, for fast feedback
bash .claude/scripts/env.sh           # only the environment check
GATE_BASE=<commit> bash .claude/scripts/gate.sh   # pick the diff base
```

## Stages and how to read them

| Stage | A failure means |
|---|---|
| Spec version | The project's `.singlefs-ai-sop-version` and singlefs-ai-sop's `VERSION` disagree. **Read the rule changes first**, then run `install.sh` to refresh the stamp |
| Gate self-check | Some rejection gives no way out (a `bad` with no `howto`, or a `die` carrying one argument) |
| Gate discriminating power | A fixture was judged against expectation — **the gate itself is broken**; fix that before anything else |
| Shell discipline | A script kills processes by pattern match, or carries a value out of a subshell through a variable |
| Document discipline | History statements in body text, or a kb number cited without its short name. See `rules/doc-discipline.md` |
| Show me test | `crates/*/src` changed with no test. **This one is not to be bypassed**; see `rules/show-me-test.md` |
| Build and unit tests | Genuinely broken, or cargo is missing |
| Project-local stages | Some local check in `.claude/gate.d/` failed, or could not be read |
| LKMM | A litmus verdict disagrees with its declaration, or a Never has no paired control |

**Two stages run only in the SOP repository itself** (invisible to consuming projects):
cross-language sync, and version discipline.

"Rule manifest" runs on both sides but asks different questions: inside the SOP repo it
asks whether the manifest is in step with the rules; inside a project it compares
**the copy you installed** — a modified or partial copy turns this red.

## Common false failures

| Symptom | Real cause |
|---|---|
| Show me test says "nothing to judge" | The working tree matches the base. **Neither a pass nor a failure**; make a change and rerun, or set `GATE_BASE=<ref>` |
| "Upstream freshness not checked" | The upstream repository is not a sibling directory. This item **could not run** and is listed separately in the summary — do not read it as a pass |
| Build stage reports cargo missing | Environment problem. Run `env.sh` for the full picture, install the toolchain, rerun |
| doc-lint flags a rule document itself | That file is missing `<!-- doc-lint:rule-definition -->` |
| Show me test says no tests, but you wrote some | The tests sit in `crates/*/src/` without `#[cfg(test)]`/`#[test]`, so the script cannot see them |

## The gate must be able to fail, too

After changing `gate.sh` or `doc-lint.sh`, **build an input that ought to be stopped and
confirm it really goes red**:

```bash
# Build a sample that should be rejected, feed it to doc-lint, confirm it goes red
d=$(mktemp -d); mkdir -p "$d/kb"
printf '# Decisions\n\nNode size is 16K (was 4K).\n\n## Revision history\n\n### 2026-01-01\n- Created.\n' \
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
