<!-- generated-from: rules/writing-discipline.md sha256:1f50309b91a6fcaad31acfb14e377ab6e15b03e8a26efdb7776eb34676d2f34f -->
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

The gate stage "代号" (`scripts/code-name-lint.py`) judges the `.md` lines added relative to the diff base for two shapes: digits followed by one lowercase letter (optionally followed by `-<letters or digits>`), and two uppercase letters followed by two digits. Nothing inside backticks, fences, HTML comments, link targets or web addresses is judged, nor references to numbered steps such as "step 1b"; labels that a `.md` file declares itself with `<!-- doc-lint:not-numbers … -->` are not judged in that file, and are judged as usual in every other file.
A project's own numbering systems that have a registry (mutation names, experiment numbers) are registered in `.claude/code-names` at the project root: one line `<regular expression>  # reason`; a code name that the expression matches in full is not judged.
Abbreviations, other shapes of code name, code comments and text outside `.md` are not checked by the gate; lines already present at the base are not swept.

### Write conjunctions as conjunctions; unpack noun strings into sentences

Do not replace conjunctions with `⇒`, `×`, `+`, and do not compress an action into a noun string like "verbatim verification of the four mandatory sites".

- Where "so / because / then / but" belongs, write the word, not `⇒`; inside a table cell, where space is tight, symbols may stay.
- Unpack noun strings: "main agent additionally verbatim-verifies four mandatory sites" becomes "I also checked the four mandatory sites word for word."
- Keep bold for verdicts and numbers; do not bold whole sentences or paragraphs.
- Label-style colons ("mechanism:", "basis:", "measurement:") may stay in a kb, but a complete sentence must follow the colon.

The gate stage "代号" (`scripts/code-name-lint.py`) judges `⇒`: a `.md` line added relative to the diff base that contains `⇒` goes red; table rows and anything inside backticks are not judged. Noun strings, `×` and `+` it does not judge; lines already present at the base are not swept.

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
