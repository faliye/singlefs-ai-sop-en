<!-- generated-from: rules/session-wrapup.md sha256:047267c7bcf442c355227adddc8f7f6914bb22825644c22c92fec20248c50a36 -->
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

**When in doubt, keep it in the project.** "Would another project need it" is not a
criterion — anyone can answer "yes", and then everything gets pushed upstream.

**Do not write it into a session's private memory.** A session's own memory (for example Claude Code's memory) lives only on this machine and is seen only by sessions on it:
other contributors cannot see it, and subagents that do not inherit the project instructions cannot read it either. So pitfalls and conventions that someone else could run into
go into the project or into this SOP; private memory holds only the user's personal preferences (when to commit, which language to reply in, and the like).

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
- **"Commit only these paths" is not `git commit -- <path>`.** A commit given paths commits what those paths hold in the **working tree**, not in the index — when the same file also carries another session's uncommitted edits, they ride along.
  And the index itself is shared: another session can put its files into it at any moment. So before staging, confirm `git diff --cached --name-only` is empty; after staging and before committing, check the list and each file's cached diff once more, then `git commit` with no paths.

If you collide on a number and it can be made into a check that goes red, make it one
(`rules/show-me-test.md`, "turn traps you have hit into checks that fail"); on the
project side, just follow the numbering shape the history files already use.

## 5. Before a subagent hands back, it deletes the build directories and repository copies it created

The repository copies a subagent made in its scratch directory, and the build directories it compiled into with its own `CARGO_TARGET_DIR`, are deleted by the subagent itself before it hands back:

- **Delete**: the build directories this subagent created this time (`target`, the directory `CARGO_TARGET_DIR` points at, and any other cache carrying `CACHEDIR.TAG`) and its repository copies (clones, copied repositories, git worktrees).
  Remove a worktree from its source repository with `git worktree remove --force <path>`, so the registration there goes too; `rm -rf` the rest.
- **Do not delete**: reports, artifacts that are to be committed, replay material the main agent still has to check; anything a predecessor, the main agent or another agent created, even in the same scratch directory;
  anything inside the project root (the project's own `target`, worktrees Claude Code created inside the project) — those belong to the main agent.
- Before deleting, record the size with `du -sh`; the handback report carries one line saying what was deleted and how big each was.
- For anything not deleted, the handback report carries one line per path: "Not deleted <full path>: <why>". Merely citing the path as a source ("ran on the copy at …") does not count.

The hook `scripts/claude-hooks/handback-scratch-check.sh` checks this, attached in two places:

- PreToolUse of the handback tool (`SubagentHandback`): stops the handback before it happens. If a build directory, worktree or repository copy that sits in a temporary directory, was created while one of this subagent's own tool calls had not yet finished,
  and was mentioned in its calls, is still there, and the report being handed back has no line for it, the handback is refused and the list is given to the subagent.
- SubagentStop: the fallback when the subagent stops without going through the handback tool. A delivered handback report that carries the line lets it through (a refused handback does not count);
  after a stop is blocked, writing the line in the reply or in a written report and then stopping lets it through (merely running du, ls or Read on it does not count).

The project registers both in `.claude/settings.json`; how to write them is in that hook's header.
What it cannot recognise still has to be deleted: paths that live only in a variable, directories a script created internally that no call mentions, anything more than one level below a directory it created,
source copies with no `.git` that were never built in, and anything created by a detached process (`nohup`, `setsid`, `&`) after the call returned. Whether the report carries the line saying what was deleted and how big, it does not check.
