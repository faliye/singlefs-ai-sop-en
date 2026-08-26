# singlefs-ai-sop-en

**English translation of the singlefs contributor governance SOP.**

> **AI-friendly to implement, human-friendly to review.**

## This repository is generated

**Do not edit the rules here to change a rule.** The blueprint is
[singlefs-ai-sop-zh](https://github.com/faliye/singlefs-ai-sop-zh); change it there
and regenerate all languages.

"Blueprint" means only *where changes originate* — Chinese is used for efficiency and
precision of expression, not because it is more authoritative. **The three texts have
equal standing.** Where they read differently on a rule, **the actual behaviour of
`scripts/` decides** — the scripts are the one unambiguous statement of these rules.

**The one exception**: if the translation itself is wrong (mistranslated, drifted,
inconsistent terminology), fix it here — **and go back and check whether the blueprint
was unclear too.** Translation exposing ambiguity is a side benefit; do not waste it.

Terminology comes from [GLOSSARY.md](GLOSSARY.md), shared across all three languages.

## Staying in sync

`SOURCE-MANIFEST.sha256` is the copy of the blueprint's manifest this translation was
generated from. Comparing it against the blueprint's current `MANIFEST.sha256` shows
which files have fallen behind. `VERSION` is identical across all three repositories;
one bump bumps all three.

## Install

```bash
git clone https://github.com/faliye/singlefs-ai-sop-install
bash singlefs-ai-sop-install/sop-install.sh --lang en
```

The installer's own prompts follow the language you choose.

## Licence

Dual-licensed: [Apache-2.0](LICENSE-APACHE) or [MIT](LICENSE-MIT), at your option.
