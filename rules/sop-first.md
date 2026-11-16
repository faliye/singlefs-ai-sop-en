<!-- generated-from: rules/sop-first.md sha256:392121d37e0810a29411c7c4a5d64b890ed3087bde020b6e5ec1a772444144d5 -->
<!-- doc-lint:rule-definition -->
# SOP before code

**The specification and the gate outrank any line of implementation code.**
In any round of work, if "add one gate check" conflicts with "write one feature",
add the gate check first (this means a gate already decided on; when to add one for a newly found trap follows the "Boundary" section: in batches).

## Corollaries

- **A script may exist before the thing it tests**: write and self-check the harness with
  no workload; plug the implementation in when it arrives.
- **The first reaction to a newly found trap is "how does this become a red line
  in the gate"**, not "which document does this go in".
- **Gate scripts need tests too**: change `scripts/` and you must construct an
  input that ought to be rejected, and confirm it really goes red.
- SOP changes bump `VERSION`.

# The gate is a teaching instrument, not a sieve

The gate must **state what "done" means clearly enough that nobody has to guess.**

## Every rejection must carry a next step

`scripts/gate-lint.sh` checks every shape of rejection:

| Shape | Where the remedy goes | Criterion |
|---|---|---|
| `bad` | the `howto` right after it | a `howto` within 5 lines counting the `bad` itself (comment lines do not count) |
| `die` | **its own second argument** (`die "what blocked" "what to do"`) | a `die` carrying only one argument fails |
| a printed `✗` (an `echo`, or a `print` in embedded python) | the `→` line before the next rejection | no `→` in between fails |

A printed `✗` has two exemptions, both written out explicitly: a per-item line
inside a loop carries `# gate-lint:detail` (its remedy lives on the summary), and a
summary line carries `# gate-lint:summary`.

**When you cannot write the `howto`, do not add the check yet.**

**The remedy itself must pass the other gates.** The side that writes the rejection feeds the remedy command it prints, in its self-test, to the project's other gates on the same trigger point, and fails if any of them rejects it; where it cannot feed them (the other gate is not in this package, or the project has not registered it), the self-test's success line reports "not run this time".

## Three properties of a good gate

1. **Runnable locally, and it agrees with the remote.**
2. **Failure messages point at the rule, not just the symptom.**
3. **Make the right thing easy.** Templates, skeletons, ready examples to copy are
   all part of the gate.

## Boundary

**Before a rule gets in, ask "can this become a check that fails?"**

- If it cannot, it does not enter this package: put it in the project-local `.claude/rules/`, or leave it in the kb as open for now. Every section of this package's rules states which script or hook judges it; a section that cannot say so moves to the project.
- If it can, it becomes a check that goes red, not a reminder; when to add it is decided in batches: a trap you hit is first recorded in the project's debt table (saying what input it must stop), and the batch is assessed once a week.
- Within a batch, checks that judge the same thing on the same set of objects merge into one; each one added on its own states which past occurrences it stops.
- A trap that keeps recurring right now, or that destroys data, is added on the spot without waiting for the batch.
- Every gate and every hook states its retirement criterion: several consecutive weeks with no red, or the objects it judges no longer exist; once that is reached, delete it.

## Before adding a gate or hook, look for an existing one

1. Run `python3 scripts/gate-overlap.py --list` (in a project, run `python3 .claude/singlefs-ai-sop/scripts/gate-overlap.py --list`) to see what each existing gate and hook judges and which trigger each hook is attached to.
2. If one already covers the same thing, append the new criterion to it; if two judge the same objects from the same input, merge them into one.
3. If two only share a piece of logic, extract it into a shared library; do not copy it a second time.
4. If the new one really has to stand alone, name in the new file each existing one you compared it with and why it is not merged in: `# gate-similar: <existing file name> <why not merged into it>`; if nothing is similar, write `# gate-similar: 无 <what you checked>`.
   Every existing hook on the same event with an overlapping matcher, and every existing gate or hook that is textually very similar, must be named; `无` does not count for them.
5. If an identical passage really has to stay in two files, write `# gate-overlap:copy-kept <the other file name> <why it is not extracted>` in one of them.
6. A new hook writes `# hook-events: <event> …` in its header, listing every event it must be attached to (one that should fire only for a particular tool is written `<event>:<tool name>`), and is registered once per event in `.claude/settings.json`; one written with a tool name is registered on a matcher that recognises that tool.

### Who checks

| Who | What it judges |
|---|---|
| The gate stage "Gate overlap check" (`门禁查重`, `scripts/gate-overlap.py`) | Before commit, the gates and hooks added or changed in the diff window: whether the markers of steps 4, 5 and 6 are written and well formed, whether the named files are existing gates or hooks, whether each reason is long enough, whether everything that must be named is named, and whether added lines duplicate an existing file wholesale (the threshold is whatever its `CLONE_MINIMUM_LINES` says) |
| The gate stage "Tool-layer gates" (`工具层的闸`, `scripts/hooks-registered.sh`) | Whether the hooks are registered, whether every event in `hook-events` is attached, and whether one written with a tool name is attached on a matcher that recognises that tool. A hook of this package whose header has `# hook-registration: optional <reason>` may be left unregistered by the project; the line counts for nothing in the project's own hooks |
| The optional stop hook `scripts/claude-hooks/gate-reuse-check.sh` | When an agent stops, the gates and hooks that this agent itself created or changed, by the criteria of "Gate overlap check" |

Whether the named file really is the closest match, and whether the reason for not merging holds, is left to review.
