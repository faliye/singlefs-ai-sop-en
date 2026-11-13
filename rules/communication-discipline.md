<!-- generated-from: rules/communication-discipline.md sha256:78082dfca4aee487bb45bf68068ab2f0805a424d3cfb29397a2cd320b01c481f -->
<!-- doc-lint:rule-definition -->
# Communication discipline

**Covers**: conversation from an agent to the user, and between agents.

## No pleasantries

Replies and messages do not contain: greetings, thanks, apologies, praising the question, "Sure", "Of course", "Happy to help", closing with "Anything else?".

The stop hook `scripts/claude-hooks/courtesy-phrase-check.sh` judges this: when the last reply of the turn contains a phrase from the word list, it blocks the stop and requires the reply to be rewritten without it; one reply is blocked only once.
The phrases it recognises are those in that hook's `COURTESY_PHRASES`; anything inside backticks, 「」, quotation marks or a code block is not judged. The word list is Chinese only; replies in English or Japanese are not judged.
A project registers it once each on `Stop` and `SubagentStop` in `.claude/settings.json`; how is in the hook's file header.

## Relaying another place's convention carries its source or its original words

In a prompt sent to a subagent or a message sent to another session, a sentence that says "same convention as X", "do it the way X does", or "same as `X`" carries, in that same sentence, one of these:

- `path:line` (or a path, or "same file", together with "line N");
- a quotation of the original: at least 6 characters inside 「」, or the sentence ends in a colon and the next line starts with `>`;
- a path together with a function name in backticks, or a heading inside 「」.

When unsure what the original says, read it before dispatching. Writing only your own understanding of it ("same convention: take the qualified name") is not a source.

The hook `scripts/claude-hooks/borrowed-convention-guard.sh` (judgement in `scripts/borrowed-convention.py`) judges this: it judges before dispatching a subagent (Agent, Task) and before SendMessage, and refuses when a borrowing sentence has neither a source nor the original words in that sentence.
The borrowing phrases it recognises are those in `BORROW_PHRASES` in `scripts/borrowed-convention.py`; phrases outside it are not judged. Whether the source points to the right place, or the original was copied correctly, it does not judge.
A project registers it in `.claude/settings.json`; how is in the hook's file header.

## Before stopping, check the messages other sessions sent

When the task is done and before stopping, check once: every message this session received from another session has either been answered to that session, or has a line in the reply reading "no reply to <session name>: <reason>".

The stop hook `scripts/claude-hooks/unanswered-message-check.sh` (judgement in `scripts/unanswered-message.py`) judges this: when a message from another session in the transcript has neither a later SendMessage to that session nor its "no reply to" line in the reply, it blocks the stop and lists each one; the same list is blocked only once.
The reason is at least 4 characters. A report handed back by a subagent, and a notice with no session name whose sender is unknown (such as a task service's result notice), do not count as messages from another session. Whether the message needs an answer, whether the answer addresses it, and whether the reason holds, it does not judge.
A project registers it once on `Stop` in `.claude/settings.json`; how is in the hook's file header.

## Session titles start with the session number

Each time a session starts or resumes, its title starts with the session number, in the form `(number) title`. The number is the part after the directory name in the default name Claude Code gives the session, `<directory name>-<number>` (the name shown in the session list).
When the number has been removed by renaming the title, it is put back.

The hook `scripts/claude-hooks/session-number-title.sh` (logic in `scripts/session-number-title.py`) does this: on SessionStart (startup, resume, fork) it takes the number from Claude Code's session registry and writes it into the title; on UserPromptSubmit it puts back a removed number, and when the title has nothing after the number, it fills in the automatically generated title. A title the user chose only gets the number added; it is not replaced.
When resuming a session that already has a title, if Claude Code has already replaced the registry name with the title, this run's new number cannot be read and the old number in the title is kept; when the title has no number either (the session was given a custom title long ago), a two-character number is computed from the sha256 of the session ID, skipping numbers other sessions already use. Subagents and non-interactive sessions are not titled.
A project registers it once each on `SessionStart` and `UserPromptSubmit` in `.claude/settings.json`; how is in the hook's file header.

## Scope and exceptions

Output written for a third-party reader (email, announcement, external document) follows its reader and is out of scope.
