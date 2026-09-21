<!-- generated-from: rules/evidence-discipline.md sha256:aaf59b21df8a35619e911e0f023174744c7370b678e14ef4f089e53bfea78089 -->
<!-- doc-lint:rule-definition -->
# Every conclusion needs three derivations: forward, backward, cross-check

**Before any conclusion is stated, all three steps must be complete, and each one
spelled out.** Missing any of them makes it a hypothesis — write it as a hypothesis,
not as a conclusion.

| Step | What it asks | If you cannot do this, you did not do it |
|---|---|---|
| **Forward** | derive the conclusion from mechanism / code / data | you can name "which command, which lines of code, which data file" |
| **Backward** | **if this conclusion were false, what should I have seen?** Then go look | you can name a **specific observation, achievable this round**, and state that it did not appear |
| **Cross-check** | reproduce the same judgement by an independent route | the two routes **do not share** the same code, the same sample, or the same tool |

Backward is the step most often skipped, and it is the most valuable: forward can only
tell you "this explanation is consistent"; backward is what rules out "another
explanation is also consistent".

## The cross-check path must itself be shown to go red

"Reproduce the same judgement by a second, independent path" is not enough —
**that second path can itself be a decoration.**

When two paths agree, it may mean "both are right", or it may mean **the second one
was never looking**. Without fault injection you cannot tell these apart, and
**they look identical: both are agreement.**

**What to do**: inject a known fault into the subject under test — one the second path
*ought* to catch — and confirm that it does. If it does not, that cross-check is
decoration, and every earlier record of "the two paths agree" is void with it.

Example: the kernel's block-layer counters are used to cross-check the I/O counts a
program keeps for itself, and the two agree cell for cell. That agreement only becomes
evidence once you have also observed that **dropping direct I/O sends the block-layer
count to zero while the program's own count does not move at all** — otherwise the
agreement could just be the second path echoing the first.

⚠️ **An independent count that matches only the total does not confirm a breakdown.** The block-layer count agreeing with the total implied by the program's read pattern shows that both sides counted the same pile of requests,
not that "N of them are of this kind and M of that kind": the counter cannot tell which request belongs to which kind.
⇒ Before using a count to confirm a breakdown, ask whether the counter can distinguish the kinds being split. If it cannot, write only "consistent with", say where the breakdown comes from, and check the other raw count as well.

**This is the same discipline as "every verification must be able to fail", applied to
the cross-check path**: the main check must be shown to go red, and so must the
cross-check. **Showing only the former is doing half the work.**

## Background material fed into multi-party argumentation must itself be checked first

When cross-checking a conclusion across multiple independent sources, you typically
write up a "known facts" brief and hand it to each party. **Every external fact in
that brief must be checked by the person asking the question, first.**

The reason is mechanical: **every party takes it as a given, so an error in the brief
gets inherited by all of them together** — and they will still come back with an
"agreement". **Agreement therefore stops being evidence.**

Observed: a background brief stated that some production implementation "has no
such-and-such field". One of the three parties itself found the opposite in the
primary source — **and still used the brief's claim as the baseline for comparison.**
The eventual agreement was three parties jointly inheriting the same error.

**What to do**: before writing "implementation X works like this" into the brief,
check it yourself. What you cannot check, mark "unverified", so every party knows not
to treat that line as a given.

## Evidence kept verbatim must not be edited afterwards

The prompt you sent to a model, the raw output it produced, the log as it was written —
each of these corresponds one-to-one with an artifact. **Change one character and the
artifact no longer corresponds to its input**, and nothing on the outside shows it: the
file is still there, the date is still there, only the sentence "this output came from
this input" has quietly stopped being true.

So a prompt can only be edited together with a re-run. To add an explanation, open a
separate file; leave the original alone.

**The freeze holds for this round only; once the round is over, the archive is the version
control history.** Within a round, not one character of its evidence — prompts, the legs'
reports, raw output, stored artifacts — may change. Once that round is committed, the next
commit deletes **the batch the previous one left behind**; this round's own evidence stays,
so this round's verification still runs. To look up what was deleted, go read it in the
version control history.

