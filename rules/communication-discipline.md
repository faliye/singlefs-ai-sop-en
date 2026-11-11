<!-- generated-from: rules/communication-discipline.md sha256:8aba64a1b8736928d7c376173febc2b7026bcf15eea18f84418059545eee63ca -->
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

## Scope and exceptions

Output written for a third-party reader (email, announcement, external document) follows its reader and is out of scope.
