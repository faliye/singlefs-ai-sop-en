<!-- generated-from: rules/test-script-discipline.md sha256:6cabd734cb37924585a6dba8a1062dcbbd0b885bd0d43e9fbd9737da7c9ae1f8 -->
<!-- doc-lint:rule-definition -->
# How to write test cases and test scripts

Covers the project's own verification code: unit and integration tests, heavy verification by exhaustive enumeration or long random runs, rigs on real devices and external tools, and entry scripts. An entry script is a script that organises and runs a batch of registered items; a tool that runs over one table or one input (the kind that requires positional arguments) and a helper script that only reads logs are not entry scripts.
They are equally bound by `show-me-test.md` (a new test is first shown to go red), `test-discipline.md`, `command-safety.md` and `preflight-discipline.md`.

## Atomic cases: every case can run on its own

**Criterion: running one case alone by name gives the same verdict as running it inside the full run.**

- One case, one scenario. A loop over parameters that rebuilds the state under test on every turn is split into one function taking the parameter plus one case per value; where it really is one scenario (several steps in order on one state), write a one-line comment beside the case saying why.
- Do not depend on run order, and do not read state another case left behind: files, global variables, environment variables, temporary directories. Each case builds and removes its own (`command-safety.md` "Test images always go in a temporary directory").
- Random input carries a seed. On red, print the seed; the seed alone reproduces that one case.
- A case name states the scenario and the expected outcome (the "Names" section of `code-discipline.md`), with no history such as milestone, step number or batch; history goes in the commit message or the project kb.
- On red, rerun only the red ones: the entry script can take the red cases out of the previous run's output and give the command that runs just those. If the number taken out does not match the failure count in that run's summary, it is red, gives no command, and the next step reads "rerun the whole set once" (`command-safety.md` "Result collection needs a completeness gate").

## Entry script: no arguments means the full run

| Argument | What it does |
|---|---|
| none | full run: every item of the quick tier and the slow tier |
| `--quick` | only the quick tier |
| `--list-items` | lists the nameable items and their tier, one per line; builds nothing, runs nothing |
| `--item <item>` | runs only the named items; given several times, takes the union |
| an item name that is wrong, or `--item` without a name | lists the nameable items, exits 2 |