⇒ **Old evidence is a reference, not a basis.** When an earlier conclusion is in doubt,
re-verify it rather than digging up the old evidence: old numbers are bound to the build
they were taken on, and whether they still hold today is answered only by running it today
(see "all old data is only a reference" in this same file). Overturning an earlier
conclusion means running the project's own inference discipline again, not confronting it
with an old file.

⇒ **A check on artifact lines quoted in prose judges "this change", not the whole repo.**
Numbers written into prose today must match today's artifacts verbatim; lines copied down
earlier are historical reference, their artifacts have been archived, and judging them now
means judging against a comparison that no longer exists.

⇒ **The gate has to steer around these directories too.** A check that demands someone
go back and edit the original forces one of two outcomes: the evidence chain breaks, or
the check gets bypassed wholesale — both worse than not checking at all.
`scripts/doc-lint.sh` reads that list from `.claude/doc-lint-exclude` at the project
root, one entry per line, **each with its reason written out**. An entry pointing at a
directory that does not exist, or one that excludes no file at all, goes red: an
exclusion that does nothing leaves people believing those files are already steered
around.

## Every number that enters a conclusion: was it measured, or guessed?

An upper bound nobody measured gets treated as fact by the reasoning downstream, and the factor by which it is wrong multiplies straight into the conclusion.

So for every rate, capacity or ratio that enters a conclusion, first ask "was this measured or guessed?" — if guessed, go measure it. What cannot be measured here (say, the fsync rate of an enterprise drive) is written as a conditional: "at 10⁵/s it is 89 years; that tier cannot be measured on this machine, so it stays an assumption" — never as an upper bound.

## How another project does it is a lead, not evidence

**Never treat another project's implementation as direct evidence.** "XFS does it this
way", "there is precedent in the kernel" proves only that someone has done it — not that
doing it here is right.

In between sits a whole set of unchecked premises: its workload, its on-disk format, its
concurrency model, its compatibility baggage, the problem it was solving at the time.
Miss on any one of them and the conclusion does not carry over. And those premises **are
not cited along with the conclusion**; they are usually not written down in its code
either, so reading the source will not hand them to you.

⇒ An external implementation may enter in exactly two places: **raising a hypothesis**,
and **pointing at which path to test**. It may not enter any of the three derivations
(forward / backward / cross-check), and it may not serve as the cross-check path — it is
not an independent path, it never touched this project's subject at all. To become
evidence it must first be reproduced here as observable data; from then on you cite that
data, not that implementation.

**Separate the half you may cite from the half you may not**:

| May | May not |
|---|---|
| **Facts**: "the constant `XLOG_CONTINUE_TRANS` exists", "that function is spread over 93 sites in 22 files" — checkable, re-runnable | **Argument**: "XFS does it this way, therefore doing it this way is right" |
| **Counter-evidence**: a scheme that appears in **no** production implementation is a signal demanding an explanation | **Positive proof**: a scheme appearing in a production implementation does not make it fit for this project |
| **Mechanism**: decompose what they did into a mechanism, then argue that mechanism holds under this project's premises | **Transplanting**: carrying the conclusion over together with the premises it never wrote down |

⚠️ **"No production implementation takes path X" is one of the most valuable external
signals this project has** — it does not prove X is wrong, but it shifts the burden of
proof onto X's side: **to walk a path nobody has walked, you must say why nobody did.**

⚠️ **When citing another implementation, you must also write down one known difference
between it and this project.** If you cannot name a difference, you have not yet
understood why that approach holds over there; the citation at that point is
**transplanting**, not argument.

**Cite it with its source and "not verified in this project"**
(`kb-discipline.md`, "Every entry carries its source and status").

## A hypothesis must be refutable by observation, or it is not a hypothesis

After writing a hypothesis down, **first ask "what phenomenon would overturn it".**
If you cannot answer, it has not taken shape yet — do not build an experiment on it.

**To rule a hypothesis out you need either a falsifiable observation or an explicit
statement that this is inference.** "I cannot think of another explanation" does not
constitute ruling out.

## Never pick the conclusion first and then build a model for it

**Choosing the conclusion first and then building a model that yields it is the number
one form this discipline exists to stop.** It does not show up as "fabricated data"; it
shows up as **every step being reasonable, while those steps were selected by the
conclusion**: which parameter, how deep to model, who serves as the opposing arm, which
dimension to ignore — every one of them is a choice, and people only remember the most
defensible of the choices they made.

