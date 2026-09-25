<!-- generated-from: rules/preflight-discipline.md sha256:85d69b77fa305080b373d73d0aeacc47b5fd7e36381c22c7b1d1e01752f39834 -->
<!-- doc-lint:rule-definition -->
# Admission and run conditions: decide whether it may run, then run it

**Every script states at its head when it should be called and when it must not be, and checks that before it does any work: if a condition is not met it refuses to run, and only `--force` makes it run anyway.**

This covers every script: this package's `install.sh` and `scripts/` (including `scripts/claude-hooks/` and `scripts/githooks/`),
the project's `.claude/gate.d/`, `.claude/scripts/` and `.claude/hooks/`, and the directories the project registers in `.claude/preflight-dirs` —
register wherever the experiment scripts and the source files of experiment binaries live. Each directory counts only its own level; register a subdirectory on a line of its own.
`.claude/preflight-dirs` has one `<directory>  # what lives there` per line; a project without experiments still creates the file, with one comment line saying so.

Three kinds are not judged: libraries that are sourced or imported, fixtures, and wrappers that only `exec` a shared script (the few lines install.sh lays down; a wrapper with any other logic in it is judged like any script).
List libraries and fixtures one by one in an exclusion table: this package's in `PACKAGE_EXCLUDED` in `scripts/preflight-lint.py`, the project's in `.claude/preflight-exclude`,
one `<path>  # reason` per line, with a reason of at least 4 characters. Scripts not yet converted go in the same table one by one; delete a line as each is done, so the table only shrinks.

## The two kinds of condition

| Condition | What it answers | Typical case |
|---|---|---|
| Admission `admission:` | When it should be called: whether this call can tell anything new | Code, decisions and preregistration unchanged since the last successful run, so rerunning yields nothing new |
| Run `run-condition:` | When it must not be called: whether the environment can carry it, or would taint the result | A missing tool, a missing device or permission, another instance of the same script already running |

Every script writes at least one line of each kind; when there are several lines, all of them must hold. An experiment's admission is `inputs-changed`, listing every input its result depends on: code, decisions, preregistration.

## How to write them

In the comment block at the head of the file (after `#!`, before the first line of code; `#` for shell and python, `//` for Rust, where inner attributes `#![…]` may sit among the lines), one per line:

| Form | Condition met when |
|---|---|
| `admission: always <reason>` | Every call is meaningful. The reason says why, for example that it judges the whole repository as it stands now |
| `admission: inputs-changed <path…> [env:<variable>…] [arguments]` | The script itself or a listed input has changed since the last successful run. Paths are relative to the repository root, or to the script's directory when they start with `./` or `../`; `env:<variable>` and `arguments` count an environment variable and this call's arguments as inputs. Several such lines are merged into one set of inputs |
| `admission: check <command> :: <what to do if not met>` | The command exits 0 |
| `run-condition: none <reason>` | No requirement on the environment |
| `run-condition: command <executable…>` | All of them are on `PATH` |
| `run-condition: single-instance` | No other process is running the same script |
| `run-condition: check <command> :: <what to do if not met>` | The command exits 0 |

The reasons of `always` and `none` and the remedy of `check` are at least 8 characters.
A `check` command runs under `bash -c` at the repository root (in the script's directory outside a git repository), with `PREFLIGHT_SCRIPT` and `PREFLIGHT_SCRIPT_DIRECTORY` in its environment, and cannot read the caller's standard input.
Outside a git repository, whether the inputs changed cannot be judged: the script runs, and no fingerprint is recorded.
Parsing and evaluating the declarations live in one place only, `scripts/preflight.py`; its behaviour is authoritative for the forms.

## Check first, at the head

| Language | How |
|---|---|
| shell | After `source lib.sh` (hooks that do not source lib.sh source `preflight.sh`) and before the first thing that does work, copy this line verbatim: `preflight "${BASH_SOURCE[0]}" "$@"; set -- ${PREFLIGHT_ARGUMENTS[@]+"${PREFLIGHT_ARGUMENTS[@]}"}`. Before it only `set <options>`, `shopt`, `source`, lines that are nothing but assignments and do not read the arguments, and `unset` are allowed |
| python | Set `sys.dont_write_bytecode = True` first, then import this package's `scripts/preflight.py`; the first statement under `if __name__ == '__main__':` calls `preflight(__file__)`; without that block, the call goes before the module's first statement that does work. Top-level statements before it may not read the arguments or standard input, start a subprocess, or open a file |
| Rust and other languages | The first statement of `main` calls a function named `preflight`: it starts `python3 <spec copy>/scripts/preflight.py check <absolute path of the source file> [--force] -- <arguments…>` directly (not through `sh -c`) and exits with the same code if that is not 0; when the line on stdout starts with `met`, it keeps the fingerprint in its last field, and when it starts with `forced`, it treats the summary in its last field as `PREFLIGHT_FORCED` |

A script that declares `inputs-changed` calls `preflight_record_success` after a successful run, before it exits (Rust runs `preflight.py record <source file> --fingerprint <fingerprint at start> -- <arguments…>`).
What is recorded is the fingerprint judged at the start; nothing is recorded if the inputs changed by the end (someone edited them during the run), if the run was forced, or if it failed.

## When a condition is not met

- Without `--force`: print each unmet condition with its remedy, exit with code 78, and do nothing else. Without `python3` the conditions cannot be judged, and that is treated as unmet too. Exit code 78 is used for this refusal only.
- With `--force`: run anyway, print each unmet condition, and set `PREFLIGHT_FORCED` to their summary.
  A script that writes an artifact writes `PREFLIGHT_FORCED` into it; whoever cites that artifact says it was a forced run.
- Malformed conditions (an unrecognised form, a reason too short) cannot be judged: the script exits 1, and the gate records that stage as failed.

## How gate.sh orchestrates

- Before starting any stage it checks that stage's conditions; if they are not met it does not start the stage, and the summary records "not run this time" (`本次未跑`) with the unmet conditions.
- `gate.sh --force` passes `--force` to the stages whose conditions are not met; a stage run that way is recorded as "run by force" (`强制跑过`) in the summary,
  not as a pass, and it does not count as covering any item on the not-implemented list.
- Whether a stage starts is decided by the gate's check alone: a stage that exits 78 after it has started is recorded as failed.
- If any stage was run by force, or was not started for a reason other than "inputs unchanged", gate-ok is not advanced this round and the closing line does not say "all passed";
  the gate's exit code only reflects whether any stage failed.

## Which half the gate covers

The gate stage "admission and run conditions" (`准入与运行条件`, `scripts/preflight-lint.py`) judges: whether both kinds are declared, whether each form is recognised, whether they sit before the first line of code,
whether the paths listed by `inputs-changed` exist, whether `preflight` is called first (for shell, also that lib.sh or preflight.sh is sourced before it),
whether a script with `inputs-changed` calls `preflight_record_success`, and whether the exclusion and registration tables point at something.
When the project has no `.claude/preflight-dirs`, the summary records the experiment-script category as "not checked this time".

What it cannot judge: whether the conditions are right and complete (an input missing from `inputs-changed`, a `check` that tests the wrong thing),
whether the reasons for `always` and `none` hold, whether an artifact carries the forced mark, and declarations written after the `preflight` line (they are not honoured at run time either). These are left to review.
