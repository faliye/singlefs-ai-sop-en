<!-- generated-from: rules/communication-discipline.md sha256:91a3a795382ab2be8c89236642bc56621717fce22089c6e15edc63e795f39d4f -->
<!-- doc-lint:rule-definition -->
# Communication discipline

**Covers**: conversation from an agent to the user, and between agents.
Output written for a third-party reader (email, announcement, external document) follows its reader and is out of scope.

The hooks each section names are registered in `.claude/settings.json`; how is in each hook's file header. Which phrases each one recognises is whatever the hook says.

## No pleasantries

Replies and messages do not contain: greetings, thanks, apologies, praising the question, "Sure", "Of course", "Happy to help", closing with "Anything else?".

The stop hook `scripts/claude-hooks/courtesy-phrase-check.sh` judges this: when the last reply of the turn contains a phrase from the word list (`COURTESY_PHRASES`), it blocks the stop and requires the reply to be rewritten without it. Anything inside backticks, 「」, quotation marks or a code block is not judged. The word list is Chinese only.

## Relaying another place's convention carries its source or its original words

In a prompt sent to a subagent or a message sent to another session, a sentence that says "same convention as X", "do it the way X does", or "same as `X`" carries, in that same sentence, one of these three:

- `path:line` (or a path, or "same file", together with "line N");
- a quotation of the original: at least 6 characters inside 「」, or the sentence ends in a colon and the next line starts with `>`;
- a path together with a function name in backticks, or a heading inside 「」.

When unsure what the original says, read it before dispatching. Writing only your own understanding of it ("same convention: take the qualified name") is not a source.

The hook `scripts/claude-hooks/borrowed-convention-guard.sh` (the judgement is in `scripts/borrowed-convention.py`; the borrowing phrases it recognises are its `BORROW_PHRASES`) judges before dispatching a subagent and before SendMessage, and refuses a sentence that carries none of the three. Whether the source points to the right place, it does not judge.

## Before stopping, check the messages other sessions sent

Before stopping, check once: every message this session received from another session has either been answered to that session, or has a line in the reply reading "no reply to <session name>: <reason>", with a reason of at least 4 characters.
A report handed back by a subagent, and a notice with no session name whose sender is unknown (such as a task service's result notice), do not count.

The stop hook `scripts/claude-hooks/unanswered-message-check.sh` (the judgement is in `scripts/unanswered-message.py`) judges this: it lists each message that has neither an answer nor a "no reply to" line, and blocks the stop.

## To find another session, look only at running ones

To find another session (to ask who owns something, to send a message, to hand over a problem), look only at the running sessions `ListAgents` lists, and copy the name from its row.
Do not dig up sessions by transcript date, and do not message a session that has stopped; when the session you want is not in the list, report what you wanted to say to the user.

The hook `scripts/claude-hooks/message-target-guard.sh` (the judgement is in `scripts/message-target.py`) judges before SendMessage: when the recipient looks like a session name and no running session on this machine matches it, it rejects the send. Subagent and teammate names are not judged.
Sessions on other machines or in the cloud are sent to with the name and `[ref]` from their `ListAgents` row.

## Session titles start with the session number

Each time a session starts or resumes, its title starts with the session number, in the form `(number) title`.

- The number is the part after the directory name in the default name Claude Code gives the session, `<directory name>-<number>`.
- The title is Claude Code's automatically generated title when there is one; until there is, a title taken from the start of the user's prompt (cut to a display width of 40, a CJK character counting 2). A title that is only `(number)` is not left in place.
- Once the text after the number is written it is not replaced; when the number has been removed, it is put back.
- Subagents and non-interactive sessions are not titled.

The hook `scripts/claude-hooks/session-number-title.sh` (the writing is in `scripts/session-number-title.py`) does this on SessionStart and UserPromptSubmit; what happens when the new number cannot be read is whatever the hook does.
