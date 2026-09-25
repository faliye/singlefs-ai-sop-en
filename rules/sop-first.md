<!-- generated-from: rules/sop-first.md sha256:7ae9f9cb294449321d3aeea77d638758ee469e124257a8fba784d4d9f302e7ea -->
<!-- doc-lint:rule-definition -->
# SOP before code

**The spec and the gate outrank any line of implementation code.**

In any round of work, if "add one gate check" conflicts with "write one feature",
add the gate check first.

## Corollaries

- **A script existing before the thing it tests is normal**, not backwards.
  A harness can be written and self-checked with no workload; plug the
  implementation in when it arrives.
- **The first reaction to a newly found trap is "how does this become a red line
  in the gate"**, not "which document does this go in". A reminder sentence stops
  nobody; a check that fails does.
- **Gate scripts need tests too**: change `scripts/` and you must construct an
  input that ought to be rejected, and confirm it really goes red. A gate that
  cannot check itself is decoration.
- SOP changes bump `VERSION`, so the project learns the rules moved the next
  time it runs the gate.

# The gate is a teaching instrument, not a sieve

**Assume by default that a submitter wants their patch to pass.**

When they are stopped, most of the time it is not that they did not want to test —
it is that they **did not know how to test.** On that premise, the gate's job is
not "keep bad things out" but **to state what "done" means clearly enough that
nobody has to guess.**

## Every rejection must carry a next step

A failure message that only says "not acceptable" leaves "what would have been
right" to guesswork — and people who can only guess will route around the gate, or
simply not submit. **Both outcomes are worse than letting the patch through.**

So: **when rejecting, state what to do next.**
Enforced by `scripts/gate-lint.sh`, which covers every shape of rejection:

| Shape | Where the remedy goes | Criterion |
|---|---|---|
| `bad` | the `howto` right after it | a `howto` within 5 lines counting the `bad` itself (comment lines do not count) |
| `die` | **its own second argument** (`die "what blocked" "what to do"`) | a `die` carrying only one argument fails |
| a printed `✗` (an `echo`, or a `print` in embedded python) | the `→` line before the next rejection | no `→` in between fails |

The `die` row is not an afterthought: `die` is `bad` + `exit`, so it is a rejection
too — and a rejection as remedy-free as `die "unit tests failed"` is one that checking
only `bad` cannot find.

The third row covers **a project's own local stages**. Most of them do not source
`lib.sh`; they `echo "  ✗ …"` directly, or put the criterion inside embedded python:
`print('  ✗ …')` — neither of the first two rows reaches them.

The window is not 5 lines here: python often prints one `✗`, then loops through the
offending items, and only then gives the remedy — 5 lines would misjudge the whole
batch. So the criterion is "the remedy appears **before the next rejection**". Both
exemptions must be **written out explicitly**; nothing gets guessed. A per-item line
inside a loop carries `# gate-lint:detail` (its remedy lives on the summary), and a
summary line carries `# gate-lint:summary`.

When you cannot write the `howto`, **do not add the check yet**: if you cannot
state the next step, the criterion behind the check is not clear to you either.

## Three properties of a good gate

1. **Runnable locally, and it agrees with the remote.** Submitters must be able
   to see the result before sending, not learn it from a rejection.
2. **Failure messages point at the rule, not just the symptom.** Let people learn
   the rule rather than paper over this one instance — otherwise the same problem
   comes back.
3. **Make the right thing easy.** Templates, skeletons, ready examples to copy are
   all part of the gate.

## The compounding effect

State the rules clearly enough and newcomers pick up the rhythm within a few
rounds; development then goes *faster*, because **nobody has to guess whether
something counts as done.** Code quality rises across the board at the same time,
because nobody wants their work rejected — all they were missing was a clear path.

## Boundary

SOP-first does not mean unbounded SOP growth. **Before a rule gets in, ask "can
this become a check that fails?"** — if it cannot, it is most likely not thought
through yet; leave it in the kb as open, do not write it as a rule.

## Before adding a gate or hook, look for an existing one

Before adding a gate or a hook, check whether an existing one already covers the same thing, the same set of objects, or the same trigger:

1. Run `python3 scripts/gate-overlap.py --list` (in a project, run the installed copy: `python3 .claude/singlefs-ai-sop/scripts/gate-overlap.py --list`) to see what each existing gate and hook judges and which trigger each hook is attached to.
2. If one already covers the same thing, append the new criterion to it; if two judge the same objects from the same input, merge them into one.
3. If two only share a piece of logic (reading the same table, applying the same criterion), extract it into a shared library that both call; do not copy it a second time.
4. If the new one really has to stand alone, name in the new file each existing one you compared it with and why it is not merged in: `# gate-similar: <existing file name> <why not merged into it>`; if nothing is similar, write `# gate-similar: 无 <what you checked>`.
   Every existing hook on the same event with an overlapping matcher, and every existing gate or hook that is textually very similar, must be named; `无` does not count for them.
5. If an identical passage really has to stay in two files, write `# gate-overlap:copy-kept <the other file name> <why it is not extracted>` in one of them.
6. A new hook writes `# hook-events: <event> …` in its header, listing every event it must be attached to (one that should fire only for a particular tool is written `<event>:<tool name>`), and is registered once per event in `.claude/settings.json`; one written with a tool name is registered on a matcher that recognises that tool.

### Who checks

- **When an agent stops**: the stop hook `scripts/claude-hooks/gate-reuse-check.sh` checks the gates and hooks that this agent itself created or changed. If `gate-similar` or `hook-events` is missing, something that must be named is not, or a passage was copied wholesale from an existing file, it blocks the stop and makes the agent self-check first.
  The same judgement blocks only once within a continuation that a stop hook sent back; a changed judgement blocks again.
  Both the main agent and subagents are covered: the project registers the hook once on `Stop` and once on `SubagentStop` in `.claude/settings.json`; the registration is spelled out in the hook's header.
- **Before commit**: the gate stage "Gate overlap check" (`门禁查重`, `scripts/gate-overlap.py`) judges the gates and hooks added or changed in the diff window: whether `gate-similar` and `hook-events` are written, whether the named files are existing gates or hooks, whether each reason is long enough, whether everything that must be named is named, whether added lines duplicate an existing file wholesale, and whether `copy-kept` is well formed.
  The threshold for "wholesale" is whatever `CLONE_MINIMUM_LINES` says in that script.
- The gate stage "Tool-layer gates" (`工具层的闸`, `scripts/hooks-registered.sh`) checks that the hooks are registered, that every event in `hook-events` is attached, and that one written with a tool name is attached on a matcher that recognises that tool.

Whether the named file really is the closest match, and whether the reason for not merging holds, the gate cannot judge; that is left to review.
