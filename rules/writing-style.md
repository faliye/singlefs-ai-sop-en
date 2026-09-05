<!-- generated-from: rules/writing-style.md sha256:2c185c99940ef836ba360a74a98dcf828f88012b02ec0ac3c3bfc9203f3e5aaa -->
<!-- doc-lint:rule-definition -->
# How to write: plainly

**Write in plain modern English. Say it straight; don't circle around it.**

This covers everything written for people to read: rules, README, design notes, commit
messages, gate failure messages, kb conclusions. Length is `writing-economy.md`'s job;
this page is about wording.

## Four things not to write

| Don't | What it looks like | Instead |
|---|---|---|
| **Bureaucratic padding** | "in the event that", "with respect to", "it is the case that", "perform an analysis of", "make use of" | "if", "about", drop it, "analyse", "use" |
| **Archaic or literary register** | "hereinafter", "thus it follows", "whereupon", "one might posit" | Say what it is and what follows |
| **Concessions that concede nothing** | "While X, however Y" where X and Y do not actually conflict | Keep only the half you mean |
| **Over-explaining** | "as is well known", "needless to say", "it goes without saying"; stating a conclusion and then restating its negation | Say it once |

## The test

**Read it out loud.** Would you say this sentence to a colleague? If not, rewrite it.

Three concrete questions:

- Do people use this word when talking? If not, swap it.
- Is this sentence explaining the previous one? If the previous one was already clear,
  delete this one.
- Does this "however" actually reverse anything? If the two halves agree, drop it.

## Don't overcorrect

**Plain does not mean vague.** Terms, measurement bases, numbers and commands stay
exact — "block size 16 KiB" must not become "blocks aren't big" in the name of
readability.

**Plain does not mean opinion-free either.** "This shouldn't be settled yet", "this
approach is wrong" — write exactly that. Saying it straight includes **stating the
conclusion first**.

## Which half the gate handles

`scripts/doc-lint.sh` checks **the fixed phrasings in a word list** — the common
padding and archaic constructions. A hit turns red and suggests the replacement.

**What it cannot check**: whether a sentence flows, whether a concession is redundant,
whether an explanation is bloated. Those need a person to read it aloud. So a green
word list does not mean this rule is being kept; it only means the most obvious traps
were avoided.

The word list is Chinese. In this edition the check reports **unimplemented** rather
than passing silently — inventing an English word list would add false positives, not
remove them (`show-me-test.md`: the gate must not pretend to pass).
