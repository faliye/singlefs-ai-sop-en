<!-- generated-from: rules/writing-discipline.md sha256:805c4dce4880e2b75b4fe5cf27182dfdc4f792c88740d442f852375d0c695904 -->
<!-- doc-lint:rule-definition -->
# Writing discipline

This covers everything written for people to read: rules, README, design notes, the kb,
commit messages, code comments, gate failure messages. Three things: first work out who
it is for, make the length match the weight of the change, and write plainly.

## First work out who this is for

**Mixing them serves neither.** Three kinds of document, three ways of writing:

| Kind | Who uses it | Goal | Rule |
|---|---|---|---|
| **Design docs** | humans read them through | make people **agree** and **get started** | `design-doc-discipline.md` |
| **Engineering kb** | model retrieval | keep the model from **making things up** | `kb-discipline.md` |
| **Rules** | execution | must be turnable into a check that fails | `rules-discipline.md` |

Before writing, ask: will this be read start to finish by a person, or pulled out
one item at a time by a model? Different answers, different writing.

**The one rule common to all three**: body text states only the current state;
history goes to the end. But the *reason* differs for each kind, and is stated in
each rule.

## Length must match the weight of the change

**The ruler: for a change of a few to a few dozen lines, the commit body caps at two
paragraphs and new code comments cap at one line.** If it will not fit, first suspect
you are writing something that should not be written.

**Cutting it down to size is the job of whoever wrote the code, not of whoever
reviews it.** However solid the argument, that is not a reason to write all of it out
— the paragraph you feel is "load-bearing" is often exactly the one to cut.

### As short as possible without losing content

**The test: if this sentence goes, what does the reader no longer know?** If you cannot
name what was lost, cut it — "it reads more smoothly" and "it looks more thorough" are
not content.

The cost of padding is not the space it takes, it is **dilution**: readers and retrieving
models alike have to pick the one useful sentence out of the filler, and the odds of
missing it rise with length. **Every padded sentence downweights the real content.**

The three most common are none of them "wrote something new": restating a point already
made in different words, prefacing an unchallenged conclusion with a run-up, and writing
the same criterion once positively and once negatively.

