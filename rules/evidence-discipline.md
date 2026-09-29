<!-- generated-from: rules/evidence-discipline.md sha256:d0035ab79ae9bca060680503219ecd5026c3effd2b303801d32d9080db1e675a -->
<!-- doc-lint:rule-definition -->
# Every conclusion needs three derivations: forward, backward, cross-check

**Before any conclusion is stated, all three steps must be complete, and each one
spelled out.** Missing any of them, write it as a hypothesis, not as a conclusion.

| Step | What it asks | If you cannot do this, you did not do it |
|---|---|---|
| **Forward** | derive the conclusion from mechanism / code / data | you can name "which command, which lines of code, which data file" |
| **Backward** | **if this conclusion were false, what should I have seen?** Then go look | you can name a **specific observation, achievable this round**, and state that it did not appear |
| **Cross-check** | reproduce the same judgement by an independent route | the two routes **do not share** the same code, the same sample, or the same tool |

## The cross-check path must itself be shown to go red

Inject a known fault into the subject under test — one the second path *ought* to catch —
and confirm that it does. If it does not, that cross-check is decoration, and every earlier
record of "the two paths agree" is void with it.
Example: when block-layer counters cross-check the I/O counts a program keeps for itself, the
agreement counts as evidence only once dropping direct I/O is seen to send the block-layer count
to zero while the program's own count does not move.

⚠️ **An independent count that matches only the total does not confirm a breakdown.**
Before using a count to confirm a breakdown, ask whether the counter can distinguish the kinds being split. If it cannot, write only "consistent with", say where the breakdown comes from, and check the other raw count as well.

## Background material fed into multi-party argumentation must itself be checked first

When cross-checking a conclusion across multiple independent sources, **every external fact in
the "known facts" brief handed to each party is checked first by the person asking the question**.
Before writing "implementation X works like this" into the brief, check it on the spot; what you
cannot check, mark "unverified".

## Evidence kept verbatim must not be edited afterwards

The prompt you sent to a model, the raw output it produced, the log as it was written — not one
character of them changes within the round.
A prompt can only be edited together with a re-run. To add an explanation, open a
separate file; leave the original alone.

**The freeze holds for this round only; once the round is over, the archive is the version
control history.** Within a round, not one character of its evidence — prompts, the legs'
reports, raw output, stored artifacts — may change. Once that round is committed, the next
commit deletes **the batch the previous one left behind**; this round's own evidence stays.
To look up what was deleted, go read it in the version control history.

