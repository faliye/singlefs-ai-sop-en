<!-- generated-from: rules/communication-discipline.md sha256:db69402d0c8bf4462cf8309635a02196802f0b33226c83842b10af2cbc87c669 -->
<!-- doc-lint:rule-definition -->
# Communication discipline

**Covers**: conversation from an agent to the user, and between agents.
Output written for a third-party reader (email, announcement, external document) follows its reader and is out of scope.

The hooks each section names are registered in `.claude/settings.json`; how is in each hook's file header. Which phrases each one recognises is whatever the hook says.

## No pleasantries

Replies and messages do not contain: greetings, thanks, apologies, praising the question, "Sure", "Of course", "Happy to help", closing with "Anything else?".

The stop hook `scripts/claude-hooks/courtesy-phrase-check.sh` judges this: when the last reply of the turn contains a phrase from the word list (`COURTESY_PHRASES`), it blocks the stop and requires the reply to be rewritten without it. Anything inside backticks, 「」, quotation marks or a code block is not judged. The word list is Chinese only.

## Verify a guess before answering

When a reply contains the wording of a guess ("可能是", "应该是", "大概", "估计是", "似乎", "我记得" and the like; in English: "probably", "should be", "I think", "as far as I remember"), or wording that labels itself a guess or unverified ("推测", "推论", "这是推的", "没核实", "没量过" and the like; in English: "this is an inference", "not verified", "not measured"), verify first, then answer:

1. Read the file, run the command, look up the record, and get the result.
2. Rewrite that sentence as the verified conclusion, with its evidence (the path, the command and its output).
3. What cannot be verified (it needs information from the user, or it is about something that has not happened yet) carries `未验证：<why it cannot be verified>` on the same line, with a reason of at least 4 characters.

The stop hook `scripts/claude-hooks/courtesy-phrase-check.sh` judges this: when the last reply of the turn contains a phrase from the word list (`SPECULATION_PHRASES`) and the same line does not carry `未验证：`, it blocks the stop and requires the reply to be rewritten after verifying. Anything inside backticks, 「」, quotation marks or a code block is not judged. The word list is Chinese only.
Whether the verification was actually done, and whether the reason after `未验证：` holds, is left to review.

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
- When the number is already held by another running session, the session that started later changes its number; the one that started earlier keeps it.
- The title is Claude Code's automatically generated title when there is one; until there is, a title taken from the start of the user's prompt (cut to a display width of 40, a CJK character counting 2). A title that is only `(number)` is not left in place.
- Once the text after the number is written it is not replaced; when the number has been removed, it is put back.
- Subagents and non-interactive sessions are not titled.

The hook `scripts/claude-hooks/session-number-title.sh` (the writing is in `scripts/session-number-title.py`) does this on SessionStart and UserPromptSubmit; what happens when the new number cannot be read is whatever the hook does.

## Checks at session start and after compaction

Checks at session start, resume, clear and after compaction only report and change nothing. The hook `scripts/claude-hooks/session-checks.sh` (judgement in `scripts/session-checks.py`) does them on SessionStart:

- After compaction it first adds a line-up reminder written in this package's language; the first point is that conversation and output use this package's language. The others are the branch and the number of uncommitted paths, checking whether dispatched subagents are still running, and taking the state to resume from the records.
- A project registers what to check in `.claude/session-checks/`: one executable file per item, whose header has `# session-check-sources: <some of startup resume clear compact>` and `# session-check-timeout: <seconds>`; an item that takes `--selftest` must pass it. A project does not write its own hook for these items.
- An item's output on exit 0 goes into the context; on a non-zero exit, a timeout, a failure to start or a wrong header, the session goes on as usual, one line says it did not run, and one line is added to the detection record.

The gate stage "Tool-layer gates" (`工具层的闸`, `scripts/hooks-registered.sh`) judges each item's header, execute bit and `--selftest`. Whether an item's output is right is up to whoever writes the item.