**Criteria (ask before writing the conclusion down; if you cannot answer, you are not
done)**:

| Ask | Cannot answer ⇒ conclusion came first |
|---|---|
| Did this model exist **before** my leaning did | You can say "when I built it I did not yet know which side it would come out on" |
| Did I build an equally serious arm for **the other side** | You can say what the opposing arm's best form is, and whether you measured it |
| Would I **accept** the opposite result | You can say "if the numbers came out the other way, here is how I would change the conclusion" — written down **before** the run |
| Did I take readings at **only one point** | A multiple derived from one size, one parameter, one workload is not "worst case" |
| Does the **same artifact** hold a counterexample to the range I quoted | You can show the range covers every sampled point in the artifact — picking three points to report "1.25–1.5" while a 1.000 sits in that very same output |
| Is this number a function of **some parameter that was never swept** | You can say which knobs in the apparatus this arm's cost / benefit depends on, and that each of them was swept — before an arm's cost goes into the body text, ask "which parameter is it a function of" |

⚠️ **The moment of greatest danger is "overturning an old conclusion"**: by then you
already have a new direction, and the thrill of overturning tilts every choice in the new
model that way. **Overturning must be held to a stricter standard than establishing**,
not a looser one.

⚠️ **A straw-man opposing arm is this failure's signature product**: pick an
implementation form for the other path that nobody would actually adopt ("one record per
entry" and the like), measure how badly it does, then present that as the cost of that
path. **Criterion**: is the opposing arm's form **the one its own proponents would
recognize**.

⚠️ **When you cite an arm's number, carry its definition with it.** The arm is defined in
the experiment write-up, the verdict takes only the number: same failure, one step
downstream. The number is right; what is wrong is that it measures something other than
what the verdict is asking. (The experiment-side form of `verify-before-claiming.md`'s
"whether it is settled and what it actually says are two different questions".)

### An arm's definition is nailed down before the run too — what to do when a failure clause fires

What gets nailed down before a run is not only the criteria, the thresholds and the void
clauses — it is also **what each arm actually is**. Leave an arm loosely worded and the
attacking leg will hit **its weakest reading**, while the person who wrote it had a different
reading in mind. "Clarifying" the arm into the strong reading and then declaring it the
winner changes no criterion on paper, and is post-hoc modelling in substance.

⇒ **Only three steps are admissible; skip one and it is a new experiment — re-run it**:

| Step | What it means |
|---|---|
| **Record the loss** | Write down the failure clause's verdict as it stands under the **weakest reading**: this arm did lose that cell, and to whom |
| **Tighten only** | The new form must be judged **at least as harshly by the original criteria**. Not one word looser |
| **State where it tightened** | Say what the new form **demands more of** than the old one. If you cannot say, it is a loosening |

⚠️ **The first step is the one people skip**, and once it is skipped an honest tightening
and a "loosen it, then declare victory" **read identically on the page**.

⚠️ **When an arm's definition states both "how it is done" and "what that achieves", first show that the first implies
the second.** Read separately, both sentences make sense; together they can contradict each other. Each leg then judges
by one of the two sentences and reaches the opposite verdict, and the author of the definition cannot see that they have
in fact written two arms.


⇒ When writing an arm, treat "what that achieves" as a proposition to be proven: if it cannot be derived, delete it, or
change the method to one it can be derived from.

**Revising an arm or a criterion before the artifact runs is legitimate, but it must leave a record.** Seeing where the
first arm is weak once the unit tests have run but the artifact has not run even once, and adding a stronger arm or a
segment-granularity criterion, is a different thing from "changing it after seeing the artifact" — the former has no
number in any cell yet. The legitimate way: state in the pre-run registration **what changed, which unit-test reading it
rests on, and that the moment was before the artifact**; keep the original criterion as written and judge the two arms
each on its own. Without that passage in the registration, nobody can tell afterwards whether it was added after seeing
the numbers.

### A criterion can be written wrong too: when it fires, first decide which kind it is

A criterion nailed down before the run stops "change the criterion after seeing the result", but the criterion itself
can be wrong, and that usually shows only at the moment it fires. Four forms:

| Form | What it looks like | What to do |
|---|---|---|
| **The hit does not tell the arms apart** | One cell knocks every arm out at once; the cause lies in a premise all arms share | Do not use it to judge the arms: open a separate item for that shared premise and fix it first; judge the arms only on the cells that tell them apart. When writing a clause like "all out ⇒ fall back to one arm", also write how to record a cell that knocks every arm out |
| **The discriminator cannot be observed** | The criterion asks the system under judgement to tell two situations apart, but what that system can see at the moment it decides is identical, item for item, in both | Two criteria that demand different outcomes for those two situations contradict each other; any design can satisfy only one. Change the criterion — and a changed criterion is a new round |
| **The hit is filed under the wrong criterion** | A leg reports "criterion X fires", while what happened belongs, by the wording, to another criterion with a different threshold | Check word by word which clause of which criterion the hit satisfies; filing it wrong knocks an arm out under the wrong threshold |
| **The fix does not reach the cells that were hit** | A reverse-acceptance clause written before the run says "hit ⇒ choose between fix A and fix B", and one of the fixes is still hit on exactly the cells that were hit | When writing the clause, state for each fix which cell it repairs; after a hit, check each fix against the hit cells first, and when handing over one that is still hit, write "does not work on these cells" — leave that out and choosing it keeps the misjudgement in place |



⇒ **When a criterion fires, ask four questions first**: does it tell the arms apart? Can the system under judgement
actually see, at that moment, what it is asked to tell apart? Which clause of the criterion's wording does it satisfy?
Is each fix the pre-run clause offers still hit on the cells that were hit? Only when all four pass, judge by the
clauses written before the run.

## Quote an artifact by copying the line whole

Paraphrase drifts, and it drifts one way — each retelling leans a little further toward
what you wanted, and every single step still looks faithful. Measured: the same
difference out of one artifact was misstated four rounds running — "identical" → "identical
cell by cell" (three of the four cells were not) → "slightly less" (wrong direction) →
"exactly 2 nodes" (extrapolating one saturated cell into a range). Nobody caught it.

⇒ **Quote the artifact line whole, with its filename**; summarize only the one cell you
computed yourself. The fix is not "be more careful" — care leaves no trace, copying does.

### After a re-run, check the prose back against it — a green replay does not mean the prose is right

Re-run an artifact and every number the prose quotes from it has to be checked back, one by
one. **A green replay does not constitute "the numbers in the prose are right"**: the replay
pins which range the conclusion lands in, and the wider that range, the further the prose can
drift inside it — and it drifts toward the number the writer happened to remember.


⇒ The same governs **claims a single command could count**: "N unit tests", "M mutations",
"K lines of artifact". They look decorative; they are in fact the measurement basis of how
strong the evidence is, and nothing says a word when they drift. Five such claims were found
disagreeing with the source or the artifact in one repository on one day (unit-test count off
by 1, mutation count off by 3, line counts off by 6 and 9). **A number a single command can
count should be counted by that command**, not left to the writer to remember to come back
and fix it.

⚠️ **Prose disagreeing with its artifact has a second shape, harder to catch than a drifting
number: the number is not wrong, the qualifier is gone.** A drifting number at least leaves you
two numbers to lay side by side; a missing qualifier **leaves nothing to compare against** —
every number at the citing site is true, and what is wrong is that the conclusion claims a wider
range than the artifact supports.


⇒ **Criterion**: when a conclusion says "only A buys it / unique to A / only A can", go to the
artifact and read **the non-A arms at every parameter setting**; if the table in the prose
**has no column for that parameter**, the scope has been shed and the "only" does not hold.

### An extreme found by a sweep that lands on the sweep's endpoint is not a measured number

When you sweep a parameter and report a "minimum viable value" or a "maximum safe value", and that value is exactly an
endpoint of the swept range, it only says "viable up to the endpoint"; the real boundary lies outside the range and was
not measured. Writing "the minimum viable value is X" turns the starting point of the sweep into a measurement.


⇒ Before reporting an extreme found by a sweep, check whether it is an endpoint. If it is, write "≤ endpoint (lower edge
of the sweep; boundary not measured)"; otherwise widen the range and re-run until the extreme falls inside it.