⇒ **Old evidence is a reference, not a basis.** When an earlier conclusion is in doubt,
re-verify it rather than digging up the old evidence (`kb-discipline.md` item 2, "All historical
data is reference only"). Overturning an earlier conclusion means running the project's own
inference discipline again, not confronting it with an old file.

⇒ **A check on artifact lines quoted in prose judges "this change", not the whole repo.**
Numbers written into prose today must match today's artifacts verbatim; lines copied down
earlier are historical reference and are not judged again.

⇒ **The gate has to steer around these directories too.**
`scripts/doc-lint.sh` reads that list from `.claude/doc-lint-exclude` at the project
root, one entry per line, **each with its reason written out**. An entry pointing at a
directory that does not exist, or one that excludes no file at all, goes red.

## Every number that enters a conclusion: was it measured, or guessed?

For every rate, capacity or ratio that enters a conclusion, first ask "was this measured or guessed?" — if guessed, go measure it.
What cannot be measured here (say, the fsync rate of an enterprise drive) is written as a conditional: "at 10⁵/s it is 89 years; that tier cannot be measured on this machine, so it stays an assumption" — never as an upper bound.

## How another project does it is a lead, not evidence

**Never treat another project's implementation as direct evidence.**
An external implementation may enter in exactly two places: **raising a hypothesis**,
and **pointing at which path to test**. It may not enter any of the three derivations
(forward / backward / cross-check), and it may not serve as the cross-check path. To become
evidence it must first be reproduced here as observable data; from then on you cite that
data, not that implementation.

**Separate the half you may cite from the half you may not**:

| May | May not |
|---|---|
| **Facts**: "the constant `XLOG_CONTINUE_TRANS` exists", "that function is spread over 93 sites in 22 files" — checkable, re-runnable | **Argument**: "XFS does it this way, therefore doing it this way is right" |
| **Counter-evidence**: a scheme that appears in **no** production implementation is a signal demanding an explanation | **Positive proof**: a scheme appearing in a production implementation does not make it fit for this project |
| **Mechanism**: decompose what they did into a mechanism, then argue that mechanism holds under this project's premises | **Transplanting**: carrying the conclusion over together with the premises it never wrote down |

⚠️ **"No production implementation takes path X" does not prove X is wrong, but the burden of
proof is on X's side**: to walk a path nobody has walked, you must say why nobody did.

⚠️ **When citing another implementation, you must also write down one known difference
between it and this project.** If you cannot name a difference, the citation is
**transplanting**, not argument.

**Cite it with its source and "not verified in this project"**
(`kb-discipline.md`, "Every entry carries its source and status").

## A hypothesis must be refutable by observation, or it is not a hypothesis

After writing a hypothesis down, **first ask "what phenomenon would overturn it".**
If you cannot answer, do not build an experiment on it.

**To rule a hypothesis out you need either a falsifiable observation or an explicit
statement that this is inference.** "I cannot think of another explanation" does not
constitute ruling out.

## Never pick the conclusion first and then build a model for it

**Ask these six before writing the conclusion down; if you cannot answer, you are not
done**:

| Ask | Cannot answer ⇒ conclusion came first |
|---|---|
| Did this model exist **before** my leaning did | You can say "when I built it I did not yet know which side it would come out on" |
| Did I build an equally serious arm for **the other side** | You can say what the opposing arm's best form is, and whether you measured it |
| Would I **accept** the opposite result | You can say "if the numbers came out the other way, here is how I would change the conclusion" — written down **before** the run |
| Did I take readings at **only one point** | A multiple derived from one size, one parameter, one workload is not "worst case" |
| Does the **same artifact** hold a counterexample to the range I quoted | You can show the range covers every sampled point in the artifact |
| Is this number a function of **some parameter that was never swept** | You can say which knobs in the apparatus this arm's cost / benefit depends on, and that each of them was swept; ask which parameter it is a function of before it goes into the body text |

⚠️ **Overturning an old conclusion must be held to a stricter standard than establishing one**,
not a looser one.

⚠️ **A straw-man opposing arm**: pick an implementation form for the other path that nobody
would actually adopt ("one record per entry" and the like), and present how badly it measures
as the cost of that path. **Criterion**: is the opposing arm's form **the one its own proponents would
recognize**.

⚠️ **When you cite an arm's number, carry its definition with it**
(the experiment-side form of `verify-before-claiming.md`'s
"whether it is settled and what it actually says are two different questions").

### An arm's definition is nailed down before the run too — what to do when a failure clause fires

What gets nailed down before a run is not only the criteria, the thresholds and the void
clauses — it is also **what each arm actually is**. When a failure clause hits a loosely
worded arm, do not "clarify" the arm into its strong reading and declare it the winner
without first recording a loss.

⇒ **Only three steps are admissible; skip one and it is a new experiment — re-run it**:

| Step | What it means |
|---|---|
| **Record the loss** | Write down the failure clause's verdict as it stands under the **weakest reading**: this arm did lose that cell, and to whom |
| **Tighten only** | The new form must be judged **at least as harshly by the original criteria**. Not one word looser |
| **State where it tightened** | Say what the new form **demands more of** than the old one. If you cannot say, it is a loosening |

⚠️ **When an arm's definition states both "how it is done" and "what that achieves", first show that the first implies
the second.** If it cannot be derived, delete "what that achieves", or change the method to one it can be derived from.

**Revising an arm or a criterion before the artifact runs is legitimate, but it must leave a record.** When a stronger
arm or a segment-granularity criterion is added once the unit tests have run but the artifact has not run even once,
state in the pre-run registration **what changed, which unit-test reading it rests on, and that the moment was before
the artifact**; keep the original criterion as written and judge the two arms each on its own.

### A criterion can be written wrong too: when it fires, first decide which kind it is

| Form | What it looks like | What to do |
|---|---|---|
| **The hit does not tell the arms apart** | One cell knocks every arm out at once; the cause lies in a premise all arms share | Do not use it to judge the arms: open a separate item for that shared premise and fix it first; judge the arms only on the cells that tell them apart. When writing a clause like "all out ⇒ fall back to one arm", also write how to record a cell that knocks every arm out |
| **The discriminator cannot be observed** | The criterion asks the system under judgement to tell two situations apart, but what that system can see at the moment it decides is identical, item for item, in both | Change the criterion — and a changed criterion is a new round |
| **The hit is filed under the wrong criterion** | A leg reports "criterion X fires", while what happened belongs, by the wording, to another criterion with a different threshold | Check word by word which clause of which criterion the hit satisfies |
| **The fix does not reach the cells that were hit** | A reverse-acceptance clause written before the run says "hit ⇒ choose between fix A and fix B", and one of the fixes is still hit on exactly the cells that were hit | When writing the clause, state for each fix which cell it repairs; after a hit, check each fix against the hit cells first, and when handing over one that is still hit, write "does not work on these cells" |

⇒ **When a criterion fires, ask four questions first**: does it tell the arms apart? Can the system under judgement
actually see, at that moment, what it is asked to tell apart? Which clause of the criterion's wording does it satisfy?
Is each fix the pre-run clause offers still hit on the cells that were hit? Only when all four pass, judge by the
clauses written before the run.

## Quote an artifact by copying the line whole

**Quote the artifact line whole, with its filename**; summarize only the one cell you
computed yourself.

### After a re-run, check the prose back against it — a green replay does not mean the prose is right

Re-run an artifact and every number the prose quotes from it has to be checked back, one by
one. **A green replay does not constitute "the numbers in the prose are right".**

⇒ The same governs **claims a single command could count**: "N unit tests", "M mutations",
"K lines of artifact". **A number a single command can count is counted by that command**,
not left to the writer to remember to come back and fix it.

⚠️ **Prose disagreeing with its artifact has another shape: the number is not wrong, the qualifier is gone.**
⇒ **Criterion**: when a conclusion says "only A buys it / unique to A / only A can", go to the
artifact and read **the non-A arms at every parameter setting**; if the table in the prose
**has no column for that parameter**, the scope has been shed and the "only" does not hold.

### An extreme found by a sweep that lands on the sweep's endpoint is not a measured number

Before reporting an extreme found by a sweep ("minimum viable value", "maximum safe value"), check whether it is an
endpoint of the swept range. If it is, write "≤ endpoint (lower edge of the sweep; boundary not measured)"; otherwise
widen the range and re-run until the extreme falls inside it.

## When you write a new criterion, record the entries to sweep and sweep them in a batch with the next stage sync

Once the criterion is written, ask: **what does it say about the entries already
registered?** No answer means you are not done writing it.

### Withdrawing a number or a conclusion also goes into the sweep list

With the object swapped for **a value or a clause that has been withdrawn or rewritten**, sweep the same way. The sweep itself is done in batches: each new criterion and each withdrawn number gets one line in the project's sweep list and is swept in a batch with the next stage sync, not dispatched one at a time.
**Criterion**: after withdrawing, ask "**who else in this repository still uses this
number**". You are done only when you can produce that list. The scope is the whole
repository, not "the places I remember".

⚠️ **Do not replace the list a sweep turns up wholesale: for each hit, first tell whether it states the present or reports what happened on one occasion.**
**Test: swap the new value into the sentence — is it still true?** If yes, it states the present, so change it. If no, it reports that occasion, so leave it.

⚠️ **Sweep by the old wording, not only by the new names.**
Before sweeping, write down "how the repository would describe this thing before it was done" (no X, X not yet, only Y, N items) and search the whole repository for those phrasings;
also take each current-state sentence this phase's diff deleted or rewrote and search, one by one, for whether it still lives elsewhere.

### A withdrawal's rationale collapsing does not bring the withdrawn conclusion back

When the rationale of a withdrawal later collapses, the withdrawn conclusion does not come back on its own: bringing it
back requires **arguing it again**.

⇒ **When you find a withdrawal's rationale has collapsed, ask three questions in order**:

| Ask | If you cannot answer, stop here |
|---|---|
| Which rationale is holding this slot up today | You can name the file and the passage it lives in |
| Does that rationale itself stand up | Check each of its supports; do not just read its conclusion sentence |
| Has the re-argument for reviving the withdrawn one actually been done | Not done is not done — "the rationale collapsed" is not a substitute |
