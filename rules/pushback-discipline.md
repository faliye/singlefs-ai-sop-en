<!-- generated-from: rules/pushback-discipline.md sha256:9a17b7700cbb21f8fca1547b92c6570b696711f4561542f518561997e9e9acf7 -->
<!-- doc-lint:rule-definition -->
# Warning records

**Covers**: the record left after an objection was raised to a proposal and the other side restated it.

## Where the warning goes: `.claude/warnings/<date>.md`

**Every project keeps `.claude/warnings/`, one file per date**, `YYYY-MM-DD.md`, one `##`
section per warning, all four items present:

```markdown
# 2026-09-06

## Collapse the index into one tree, keep no second copy

**Proposal**: <what they want done>
**Objection**: <which item it contradicts: which experiment, which number, which decision, with its source>
**Known risk**: <what goes wrong, and what observation would show it happening>
**Outcome**: <they restated it, done as asked / plan changed / withdrawn>
```

**Everywhere else links here and copies nothing** (`kb-discipline.md` item 4 "A contradiction is worse than a gap"). The decision in
`kb/decisions.md` needs one line — "there was a warning, see `.claude/warnings/2026-09-06.md`" —
and no copy of the body.

A warning once written is not edited again; what happens later goes into that later day's file.
Write it as a neutral fact, not as "I told you so".

## Which half the gate handles

`scripts/doc-lint.sh` checks that files under
`.claude/warnings/` are named `YYYY-MM-DD.md` and that every `##` section carries all four
items.

It cannot see "nothing was written at all".
