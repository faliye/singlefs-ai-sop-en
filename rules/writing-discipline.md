<!-- generated-from: rules/writing-discipline.md sha256:495897761644fd7524735650de56e8d7a46027abe927bf06a3a7f8d092867192 -->
<!-- doc-lint:rule-definition -->
# Writing discipline

This covers everything written for people to read: rules, README, design notes, the kb,
commit messages, code comments, gate failure messages.

## Keep the style simple and natural

**Write in plain modern English. Say it straight; don't circle around it.**

### Four things not to write

| Don't | What it looks like | Instead |
|---|---|---|
| **Bureaucratic padding** | "in the event that", "with respect to", "it is the case that", "perform an analysis of", "make use of" | "if", "about", drop it, "analyse", "use" |
| **Archaic or literary register** | "hereinafter", "thus it follows", "whereupon", "one might posit" | Say what it is and what follows |
| **Concessions that concede nothing** | "While <one point>, however <another point>" where the two points do not actually conflict | Keep only the half you mean |
| **Over-explaining** | "as is well known", "needless to say", "it goes without saying"; stating a conclusion and then restating its negation | Say it once |

### No abbreviations, nothing referred to by a code name

The criterion and the registry are those of `code-discipline.md`: the "Names" section and the rule "Comments contain no abbreviations and refer to nothing by a code name".

A project's own numbering systems that have a registry (mutation names, experiment numbers) are registered in `.claude/code-names` at the project root: one line `<regular expression>  # reason`; a code name is not judged only when the expression matches it in full.
Labels that a `.md` file declares itself with `<!-- doc-lint:not-numbers … -->` are used as usual in that file.

### Write conjunctions as conjunctions; unpack noun strings into sentences

- Where "so / because / then / but" belongs, write the word, not `⇒`, `×` or `+` in its place; inside a table cell, symbols may stay.
- Unpack noun strings: "main agent additionally verbatim-verifies four mandatory sites" becomes "I also checked the four mandatory sites word for word."
- Keep bold for verdicts and numbers; do not bold whole sentences or paragraphs.
- Label-style colons ("mechanism:", "basis:", "measurement:") may stay in a kb, but a complete sentence must follow the colon.

### The test

**Read it out loud.** Would you say this sentence to a colleague? If not, rewrite it.

- Do people use this word when talking? If not, swap it.
- Is this sentence explaining the previous one? The previous one was already clear: delete this one.
- Does this "however" actually reverse anything? If the two halves agree, drop it.

### Don't overcorrect

- Terms, measurement bases, numbers and commands stay exact: "block size 16 KiB" must not become "blocks aren't big".
- Write a judgement directly: "This shouldn't be settled yet", "this approach is wrong", without a run-up before turning into it.

### Which half the gate handles

| Who judges | What |
|---|---|
| `scripts/doc-lint.sh` | The fixed phrasings in a word list (the common padding and archaic constructions); a hit turns red and suggests the replacement. The rules proper are checked the same way: in a file whose head carries `<!-- doc-lint:rule-definition -->`, only the examples quoted in backticks and 「」 are exempt; the body is checked as usual |
| The gate stage "代号" (`scripts/code-name-lint.py`) | In the `.md` lines added relative to the diff base, two shapes of code name (digits followed by one lowercase letter, optionally followed by `-<letters or digits>`; two uppercase letters followed by two digits) and `⇒` (table rows and anything inside backticks are not judged); lines already present at the base are not swept; a code name inside backticks, fences, HTML comments, link targets or web addresses is not judged, nor are references to numbered steps such as "step 1b" |

The word list is Chinese only; in the English and Japanese repositories the word-list item reports unimplemented explicitly.
Whether a sentence flows, whether a concession is redundant, noun strings, `×` and `+`, other shapes of abbreviation and code name, code comments and text outside `.md` are not checked by the gate; those need a person to read it aloud.
