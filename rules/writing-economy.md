<!-- generated-from: rules/writing-economy.md sha256:fff87640a3434a992e351932068f15a20d7f6aad94c7eb505654ffeabbd4dd64 -->
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