## When you write a new criterion, sweep it back over the entries already on the books

**A criterion applied to half the cases will be used by the next person on the lenient
half.** Measured: "no invariant may enter whose gate has no input" kept a new invariant
out, while four already-registered invariants in the same family had no input either —
the author swung the ruler only at the newcomer.

⇒ Once the criterion is written, ask: **what does it say about the entries already
registered?** No answer means you are not done writing it.

### Withdrawing a number or a conclusion also means sweeping for who cites it

Same discipline, with the object swapped for a value or a clause that has been withdrawn
or rewritten. It hides better than "A new criterion must be swept back over the entries already
on the books": whoever withdrew it usually did sweep
a few places, so they have every reason to believe they finished, and the one they missed
does not surface for days.

**Criterion**: after withdrawing, ask "**who else in this repository still uses this
number**". You are done only when you can produce that list. The scope is the whole
repository, not "the places I remember" — memory hands you exactly the easy ones.


⇒ This and "background material fed to a multi-party argument must itself be checked
first" are two ends of one hole: at one end the citer did not re-check, at the other the
withdrawer did not sweep clean. **Both ends have to be plugged.**

⚠️ **Do not replace the list a sweep turns up wholesale: for each hit, first tell whether it states the present or reports what happened on one occasion.**
The same old value in a repository is often half "it is N now" and half "on that occasion it became N", "written as X from run seven on" —
the latter stays true, and replacing it turns a true sentence false.

