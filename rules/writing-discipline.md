<!-- generated-from: rules/writing-discipline.md sha256:3d77034abe0130dfbca37037e11a7ed1b92635b8329ae403df04cea4ffe3cd4a -->
<!-- doc-lint:rule-definition -->
# Writing discipline

This covers everything written for people to read: rules, README, design notes, the kb,
commit messages, code comments, gate failure messages. Three things: first work out who
it is for, make the length match the weight of the change, and write plainly.

## First work out who this is for

Three kinds of document, three ways of writing:

| Kind | Who uses it | Goal | Rule |
|---|---|---|---|
| **Design docs** | humans read them through | make people **agree** and **get started** | `design-doc-discipline.md` |
| **Engineering kb** | model retrieval | keep the model from **making things up** | `kb-discipline.md` |
| **Rules** | execution | must be turnable into a check that fails | `rules-discipline.md` |

Before writing, ask: will this be read start to finish by a person, or pulled out
one item at a time by a model?

**The one rule common to all three**: body text states only the current state;
history goes to the end.

## Length must match the weight of the change

**The ruler: for a change of a few to a few dozen lines, the commit body caps at two
paragraphs and new code comments cap at one line.** If it will not fit, first check whether
you are writing something that should not be written.

**Cutting it down to size is the job of whoever wrote the code, not of whoever
reviews it.** The paragraph you feel is "load-bearing" is often exactly the one to cut.

### As short as possible without losing content

**The test: if this sentence goes, what does the reader no longer know?** If you cannot
name what was lost, cut it — "it reads more smoothly" and "it looks more thorough" are
not content.

Cut these three outright: restating a point already
made in different words, prefacing an unchallenged conclusion with a run-up, and writing
the same criterion once positively and once negatively.

**But brevity may not be bought with content** — evidence, the basis behind a number, and
the next step (`sop-first.md`'s `howto`) may none of them be dropped. What to keep is in two sections:
"'Shorter is better' is not 'shorter is righter'" and "Hard data does not count against the ruler".

### "Shorter is better" is not "shorter is righter"

When in doubt, keep it.

These three are cut too far:

| Cut down to | What was lost |
|---|---|
| "measured 3.26×" | **The basis**: what hardware, what workload, what block size |
| "rejected" | **The next step**: `sop-first.md` requires every rejection to carry a `howto` |
| "per that one decision" | **The grounds**: why it was settled that way |

**The reverse test pairs with the forward test; a sentence must pass both**:

- Forward: if I **delete** this sentence, what does the reader know less? Cannot say → delete it.
- Reverse: if I **keep** this sentence, what does the reader know more? Can say → it stays, however short you wanted to be.

### Hard data does not count against the ruler

Core measurements provided for review do not count toward the length: this covers only what a
reviewer can take and verify for themselves.

| Does not count (include it) | Still counts (cut it) |
|---|---|
| numbers: hit/miss counts, before-and-after, timings | why this number matters and what it shows |
| the reproduction command and its verbatim output | how I came to think of running that command |
| the basis needed to re-run: version, config, hardware | which dead ends were tried |

**Include only the few decisive numbers**; the full record stays in `kb/` or `records/`.

### Cut these categories

1. **Subjective assessment.** Replace it with a checkable fact.
2. **Content duplicated between the body and the kb.** Conclusions in the body,
   verification method in the kb.
3. **Circumstantial evidence.** Evidence requiring the reader to supply a reasoning
   step; replace it with the kind that is the conclusion directly.
4. **The full derivation chain.** How you found it step by step: leave it out.
5. **Background exposition in code comments.** A comment says what this line is; it
   does not carry an argument.
6. **Defending the history.** A commit says only **what changed**; the discussion is
   in the previous round's record.
7. **Quoted source code.** Same for logs and dumps:
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

Do not replace conjunctions with `⇒`, `×`, `+`, and do not compress an action into a noun string like "verbatim verification of the four mandatory sites".

- Where "so / because / then / but" belongs, write the word, not `⇒`; inside a table cell, where space is tight, symbols may stay.
- Unpack noun strings: "main agent additionally verbatim-verifies four mandatory sites" becomes "I also checked the four mandatory sites word for word."
- Keep bold for verdicts and numbers; do not bold whole sentences or paragraphs.
- Label-style colons ("mechanism:", "basis:", "measurement:") may stay in a kb, but a complete sentence must follow the colon.

This does not conflict with "as short as possible": what gets cut is padding, not the skeleton of the sentence.

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
exact: "block size 16 KiB" must not become "blocks aren't big".

**Plain does not mean opinion-free either.** "This shouldn't be settled yet", "this
approach is wrong" — write exactly that, without a run-up before turning into it.

### Which half the gate handles

`scripts/doc-lint.sh` checks **the fixed phrasings in a word list** (the common
padding and archaic constructions). A hit turns red and suggests the replacement. The rules
proper (`rules/*.md`, `CLAUDE.md`, skill bodies) are checked the same way: the
`<!-- doc-lint:rule-definition -->` at the head of a file exempts only the examples quoted in
backticks and 「」; the body is checked as usual.

**What it cannot check**: whether a sentence flows, whether a concession is redundant,
whether an explanation is bloated; those need a person to read it aloud.
A green word list only means the traps in the word list were avoided.

The word list is Chinese and covers Chinese only. In the English and Japanese repositories this check
reports **unimplemented** explicitly rather than passing silently (`show-me-test.md`: the gate must not pretend to pass).