- A gate stage is not an entry script: `gate.sh` starts stages with no arguments; a stage that starts the test framework directly or calls an entry script names the tier explicitly either way (`--quick`, `--item`, or the test framework's own filter arguments).
- When tiers are marked with the test framework's own markers (`#[ignore]` and the like), a run with no arguments runs the marked ones too (`--include-ignored`).
- If a scale variable (number of seeds, expansion depth and the like) was set, that run's product writes out its value; the full run's completion marker does not accept a run whose scale was reduced.
- Tools, devices and hardware the whole script cannot do without are written as a `run-condition` (`preflight-discipline.md`) and exit 78 when not met; where only some items need them, those items are reported one by one as "not run this time" under `show-me-test.md` "The gate must not pretend to pass", and that run does not say "full run passed".
- Each item prints one line when it ends: item name, exit code, how many judged, how many red, how many reused (an item with no verdict-reuse layer writes "no reuse"; one that has the layer but could not read the number this run writes "unavailable", is left out of the total, and is named in the success line; neither writes 0), seconds taken; the last line is the total. The number of items collected is checked under `command-safety.md` "Parallelism must not swallow the failures".
- Exit codes 0 everything that ran is green (still 0 when some items reported "not run this time"), 1 some red, 2 usage error, 77 no item ran at all, 78 admission or run condition not met (`preflight-discipline.md`); these five have fixed meanings. Other forms of refusal may use other exit codes, listed one by one in the head of the file.

## The test script itself must be proven valid by a test

**Criterion: the test script and the judging it uses both have a self-test; break them and the self-test goes red** (as in `test-discipline.md` "The check itself can also be wrong", applied to the project's test scripts).

- Write them in a testable shape: the judging is a pure function; the item driver, devices, clock and random source can all be swapped for fakes through arguments or environment variables; no hard-coded paths or external services. Any part that cannot be swapped is named in the head of the file.
- The self-test is `--selftest`, tests in the same package, or a directory of red and green fixtures (each fixture carrying a file that states the expected exit code and output; an empty directory a fixture needs gets a `.keep`, since git cannot store an empty directory, judged by the gate stage "Script executable bits" (`脚本执行位`)): every kind of verdict (green, red, not run this time, usage error) is fed at least one input with a known answer and judges the same as the answer; the entry script's listing, naming, refusing an unknown item name, failure counting and item-count check are each walked once.
- Every branch that can judge red gets a break switch or a mutation: once broken, the self-test goes red, on the named item. Break switches are listed one by one in the head of the script: switch name, what it breaks, which item should go red when it is on; mutations go into the checked-in mutation list (`show-me-test.md` "A new test must first be shown to go red"). Whoever replays them (the self-test itself, or the gate's self-test) switches each one on and watches it go red on the named item. The switches are honoured only by the self-test, never in an ordinary run.
- The self-test is wired into the gate or the quick tier, so that changing the test script or the judging runs it. The self-test uses fake drivers and starts no build or heavy work, so whoever changes the script can run it at any time; the project's guard against heavy tests does not block the self-test.
- The self-test also keeps to "Atomic cases: every case can run on its own".

## Two tiers, quick and slow

| Tier | What it holds | When it runs |
|---|---|---|
| quick tier | cases the project's quick/slow criterion classes as quick, plus the kinds that always go into the quick tier | after every change |
| slow tier | cases the quick/slow criterion classes as slow: exhaustive, long random, real device, external tools and the like | with the full run: before commit or on the user's request; any other timing is set by the project |

- The quick/slow criterion and each tier's command are written in the project-local rules. The criterion may be a per-item time threshold (judged by what was measured in ordinary runs), or a split by whether a case enumerates exhaustively or needs a real device or external tool; when unsure, it goes to the slow tier.
- The tier is marked on the case or in a registry table, not in a second list copied into the script.
- These kinds always go into the quick tier, whatever the quick/slow criterion says: a small-input case for every kind of judging in the slow tier (same judging, small input), the pruning self-test, the multi-threaded versus single-threaded differential, the before-and-after-optimisation differential, the accelerated versus reference path differential, and reds pinned back from the full run. Those that need particular hardware report "not run this time" on machines without it.
- Where tiers are split by a compile switch (a feature, `cfg` and the like), at least one run in the quick tier compiles it in; a kind that compiles in with 0 cases is reported as "not run this time" and does not count as run.

## Reusable verdicts: stored by input, pruned by prefix

- Every verdict is stored under a key computed from all the inputs it depends on: the code under test, the judging code, the parameters, the toolchain. A matching key is reused, not recomputed.
- The verdict store is not this run's output: the output is written fresh every run under `show-me-test.md` "The gate must not pretend to pass", and reused verdicts are reported in it separately from those judged now; where the store lives follows the sentence on cross-run caches in `command-safety.md` "Test images always go in a temporary directory".
- The state space is organised by shared prefix: a prefix shared by several streams or items is judged once, and what follows is judged on top of it.
- Only two kinds of thing may be pruned: those with a matching key that have already been judged; and those equivalent to an already judged state. The equivalence is pinned by a case.
- A class left unjudged wholesale by registration is not pruning: the registry has one row per class saying why it is not judged, the success line reports how many were skipped, and the classes not judged are listed under `show-me-test.md` "The gate must not pretend to pass".
- Pruning has a self-test: on a small input, run once with pruning and reuse off; the verdicts match the run with them on, item by item. The pruning self-test goes into the quick tier.
- A block is written to disk only once it is fully judged, and the write is atomic; after an interruption, rerunning the same command resumes, filling in only what was not judged.
- The success line reports "judged now N, reused M".
- When the whole script's inputs have not changed, it does not rerun: write `admission: inputs-changed` (`preflight-discipline.md`); reuse at the verdict level is done inside the script. Both layers are required.

## Fast to build, fast to run

- Heavy cases run on an optimised build, with overflow checks and assertions still on (Rust: `opt-level = 3` plus `overflow-checks = true`, and every `assert!` kept). Whether assertions that only take effect in debug builds (`debug_assert!` and the like) are on is decided by the project after measuring the cost; if they are off, the measured cost is written beside the build configuration.
- One build per run: several items share one build; build output is reused across runs (where it lives follows the sentence on cross-run caches in `command-safety.md` "Test images always go in a temporary directory") and is not cleaned before running.
- The hot path does nothing unrelated to judging: formatting, per-item logging, repeated allocation.
- Independent units within a process are sliced by range and run in parallel, with the degree of parallelism under `command-safety.md` "Within one script, run the checks in parallel when they can be". Merging is deterministic: counts are summed in slice order, "the first one" is the one with the smallest ordinal; the multi-threaded versus single-threaded differential is identical word for word and goes into the quick tier. Where something cannot run in parallel (shared state, or the order itself is what is under test), say why in a comment on that case.
- An item that runs longer than the progress interval the project sets prints at least one progress line per interval: each slice reports its range and how many it has judged.
- Optimisation must not swap out the criterion: before and after, verdicts on the same set of inputs match item by item, and that differential goes into the quick tier.

## Use the hardware acceleration at hand

- Which kinds of computation go to acceleration hardware (a GPU and the like), which machines are used, and how much of each, are registered in the project's local configuration; the registered kinds go only through the accelerated path, and as many machines as are configured each get a share.
- Where hardware or a machine that the configuration turns on is something the whole script cannot do without, write it as a `run-condition: check`: if it cannot be used (device absent, wrong driver, kernel fails to build, not enough device memory, the second machine unreachable), exit 78 with the next step "fix the device, or change the configuration"; do not quietly fall back to the CPU or to one machine. To fall back, change the configuration or pass it explicitly.
- The accelerated path keeps a CPU reference implementation, and the two agree verdict by verdict on the same set of inputs; that differential goes into the quick tier.
- The accelerated path is also shown red first: put a known fault into the accelerated side, and the differential must go red.

## A red from the full run is pinned back into the quick tier

- Every red from the full run has the input that triggered it turned into a quick-tier case or a row of a registry table, and is re-judged in the quick tier; after the fix it stays as a regression and is not deleted.
- Before pinning, shrink the input to the smallest one that is still red where it can be shrunk; where it cannot, say why and pin it as it is. After pinning, check once: on the code that went red, the pinned case is red too, on the same assertion.
- The project keeps a pin table with one row per pinned case: the line that went red, copied whole with the product's file name, and which change (a mutation or that commit) turns it red when put back.
- A red is closed only when every red in the full run's products has its row in the pin table.

## Which half the gate handles

**None of it is a shared check yet.** The parts that can be judged literally can be judged by a local stage the project wires into `.claude/gate.d/`:
the entry script's argument conventions (`--list-items` exits 0 and starts no build or heavy work; a wrong item name exits 2), whether a self-test (in one of the three forms) exists and runs green, whether each break switch listed in the head of the file, once switched on, turns the self-test red on the named item, whether every red in the full run's products is in the pin table, whether loops over parameters have been split into cases, whether the differentials that always go into the quick tier are in it.

Left to review: whether the self-test covers every kind of verdict, whether the cases really are independent of each other, whether the reuse key covers all inputs, whether the pruning equivalence is right, whether the quick/slow criterion is reasonable, whether the hot path does anything unrelated to judging, whether every computation that should go to acceleration hardware is registered, whether a pinned case and the red from the full run are the same defect.
