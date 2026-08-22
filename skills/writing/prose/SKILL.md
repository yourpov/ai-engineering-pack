---
name: prose
description: >
  Prose quality for engineering writing: READMEs, docs, PR descriptions, commit
  messages, changelogs, release notes, and comment text. Use when writing or
  editing any of those, or when someone says a doc reads like AI wrote it, sounds
  generic, is too wordy, needs tightening, or asks to remove AI tells, slop, or
  filler. Single source of truth for prose rules in this pack. Does not score
  output against AI detectors and does not add fake inconsistency to look human.
---

# Prose (Universal)

The writing around code is read by someone who is trying to do a task. A teammate setting
the project up, a reviewer deciding whether to approve, a person reading a commit message
six months from now to find out why a line exists. Prose earns its place by making that
task faster.

This file is the only place in the pack that defines prose rules. Other files point here
rather than restating them. See [Authority](#authority).

**Non-goal:** passing an AI detector. Detector scores are not a quality measure, and
writing to beat one produces worse docs. Do not introduce typos, inconsistent formatting,
or deliberately uneven structure to seem human.

---

## 1. Words

Cut on sight:

| Category | Examples |
|---|---|
| **Filler adjectives** | seamlessly, robust, powerful, comprehensive, cutting-edge, blazingly |
| **Verb inflation** | leverage, utilize, facilitate, enable (when "let" works), delve into |
| **Discourse glue** | furthermore, moreover, additionally, in conclusion, that said, it is worth noting |
| **Throat-clearing openers** | "Let's dive in", "Here's the thing", "At its core", "In today's landscape" |
| **Meta-commentary** | "This section covers", "As mentioned above", "Great question" |
| **Hedges with no content** | arguably, essentially, basically, quite, very, really |

Adverbs get a higher bar than other words. Most that survive a first draft are describing
something the verb should have carried. `significantly improves` is a claim without a
number. `cuts p99 from 400ms to 90ms` is the sentence you meant.

Named things beat categories. `Redis` over `a caching layer`, `bun test` over `the test
suite`, `src/auth/session.ts` over `the relevant module`.

---

## 2. Punctuation

- **No em dash as sentence punctuation.** Use a period, comma, colon, or parentheses.
  Hyphens inside compound words are fine.
- **No exclamation marks** in docs, commits, or error copy.
- **Bold is for one thing per section**, at most. Bolding five phrases on a screen leaves
  nothing emphasized.
- Straight quotes in code and CLI output. Typographic quotes are a product-UI decision,
  covered in `prompts/scaffolds/Website-Design.md`.

---

## 3. Structural patterns

These are the tells that survive a word-level cleanup, and the reason a doc still reads as
generated after every banned adjective is gone.

**Binary contrast.** Setting up a thing only to negate it in the same breath.

> The language changes per project, the definition of good code does not.

Write the half that carries information: `Every template starts with Clean-Code because the
standards outlive the stack.`

**Negative listing.** Defining something by what it is not, when nobody suspected it was.

> This is not a framework. It is not a linter. It is a set of files.

Say what it is once.

**Dramatic fragmentation.** Sentence fragments deployed for rhythm.

> Load one file at a time. That is the rule. Nothing else matters as much.

One complete sentence does the same work.

**Rhetorical setup.** Announcing a count or a question, then answering yourself.

> Two things go wrong when you build with an agent.

Go straight to the first thing. Numbered headings already tell the reader there are two.

**False agency.** Giving inanimate subjects intent: `the config wants`, `the test is trying
to tell you`, `your code deserves`. Name the actual actor.

**Narrator distance.** Describing the reader's inner life from outside. `You might be
wondering`, `by now you have probably noticed`, `nothing is technically broken, so nothing
gets fixed`. State the condition instead.

**Passive voice hiding the actor.** `The migration is run automatically` leaves out what
runs it. Only use passive when the actor genuinely does not matter.

---

## 4. Rhythm tics

- **Always three.** Three examples, three bullets, three clauses, on every list. Real sets
  are rarely all size three. Use the number of items that exist.
- **Punchy closer.** Ending a section with a short declarative for effect: `Delete when in
  doubt.` `That is deliberate.` `Ship it.` Cut them, or fold the content into the paragraph
  above.
- **Forced parallelism.** Bending every bullet into the same grammatical shape when the
  items are not parallel in meaning.
- **Uniform sentence length.** Twelve sentences in a row at fifteen words each reads flat
  no matter how good the words are. Vary length because the ideas vary in size, not to
  perform variation.

---

## 5. Do not overcorrect

The rules above cost more than they save when applied blindly.

- A contrast that carries real information is a good sentence. The pattern to cut is the
  decorative kind that adds nothing.
- A list of three is fine when there are three things.
- Precise technical terms are not jargon. `idempotent`, `backpressure`, and `nullable` are
  the shortest correct words available.
- Repeating a term is better than reaching for a synonym. `Session` on every line beats
  alternating with `token`, `handle`, and `context`.
- Never change a factual claim to improve a sentence. Accuracy outranks every rule here.

---

## 6. By document type

| Type | The job | Common failure |
|---|---|---|
| **README** | Get someone running the project | Feature marketing instead of setup steps |
| **PR description** | Why this change, and what could break | A narrative of every commit |
| **Commit message** | Why, in one line plus optional body | Restating the diff |
| **Changelog** | What changed for someone using it | Internal refactors nobody outside can observe |
| **Error copy** | What happened and what to do next | Apology with no action. See `skills/engineering/errors/SKILL.md` |
| **Code comment** | Why, constraint, hazard | Narrating the next statement. See `skills/engineering/necessary-comments/SKILL.md` |

---

## 7. Checklist

- [ ] No filler adjectives, verb inflation, or discourse glue
- [ ] Every adverb either carries a number or is gone
- [ ] No em dash used as punctuation
- [ ] No binary contrast, negative listing, or rhetorical setup
- [ ] No fragments written for rhythm, no punchy section closers
- [ ] List lengths match the number of real items
- [ ] Claims name a file, command, or number the reader can check
- [ ] Nothing restates what a linked file already shows
- [ ] No detector-driven edits, no deliberate inconsistency

---

## Authority

Prose rules live here. These files point at this one:

| File | Relationship |
|---|---|
| `skills/engineering/craft/SKILL.md` | Code craft. Its prose section defers to this file |
| `skills/writing/create-readme/SKILL.md` | README discovery and evidence rules. Style defers here |
| `prompts/scaffolds/Readme-Voice.md` | README tone and section templates by project type |
| `prompts/reviews/Docs-Review.md` | Runs these rules as a scoped audit |

On naming inside code, `standards/Clean-Code/` wins (04, 06, 20, 30, 36, 39). This file
governs sentences, not identifiers.
