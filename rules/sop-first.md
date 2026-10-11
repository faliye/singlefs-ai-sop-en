<!-- generated-from: rules/sop-first.md sha256:4284674fdf65011fc7f2518723e8ca23de5f235ab4a953d27b80c5dfb6205c89 -->
<!-- doc-lint:rule-definition -->
# SOP before code

**The spec and the gate outrank any line of implementation code.**
In any round of work, if "add one gate check" conflicts with "write one feature",
add the gate check first (this means a gate already decided on; when to add one for a newly found trap follows the "Boundary" section: in batches).

## Corollaries

- **A script existing before the thing it tests is normal**: a harness can be written and
  self-checked with no workload; plug the implementation in when it arrives.
- **The first reaction to a newly found trap is "how does this become a red line
  in the gate"**, not "which document does this go in"; once its form is clear, record it in the project's debt table first, and add it when the "Boundary" section says, in batches.
- **Gate scripts need tests too**: change `scripts/` and you must construct an
  input that ought to be rejected, and confirm it really goes red.
- SOP changes bump `VERSION`.

# The gate is a teaching instrument, not a sieve

**Assume by default that a submitter wants their patch to pass.** The gate must **state
what "done" means clearly enough that nobody has to guess.**

## Every rejection must carry a next step

**When rejecting, state what to do next.** `scripts/gate-lint.sh` checks every shape of rejection:

| Shape | Where the remedy goes | Criterion |
|---|---|---|
| `bad` | the `howto` right after it | a `howto` within 5 lines counting the `bad` itself (comment lines do not count) |
| `die` | **its own second argument** (`die "what blocked" "what to do"`) | a `die` carrying only one argument fails |
| a printed `✗` (an `echo`, or a `print` in embedded python) | the `→` line before the next rejection | no `→` in between fails |

A printed `✗` has two exemptions, both **written out explicitly**: a per-item line
inside a loop carries `# gate-lint:detail` (its remedy lives on the summary), and a
summary line carries `# gate-lint:summary`.

**When you cannot write the `howto`, do not add the check yet.**

## Three properties of a good gate

1. **Runnable locally, and it agrees with the remote.**
2. **Failure messages point at the rule, not just the symptom.**
3. **Make the right thing easy.** Templates, skeletons, ready examples to copy are
   all part of the gate.

## Boundary

**Before a rule gets in, ask "can this become a check that fails?"**
If it cannot, leave it in the kb as open; do not write it as a rule.

**What can become a check takes the form the "newly found trap" item gives: a check that goes red, not a reminder; when to add it is decided in batches.** A trap you hit is first recorded in the project's debt table (saying what input it must stop), and the batch is assessed once a week:
within a batch, checks that judge the same thing on the same set of objects merge into one, and each one added on its own states which past occurrences it stops; a trap that keeps recurring right now, or that destroys data, is added on the spot without waiting for the batch.
Every gate and every hook states its retirement criterion: several consecutive weeks with no red, or the objects it judges no longer exist; once that is reached, delete it rather than keep it running for nothing.

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

- **When an agent stops (optional)**: the stop hook `scripts/claude-hooks/gate-reuse-check.sh` checks the gates and hooks that this agent itself created or changed. If `gate-similar` or `hook-events` is missing, something that must be named is not, or a passage was copied wholesale from an existing file, it blocks the stop and makes the agent self-check first.
  The same judgement blocks only once within a continuation that a stop hook sent back; a changed judgement blocks again.
  A project may leave it unregistered; the overlap check then rests on the "Gate overlap check" stage before commit. A project that registers it registers it once on `Stop` and once on `SubagentStop` in `.claude/settings.json`; the registration is spelled out in the hook's header.
- **Before commit**: the gate stage "Gate overlap check" (`门禁查重`, `scripts/gate-overlap.py`) judges the gates and hooks added or changed in the diff window: whether `gate-similar` and `hook-events` are written, whether the named files are existing gates or hooks, whether each reason is long enough, whether everything that must be named is named, whether added lines duplicate an existing file wholesale, and whether `copy-kept` is well formed.
  The threshold for "wholesale" is whatever `CLONE_MINIMUM_LINES` says in that script.
- The gate stage "Tool-layer gates" (`工具层的闸`, `scripts/hooks-registered.sh`) checks that the hooks are registered, that every event in `hook-events` is attached, and that one written with a tool name is attached on a matcher that recognises that tool.
  A hook of this package whose header has `# hook-registration: optional <reason>` may be left unregistered by the project, and the success line lists each one left unregistered; once registered, it is judged as usual. The line counts for nothing in the project's own hooks.

Whether the named file really is the closest match, and whether the reason for not merging holds, the gate cannot judge; that is left to review.

## Before setting up a verification rig or a test entry point, look for an existing one too

Scope: verification rigs (the code that enumerates, runs differentials, or drives a real device or an external tool), test entry points (the entry scripts of `test-script-discipline.md`), and the gate stages and quick/slow tiers that run them.

1. First list what each existing rig and entry point judges: which matter, which batch of objects (which streams, which enumeration domain), with which judging method, in which stage.
2. Where one judges the same matter on the same batch of objects, merge the new one into the existing set: another algorithm, a move to acceleration hardware, one more stream, one more domain are all written as its acceleration path or new items, under its name, its entry point, its stage and its quick/slow tiers; no new name, no new stage, no second set of quick tier and full run. The acceleration path keeps a reference path and the two are run as a differential, per `test-script-discipline.md` "Use the hardware acceleration at hand".
3. Where only part overlaps (some streams, a stretch of the domain), merge the overlapping part, and add the rest as new items of the same set.
4. Where the repository already has two sets judging the same matter, merge them into one first, then run its pruned run or full run; before the merge, do not run the full run of either set.
5. If it must stand alone, write in the head of the new entry point `# gate-similar: <existing rig or entry point> <why it is not merged into that one>`; if nothing is alike at all, write `# gate-similar: 无 <what was checked>`.

The gate does not yet judge "Before setting up a verification rig or a test entry point, look for an existing one too": whether the head of a new entry point has `gate-similar`, and whether two rigs judge the same matter, are left to review.
