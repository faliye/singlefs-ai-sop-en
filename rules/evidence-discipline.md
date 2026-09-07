<!-- generated-from: rules/evidence-discipline.md sha256:68e528bc661a84c34fd4c022d3db500213b50c05e14ee8c0a3473cc6fd6ffc55 -->
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

⇒ **The gate has to steer around these directories too.** A check that demands someone
go back and edit the original forces one of two outcomes: the evidence chain breaks, or
the check gets bypassed wholesale — both worse than not checking at all.
`scripts/doc-lint.sh` reads that list from `.claude/doc-lint-exclude` at the project
root, one entry per line, **each with its reason written out**. An entry pointing at a
directory that does not exist, or one that excludes no file at all, goes red: an
exclusion that does nothing leaves people believing those files are already steered
around.

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

Measured (2026-09-06): an experiment's prose said "durable semantics is 1.475× slower",
"durable ÷ nosync = 22.33×", "the injected 5 ms came back as 5 123 506 ns", while its own
kept artifact says, verbatim, 1.625× / 30.36× / 5 199 857 ns. All three sit inside the
replay's range assertions ([1.15, 2.10], [10, 500], [4.5 ms, 5.5 ms]) — **the gate was all
green**.

⇒ The same governs **claims a single command could count**: "N unit tests", "M mutations",
"K lines of artifact". They look decorative; they are in fact the measurement basis of how
strong the evidence is, and nothing says a word when they drift. Five such claims were found
disagreeing with the source or the artifact in one repository on one day (unit-test count off
by 1, mutation count off by 3, line counts off by 6 and 9). **A number a single command can
count should be counted by that command**, not left to the writer to remember to come back
and fix it.

## When you write a new criterion, sweep it back over the entries already on the books

**A criterion applied to half the cases will be used by the next person on the lenient
half.** Measured: "no invariant may enter whose gate has no input" kept a new invariant
out, while four already-registered invariants in the same family had no input either —
the author swung the ruler only at the newcomer.

⇒ Once the criterion is written, ask: **what does it say about the entries already
registered?** No answer means you are not done writing it.

### Withdrawing a number or a conclusion also means sweeping for who cites it

Same discipline, with the object swapped for a value or a clause that has been withdrawn
or rewritten. It hides better than the case above: whoever withdrew it usually did sweep
a few places, so they have every reason to believe they finished, and the one they missed
does not surface for days.

**Criterion**: after withdrawing, ask "**who else in this repository still uses this
number**". You are done only when you can produce that list. The scope is the whole
repository, not "the places I remember" — memory hands you exactly the easy ones.

Measured (2026-09-06): when the width of a location entry was rewritten, the decision text
itself said "this item settles a number that never went through the decision process and
had already been consumed", and it named and swept **two** downstream derived numbers.
Three days later a third citation in another document still carried the old value, was
copied into the background material of a three-way argument, and **all three legs
inherited it** — every argument resting on that number was voided for the whole round.
The one who withdrew it swept two places, missed one, and did not know it.

⇒ This and "background material fed to a multi-party argument must itself be checked
first" are two ends of one hole: at one end the citer did not re-check, at the other the
withdrawer did not sweep clean. **Both ends have to be plugged.**

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
