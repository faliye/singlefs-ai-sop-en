# CHANGELOG

Version history for the rules and the gate. `CLAUDE.md` and `rules/*.md` keep no
history sections (design-doc-discipline); history lives here. For per-change
detail see `git log` — commit messages are the change notes.

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
  manifest / i18n-sync — 51 cases.

## 0.0.22 — 2026-09-02

- session-wrapup gains "are other sessions flying in this repo"; show-me-test
  gains the mutation list; test-discipline gains "deterministic model × N runs"
  and "mutation testing proves assertions can go red"; GLOSSARY gains
  mutation-testing terms.

## 0.0.21 and earlier

See `git log --oneline` — each commit message is written as "version: what changed".
