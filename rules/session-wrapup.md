<!-- generated-from: rules/session-wrapup.md sha256:dcb3a51a09042ac52fc246c0ac38af5db39504caae67244e606ae1b450ef1628 -->
<!-- doc-lint:rule-definition -->
# Wrap-up: required before the end of every round of work

## 0. Report progress — first, and never skipped

One line, three numbers: **which milestone we are at / whether the gate passes / what
evidence is still missing.** Status is always checked now (run `gate.sh`), never
copied from the previous round's notes.

**State at the same time how much this round added to that number.** Often it is 0: a
null result, a decision overturned, or merely establishing that some piece of evidence
is not yet obtainable. **Then write 0.** Counting other rounds' results into this one
turns the progress table into something that only ever goes up, and it stops being
useful.

## 1. Any script or command typed this round that should move into `scripts/`?

The criterion is "**will it get copied a second time**", not "is it well written".

**Turn traps you have hit into checks that fail, not into reminder sentences.**
Writing "careful not to do Y while X" stops nobody typing commands by hand; a check
that **refuses to run** at that moment does.

Look the other way too: is anything in there no longer used? Delete it.

## 2. Any skill or rule used this round that should be changed?

Was any step held together by memory? Was a criterion added only after the fact? Was
the same boilerplate copied a third time?
**If so, change that file now**, not "next time".

Where to change it — look at what the item governs:

- **Collaboration** (evidence, documents, where decisions land, gate feedback) →
  change singlefs-ai-sop and bump `VERSION` (which paths require the bump is defined
  by `GOVERNED` in `scripts/version-discipline.sh`)
- **How the filesystem is designed** → change the project's own `kb/`,
  `.claude/rules/`, or the project's `CLAUDE.md`

**When in doubt, keep it in the project.** singlefs is this SOP's only user, so
"would another project need it" is not a criterion — there is no other project to
look at, and anyone can answer "yes".

**Do not write it into a session's private memory.** A session's own memory (for example Claude Code's memory) lives only on this machine and is seen only by sessions on it:
other contributors cannot see it, and subagents that do not inherit the project instructions cannot read it either. So pitfalls and conventions that someone else could run into
go into the project or into this SOP; private memory holds only the user's personal preferences (when to commit, which language to reply in, and the like).
Measured (2026-09-17, singlefs): one inventory of the private memory found 10 of 18 entries were project conventions or progress, and two of them contradicted the rule text in the project,
so other sessions kept reading the old wording in the rules.

## 3. Did any decision change?

If this round overturned or settled any design decision, **write it into
`kb/decisions.md` right away**, with what overturned it.
One missing decision record means that in three months "why did we decide this?" needs
archaeology to answer.

**A verdict reached by argument is also a decision change.** When weighing something up
leads you to "this one should not be settled yet" or "keep the current approach", and
that conclusion in substance overturns a sentence already written in `kb/decisions.md`,
that is an overturn: record it on the spot, with the grounds.
A verdict that lives only in the conversation while the decision file stays untouched
leaves nobody, three months later, knowing why that sentence no longer holds.

## 4. Is another session in flight in the same repository?

When several sessions work concurrently, four default assumptions stop holding. Go
through them before wrapping up:

- **"Everything in the working tree is mine" no longer holds.** Before committing, sort
  the changes into "this round" and "not this round" and commit only your own. Sweeping
  another session's work-in-progress into your commit means publishing, on their behalf,
  something they had not finished verifying.
- **"The gate went red = I broke something" no longer holds.** On a red, first check
  whether the files it names are part of this round's changes. If they are not, report
  honestly "red, but not from this round" and do not fix it in passing — that is another
  session's wrap-up, still unfinished.
  When you cannot tell, run `bash .claude/scripts/gate.sh --staged`: it runs the whole gate
  in a temporary worktree on HEAD plus the index only, so other sessions' unstaged changes
  and untracked files stay out — whatever goes red there is what this commit brings in.
  Conversely, when you run without `--staged` and the working tree changes while the gate runs (you are still editing,
  or another session is), the stages do not all read the same version and the red/green summary corresponds to no version
  at all; `gate.sh` fingerprints the working tree at the start and at the end and goes red when they differ.
  **Wait until the edits stop, or use `--staged`.**
- **Shared numbering is first-come, first-served.** For history entry ordinals,
  experiment numbers and the like, look up the highest existing number before taking
  one. Edit shared files by targeted replacement only, never by rewriting the whole
  file — a rewrite silently erases what a concurrent session has already written.
- **"Creating a new file overwrites no one" no longer holds.** Another session may have taken the same number
  minutes ago and written an uncommitted file with the same name, and a tool that writes whole files overwrites it
  silently — something that never went into git is gone once overwritten. Create new files exclusively
  (`set -o noclobber`, `open(path, 'x')`), and look up the number in the same command that writes the first file;
  where the tool layer can stop it (a pre-write hook that refuses to overwrite untracked files), stop it there.
  Measured (2026-09-12): one whole-file write overwrote another session's freshly written pre-run registration, which
  was restored word for word only by replaying that session's conversation transcript.
- **"Commit only these paths" is not `git commit -- <path>`.** A commit given paths commits what those paths hold in the **working tree**, not in the index — when the same file also carries another session's uncommitted edits, they ride along.
  And the index itself is shared: another session can put its files into it at any moment. So before staging, confirm `git diff --cached --name-only` is empty; after staging and before committing, check the list and each file's cached diff once more, then `git commit` with no paths.
  Measured (2026-09-12), the second kind: only explicit paths were `git add`ed and the commit was made without paths, yet a version-stamp file another session had staged in the meantime went into the commit.

If you collide on a number and it can be made into a check that goes red, make it one
(`rules/show-me-test.md`, "turn traps you have hit into checks that fail"); on the
project side, just follow the numbering shape the history files already use.