**Test: swap the new value into the sentence — is it still true?** If yes, it states the present, so change it. If no, it reports that occasion, so leave it.

⚠️ **Sweep by the old wording, not only by the new names.** A stale sentence is written in the old wording ("only the first transaction", "no second instance yet");
searching by the names of what this phase newly built mostly turns up sentences that have already been fixed.
The ready-made list of old wordings is this phase's own diff: current-state sentences it deleted or rewrote in one place often live on unchanged elsewhere.

⇒ Before sweeping, write down "how the repository would describe this thing before it was done" (no X, X not yet, only Y, N items) and search the whole repository for those phrasings;
also take each current-state sentence this phase's diff deleted and search, one by one, for whether it still lives elsewhere.

### A withdrawal's rationale collapsing does not bring the withdrawn conclusion back

This is the mirror image of "Withdrawing a number or a conclusion also means sweeping for who
cites it", and it is easier to fall for, because it
looks like you are correcting an error.

When a conclusion is withdrawn, the rationale written down at the time is often just the
one that was easiest to write. Days later that rationale collapses on its own — a premise
changed, or a decision it depended on was rewritten — and it is tempting to conclude "then
the withdrawal no longer holds, the original should come back." **It does not follow.** A
withdrawal is a verdict; its rationale collapsing only means that verdict lost its
grounds, not that the opposite is true. Bringing the original back requires **arguing it
again**, and that argument may land the other way.

**There is a more valuable step still**: after a withdrawal, that slot usually already has
a different rationale holding it up. Staring only at the withdrawn one makes you miss the
replacement entirely — and the replacement is what is actually load-bearing today.

⇒ **When you find a withdrawal's rationale has collapsed, ask three questions in order**:

| Ask | If you cannot answer, stop here |
|---|---|
| Which rationale is holding this slot up today | You can name the file and the passage it lives in |
| Does that rationale itself stand up | Check each of its supports; do not just read its conclusion sentence |
| Has the re-argument for reviving the withdrawn one actually been done | Not done is not done — "the rationale collapsed" is not a substitute |

⇒ None of the three has anything to do with whether the exclusivity still holds.
**Following "the premise is gone, so the withdrawal is void" touches none of them.**

## The backward-reasoning gap specific to filesystems

Most quantities measured in this project are **binary** — right or wrong. The risk is
not measurement precision, it is coverage:

- **"All tests green" does not mean "the implementation is correct".** The crash
  window may be one write wide; not hitting it may only mean that crash point was
  never enumerated. To claim "crash consistency holds", state whether this round
  **could have hit it at all** — how many crash points were enumerated, and whether
  that was all of them.
- **"The checker reported nothing" does not mean "the image is good".** The checker
  may simply not have implemented that check yet. Before saying it, look at the
  implementation status of the corresponding entry in `kb/invariants.md`.