**But brevity may not be bought with content** — evidence, the basis behind a number, and
the next step (`sop-first.md`'s `howto`) may none of them be dropped. What to keep is in
"Hard data does not count against the ruler" and "Cut these categories".

### "Shorter is better" is not "shorter is righter"

The first phrase carries a precondition (nothing is lost), **and the precondition is the
first thing to fall off in transmission** — what remains, "shorter is better", gets taken
as an instruction executable on its own, and every deletion arrives with its defence
pre-written.

**The two directions of error are asymmetric, so when in doubt, keep it**: verbosity
demotes the content, and the reader still finds it after a few more seconds; over-cutting
makes content **disappear**, and the reader does not know what they are missing, so they
cannot ask for it back.

Three forms of going too far, each defensible as "well, it is shorter":

| Cut down to | What was lost |
|---|---|
| "measured 3.26×" | **The basis**: what hardware, what workload, what block size. A number without its basis cannot be re-checked, which is the same as not having measured |
| "rejected" | **The next step**: `sop-first.md` requires every rejection to carry a `howto`. Dropping it is shorter, and turns the gate back into a sieve |
| "per D22 (how unit atomicity composes)" | **The grounds**: why it was settled that way. Three months on, nobody knows what that sentence rests on |

**The reverse test pairs with the forward test; a sentence must pass both**:

- Forward: if I **delete** this sentence, what does the reader know less? Cannot say → delete it.
- Reverse: if I **keep** this sentence, what does the reader know more? Can say → it stays, however short you wanted to be.

### Hard data does not count against the ruler

Core measurements provided for review do not count toward the length: a reviewer can
take them and verify for themselves. They are evidence, not "please take my word".

| Does not count (include it) | Still counts (cut it) |
|---|---|
| numbers: hit/miss counts, before-and-after, timings | why this number matters and what it shows |
| the reproduction command and its verbatim output | how I came to think of running that command |
| the basis needed to re-run: version, config, hardware | the history of the experiment, which dead ends were tried |

**Include only the few decisive numbers**; the full record stays in `kb/` or `records/`.

### Cut these categories

1. **Subjective assessment.** Reporting your impression is not an argument; replace it
   with a checkable fact.
2. **Content duplicated between the body and the kb.** Conclusions in the body,
   verification method in the kb.
3. **Circumstantial evidence.** Evidence requiring the reader to supply a reasoning
   step; replace it with the kind that is the conclusion directly.
4. **The full derivation chain.** How you found it step by step is not something the
   reader needs to walk again.
5. **Background exposition in code comments.** A comment says what this line is; it
   does not carry an argument.
6. **Defending the history.** A commit says only **what changed**; the discussion is
   in the previous round's record.
7. **Quoted source code.** The reader has the source tree. Same for logs and dumps:
   state the conclusion. The exception is decisive evidence the reader could not
   verify without it.

## Write plainly

**Write in plain modern English. Say it straight; don't circle around it.**

### Four things not to write

| Don't | What it looks like | Instead |
|---|---|---|
| **Bureaucratic padding** | "in the event that", "with respect to", "it is the case that", "perform an analysis of", "make use of" | "if", "about", drop it, "analyse", "use" |
| **Archaic or literary register** | "hereinafter", "thus it follows", "whereupon", "one might posit" | Say what it is and what follows |
| **Concessions that concede nothing** | "While X, however Y" where X and Y do not actually conflict | Keep only the half you mean |
| **Over-explaining** | "as is well known", "needless to say", "it goes without saying"; stating a conclusion and then restating its negation | Say it once |

### Write conjunctions as conjunctions; unpack noun strings into sentences

Symbols and strings of nouns are not sentences. Replacing conjunctions with `⇒`, `×`, `+` and compressing an action into a noun string like "verbatim verification of the four mandatory sites" saves characters, but the reader has to rebuild the sentence in their head first; and a noun string needs no subject and no causation, so the writer can dodge "who did what, and why does it hold" — stiff phrasing covers for gaps in the logic.

- Where "so / because / then / but" belongs, write the word, not `⇒`; inside a table cell, where space is tight, symbols may stay.
- Unpack noun strings: "main agent additionally verbatim-verifies four mandatory sites" becomes "I also checked the four mandatory sites word for word."
- Keep bold for verdicts and numbers; do not bold whole sentences or paragraphs — a page of bold has no emphasis left.
- Label-style colons ("mechanism:", "basis:", "measurement:") carry structure in a kb and may stay, but a complete sentence must follow the colon.

This does not conflict with "as short as possible": what gets cut is padding, not the skeleton of the sentence. Natural is not verbose.

The gate does not check this. The `⇒` already in `rules/` have not been swept yet; do not take them as examples.

### The test

**Read it out loud.** Would you say this sentence to a colleague? If not, rewrite it.

Three concrete questions:

- Do people use this word when talking? If not, swap it.
- Is this sentence explaining the previous one? If the previous one was already clear,
  delete this one.
- Does this "however" actually reverse anything? If the two halves agree, drop it.

### Don't overcorrect

**Plain does not mean vague.** Terms, measurement bases, numbers and commands stay
exact — "block size 16 KiB" must not become "blocks aren't big" in the name of
readability.

**Plain does not mean opinion-free either.** "This shouldn't be settled yet", "this
approach is wrong" — write exactly that. Saying it straight includes **stating the
conclusion first**.

### Which half the gate handles

`scripts/doc-lint.sh` checks **the fixed phrasings in a word list** — the common
padding and archaic constructions. A hit turns red and suggests the replacement. The rules
proper (`rules/*.md`, `CLAUDE.md`, skill bodies) are checked the same way: the
`<!-- doc-lint:rule-definition -->` at the head of a file exempts only the examples quoted in
backticks and 「」; the body is checked as usual.

**What it cannot check**: whether a sentence flows, whether a concession is redundant,
whether an explanation is bloated. Those need a person to read it aloud. So a green
word list does not mean this rule is being kept; it only means the most obvious traps
were avoided.

The word list is Chinese. In this edition the check reports **unimplemented** rather
than passing silently — inventing an English word list would add false positives, not
remove them (`show-me-test.md`: the gate must not pretend to pass).
