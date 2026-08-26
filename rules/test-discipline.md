<!-- generated-from: rules/test-discipline.md sha256:80194a083f390eba2bedbcca56dd1e3000407f3164c0c626b1a7bd37996c1c33 -->
<!-- doc-lint:rule-definition -->
# Testing discipline

## A single observation does not count

N≥5 rounds. To call it "pass" every round must pass; to call it "fail" every round
must fail (**the thresholds are asymmetric**). If neither holds, report "unstable"
honestly and draw no conclusion.

## Could not read ≠ read zero

If a metric comes back empty, retry and **void the whole round**. Never let it enter
the judgement as a zero.

## Crash consistency can only be verified by crash-point replay

Record every write request at the block layer, then truncate at **every** possible
crash point, replay, and run the checker.
**A write path that has not been through this is not verified** — an all-green unit
test suite is no evidence of crash consistency whatsoever.

## Functional correctness comes from model-based differential testing

Maintain an "ideal filesystem" in memory (just a HashMap; no performance, no crashes),
apply the same random operation sequence to both the model and the implementation, and
compare.

**This is the only functional oracle this project has.** Designing from scratch means
there is no reference implementation to compare against — the convenience a porting
project has, of "treat an existing tool's output as the right answer", does not exist
here; it has to be built.

## An experiment's failure clause must not make its conclusion unfalsifiable

The easiest mistake to make when designing an experiment, and the hardest to catch
yourself: **defining "the effect did not show up" as an implementation bug**.

Write it that way and the experiment can only ever produce supporting evidence —
the effect appears and you record a finding, it does not appear and you record
"implementation is broken, discard the round". **No observation can refute the conclusion.**

Keep the two apart, and **run both controls**:

| Control | What it is for | Verdict when no difference shows |
|---|---|---|
| **Positive control**: a baseline where the effect is firmly established in the literature | Proves the measurement has discriminating power | **The implementation is broken, discard the round** |
| **Real baseline**: the design this project actually intends to build | Answers what the experiment is really asking | **A legitimate result, record it as such** |

Without the positive control, "no advantage" cannot be told apart from a broken
implementation. Without the real baseline you are measuring "does this mechanism exist"
rather than "what does this mechanism buy over what we already have".

## The checker is the specification

Every entry added to the invariant list adds a check to the checker.
**The authoritative answer to "what is this format" is the checker's source, not the
documentation.**

## The check itself can also be wrong

Before asserting, confirm the check **has discriminating power** — e.g. when verifying
"there are no invalid blocks", also confirm the scan actually read some blocks;
otherwise finding none may only mean nothing was scanned.

After writing a check, ask: if the thing under test really were broken, would this go
red? If you cannot answer, it is not finished.

## A negative result must be separable from "the code never ran"

"Did not reproduce" does not mean "no problem". To claim "this path is correct", you
need a reading proving that path **actually executed**.
