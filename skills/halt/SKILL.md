---
name: halt
description: Use when the user asks for the task to end right here — stop everything in progress, do no more work, send no reply, output nothing, and stay silent until the user types directly.
---
<!-- generated-from: skills/halt/SKILL.md sha256:baf187f463720b9494929c01b05b80ae1e6133d985e28138f48244474282c9b3 -->

Once triggered, the task ends here.

## What to do

1. Call no more tools: start no tasks, dispatch no agents, edit no files, make no commits.
2. Do not reply, and do not respond to any other conversation: messages from other sessions, sub-agents or the task service, and completion notices from background tasks are all left unhandled and unanswered.
3. Output and return nothing: no summary, no farewell, no confirmation that you stopped. End this turn with an empty reply.
4. Stay in this state until the user types a new instruction directly; then wake up and follow that instruction. Do not resume work left unfinished before the stop unless the user asks.

## What not to do

- Do not clean up, roll back or finish anything in order to wrap up; leave the scene exactly as it was when you stopped.
- Do not treat other sessions, sub-agents or system notices as a wake-up: only the user's direct input counts.
