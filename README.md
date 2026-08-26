# singlefs-ai-sop-en

**English edition of the contributor governance SOP.**

**It governs "how a project collaborates with AI"** — what evidence a submission must
carry, how to write documents that retrieval will not misread, where decisions are
recorded, what a gate must show when it refuses. None of this depends on what kind of
system is being built.

**It does not govern how any particular kind of system should be designed.** A
discipline only one kind of project needs (say, a filesystem's "start from the
transaction") belongs to that project, in its own `.claude/rules/`.

> **AI-friendly to implement, human-friendly to review.**

## Pick the edition in your own language

**Developers pick the edition in their own language.** Chinese, English and Japanese
exist today, more are in progress, and **English is the default**. Install one and work
in it directly — someone working in English never has to read Chinese.
**The AI does not have to translate a second time either**: the repository it works in
decides the language of collaboration. One fewer restatement is one less loss of
information. That is the whole reason the editions exist.

In `I18N`, `this=` is this repository's language, `default=en` is the default edition,
and `languages=` lists what has been published. The manifest and the gate scripts live
in the Chinese repository.

**One contribution covers every published edition.** Whatever language a contributor
writes in, a submission changes all of them and they are merged together. A submission
carrying only the contributor's own language is not accepted — it pushes the cost of
"the other languages now hold stale rules" onto someone else. **Whether the editions
say the same thing is guaranteed by the contributor**; all the gate can see is a hash
mismatch.

Where the editions read differently, the language does not decide — **the actual
behaviour of `scripts/` does**, being the one unambiguous statement of these rules.

**Current state: the editions other than Chinese are machine-translated**, to get them
out quickly. A precise translation will be made before the rules are formalised.

If one edition reads badly (drifted wording, inconsistent terminology), fix that
edition — **and go back and check whether the other two are unclear as well.** Each
language exposes a different ambiguity; that is a side benefit of keeping three, so do
not waste it.

Terminology comes from [GLOSSARY.md](GLOSSARY.md), shared across all three languages.

## The bar for changing it

**Once settled, this SOP should barely move.** It is the shared floor under every
project that hooks into it, so changing one line changes every project's floor at once.
Every such judgement is made by people — the SOP is a standard, not something under
test. The real signal is **how often it changes**: frequent change means the design is
wrong, not that it is improving.

**Work does not happen here.** Day-to-day work lives in the projects that hook in.
`0.x` means not yet settled; after `1.0.0`, changes to `rules/` should be rare.

## Staying in sync

`SOURCE-MANIFEST.sha256` is this repository's copy of the manifest as of the last time
it was brought in step. Comparing it against the current `MANIFEST.sha256` in the
Chinese repository — where the manifest and the scripts live — shows which files have
fallen behind. The manifest covers `CLAUDE.md` and `rules/*.md` —
`CLAUDE.md` is the body of the rules and also fixes the language of collaboration, so
leaving it out would let the three editions quietly say different things. `VERSION` is identical across all three repositories;
one bump bumps all three.

## Install

```bash
git clone https://github.com/faliye/singlefs-ai-sop-install
bash singlefs-ai-sop-install/sop-install.sh --lang en
```

The installer's own prompts follow the language you choose.

## Licence

Dual-licensed: [Apache-2.0](LICENSE-APACHE) or [MIT](LICENSE-MIT), at your option.
