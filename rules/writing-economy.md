<!-- generated-from: rules/writing-economy.md sha256:2af558e3574ccb016b80d85981c0cfd214604dcf922356530adb23d889807e48 -->
<!-- doc-lint:rule-definition -->
# Explanation length must match the weight of the change

**The ruler: for a change of a few to a few dozen lines, the commit body caps at two
paragraphs and new code comments cap at one line.** If it will not fit, first suspect
you are writing something that should not be written.

**Cutting it down to size is the job of whoever wrote the code, not of whoever
reviews it.** However solid the argument, that is not a reason to write all of it out
— the paragraph you feel is "load-bearing" is often exactly the one to cut.

## As short as possible without losing content

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

## Hard data does not count against the ruler

Core measurements provided for review do not count toward the length: a reviewer can
take them and verify for themselves. They are evidence, not "please take my word".

| Does not count (include it) | Still counts (cut it) |
|---|---|
| numbers: hit/miss counts, before-and-after, timings | why this number matters and what it shows |
| the reproduction command and its verbatim output | how I came to think of running that command |
| the basis needed to re-run: version, config, hardware | the history of the experiment, which dead ends were tried |

**Include only the few decisive numbers**; the full record stays in `kb/` or `records/`.

## Cut these categories

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
