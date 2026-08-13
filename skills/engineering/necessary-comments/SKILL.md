---
name: necessary-comments
description: >
  Robert C. Martin's "Necessary Comments" rule: comments are a last resort, not
  a default. Use when writing, reviewing, or stripping comments; when deciding
  whether an explanation belongs in prose or in a rename/extraction; or when a
  reviewer flags a comment as narration, noise, or non-local information.
  Pairs with skills/engineering/craft/SKILL.md (comment quality) and skills/engineering/uncle-bob/SKILL.md (Clean
  Code methodology). Language-agnostic.
---

# Necessary Comments (Universal)

Source: Robert C. Martin, ["Necessary
Comments"](https://blog.cleancoder.com/uncle-bob/2017/02/23/NecessaryComments.html),
read in full before applying this skill to a real review, plus the comment
chapter of *Clean Code* (`standards/Principles.md` §3).

**Authority:** comment craft is defined by `standards/Clean-Code/` lessons
**05–10, 17, 19, 33, 38, 41** (and protective cases in **08**). If this skill and
those lessons disagree, **Clean-Code wins**.

**The default is zero.** Every comment is a confession that the code failed to
explain itself. Before writing one, try a rename, an extraction, or a smaller
function first. Most of the time one of those wins and the comment becomes
unnecessary.

---

## 1. The test for "necessary"

A comment earns its place only when **the code cannot express it, no matter
how it's restructured.** Concretely, that's a short list:

1. **An algorithm's shape isn't visible in the code.** A choking / debounce /
   backoff algorithm's behavior across several timed scenarios is easier to
   read from a small timing diagram in a comment than to infer from the
   conditionals implementing it. The diagram is the necessary part, not prose
   restating the `if` statements.
2. **A decision that looks wrong without its rationale.** Code that looks like
   an obvious "bug" to a future reader (an inverted condition, a skipped
   validation, a magic offset) needs one line saying why, or someone will
   "fix" it back into an actual bug.
3. **A warning of consequences.** `// don't call on the hot path, allocates`,
   `// deliberately not thread-safe, see #142`. The cost isn't visible at the
   call site.
4. **Best-effort failure handling (rare).** Empty `catch` is **forbidden**
   (Clean-Code **34**). If work is truly best-effort, **log with context** and a
   short why-comment — or rethrow. Prefer design that needs no catch.
5. **Public API docs.** Javadoc / TSDoc / rustdoc on exported symbols (Clean-Code
   **41**): what, inputs, outputs, failures, example. Not noise (**19**). Still
   must not narrate the private implementation.
6. **Legal notices**, required license/copyright headers (Clean-Code **07**, **41**).

If a comment doesn't fit one of those, it almost certainly doesn't survive
this skill.

---

## 2. The test for "not necessary" (rename or extract instead)

- **Restates the next line.** `// increment i` above `i++`. Delete it.
- **Explains what a poorly-named thing does.** Rename the thing. A comment
  that says "system" means the guard/lock is protecting a subsystem you
  could have named; name it, and the comment stops earning its keep.
- **Narrates a well-known idiom.** Recovering a poisoned mutex, guarding a
  null, wrapping a third-party error, anything a competent reader of the
  language already recognizes on sight, doesn't need a caption. If the
  *idiom* needs teaching, that's a team wiki page, not a comment on every
  occurrence of it.
- **Cites something the reader can't open.** A comparison to another
  project's file, an old ticket number with no link, an author's name. If
  the reference isn't reachable from this repo, it's not evidence, it's
  trivia. (`standards/Principles.md` §3.2, "non-local information".)
- **A banner or section label.** `// ==== Helpers ====` above a group of
  functions that already read as a group. The blank line already did that
  job.
- **Duplicates a doc comment one function away.** State a rule once, at the
  layer that owns it; don't re-explain it at every call site.

---

## 3. Applying this to a review

1. Read every comment in scope. For each: which numbered case in §1 does it
   satisfy? If none, it's a candidate for deletion.
2. Before deleting, check whether removing it would leave a future reader
   confused about *why*, not *what*. "Why" gaps are real findings; "what"
   gaps mean the code needs a better name, not the comment back.
3. Don't swing to zero-tolerance past the point of usefulness. A necessary
   comment that's merely a little long is a trim, not a deletion, if the
   only content past the second sentence is restating code, cut from there.
4. Rustdoc/TSDoc/Javadoc on exported symbols is exempt from "restates the
   code" scrutiny at the *signature* level (params, return, throws) but not
   from narrating the *body*.

---

## 4. Quick reference

| Keep | Cut |
|---|---|
| Timing/algorithm diagram code can't show | Restates the next line |
| Why a "looks like a bug" line is correct | Explains a poorly-named symbol (rename instead) |
| Consequence warning (perf, thread-safety, hot path) | Narrates a well-known language idiom |
| Justified empty/swallowed branch | Cites another repo/project the reader can't open |
| Exported-symbol API doc (signature-level) | Section-banner / position marker |
| Legal notice | Duplicate of a doc comment elsewhere |
