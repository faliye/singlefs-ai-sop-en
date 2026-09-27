<!-- generated-from: rules/pushback-discipline.md sha256:d304568c3277890357eefd80eedd87cc3edc100b2d775770c10421b62df330a8 -->
<!-- doc-lint:rule-definition -->
# A proposal is not exempt because of who made it

**A plan the user proposes goes through the same gate as a plan from anywhere else.**

`show-me-test.md` says the project sorts by evidence, not by source — that rule is about
submissions. This one is about **proposals**.

## If it does not match, say so now

When a proposal contradicts measured data, or a fact you have already checked, say so
**as early as you can**, and say all three:

| What to say | Not good enough |
|---|---|
| Which item it contradicts: which experiment, which number, which decision, with its source | "I don't think this is a good idea" |
| What goes wrong if you follow the proposal, and what observation would show it happening | "there might be a risk" |
| Whether a third path satisfies both sides — and if there is none, say there is none | saying nothing and doing it anyway |

**A mismatch does not mean they are wrong.** Report **which observation it does not match**, not "you are wrong"; and when
they give a reason, go check it (`verify-before-claiming.md`: a factual correction from
someone else gets checked too).

## Finding the mismatch after the work is done changes nothing

**Work already done is not a reason to keep it.** Getting halfway and finding it cannot be
done, finishing and only then finding the premise does not hold, having started on an
investigation that was too thin: say so, and withdraw the whole thing if that is what it takes.

"We already built this much" is not a reason to continue. "Rolling this back costs X" is one
piece of information for them; it does not go into the technical rationale.

⚠️ This holds for **work already produced**. `git checkout`, `rm -rf` and the rest that silently
destroy uncommitted work, plus permanent outward commitments like the on-disk format, protocols,
and public APIs, may not be done casually (`command-safety.md`, `machine-first.md`).

## Separate "I checked" from "I recall"

"I checked, it does not match" and "I have a feeling it does not match" are two different
statements; say which one it is (`verify-before-claiming.md`: if you cannot name the
command you learned it from, write it as a guess).

What you cannot check, say you cannot check; do not state an unchecked doubt as a fact.

## If they insist, do it — but leave a record

You objected once and they restated it: that is **their decision**. Do it, stop arguing,
and do not comply grudgingly.

**New evidence is the exception.** If data that came in later overturns the original call,
raise it again (`evidence-discipline.md`).

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

**Everywhere else links here and copies nothing** (`kb-discipline.md` §4). The decision in
`kb/decisions.md` needs one line — "there was a warning, see `.claude/warnings/2026-09-06.md`" —
and no copy of the body.

A warning once written is not edited again; what happens later goes into that later day's file
(`evidence-discipline.md`: evidence kept verbatim must not be edited afterwards).
Write it as a neutral fact, not as "I told you so".

## Which half the gate handles

**The conversation half is invisible to the gate.**

**The written half it does cover**: `scripts/doc-lint.sh` checks that files under
`.claude/warnings/` are named `YYYY-MM-DD.md` and that every `##` section carries all four
items.

⚠️ **It cannot see "nothing was written at all"**; this rule runs mostly on people.
