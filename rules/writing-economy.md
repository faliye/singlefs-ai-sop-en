<!-- generated-from: rules/writing-economy.md sha256:ed4df1ff3b078619e5ea6e227fe8320190e3d373c099067c1c43b82914d5ec4c -->
<!-- doc-lint:rule-definition -->
# Explanation length must match the weight of the change

**The ruler: for a change of a few to a few dozen lines, the commit body caps at two
paragraphs and new code comments cap at one line.** If it will not fit, first suspect
you are writing something that should not be written.

**Cutting it down to size is the job of whoever wrote the code, not of whoever
reviews it.** However solid the argument, that is not a reason to write all of it out
— the paragraph you feel is "load-bearing" is often exactly the one to cut.

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
