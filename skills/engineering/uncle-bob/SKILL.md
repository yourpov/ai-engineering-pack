---
name: uncle-bob
description: >
  Robert C. Martin's craft and professionalism applied as methodology, not as a
  report about any one repository. Use when reviewing structure, naming,
  comments, architecture seams, TDD discipline, or professional scope decisions.
  Pair with standards/Clean-Code/ for mandatory lesson standards (01–41),
  standards/Principles.md for the broader handbook, and skills/engineering/errors/SKILL.md for failure
  handling. Language- and framework-agnostic. Never treat paths or product names
  in other project notes as facts about the current tree.
---

# Uncle Bob, Methodology (Universal)

Robert C. Martin's work spans four books: **Clean Code** (function and class craft),
**Clean Architecture** (the Dependency Rule, SOLID at system scale), **The Clean
Coder** (professionalism), and **Agile Software Development** (TDD laws and early
SOLID essays). `standards/Clean-Code/` is the **mandatory** lesson-by-lesson standard
set. `standards/Principles.md` is the long handbook (SOLID, testing, security). This
file is shorter: how to *apply* those ideas without inventing facts about the codebase
you happen to be in.

**Load rule:** optional methodology reference. Concrete craft rules live in
`standards/Clean-Code/` (mandatory). Do not treat this file as a description of the
current repo. If a principle needs a concrete example, derive it from code you
actually opened in *this* tree.

**Authority:** on naming, functions, comments, null, exceptions, Demeter, DRY, CQS —
`standards/Clean-Code/` wins over `Principles.md` and other skills. This file teaches
*how to apply* those ideas in reviews.

---

## 1. Clean Code, applied

### 1.1 Functions and naming

- One level of abstraction per function. If a function does X *and then* Y, split it.
- Names answer "what does this do" so a comment restating the body is unnecessary.
- Prefer small functions. Extract when a block needs its own name to stay clear.
- Leave oversized functions as **named debt** if splitting them blind risks
  behavior change without a way to verify. Do not "clean" without a seam or test.

### 1.2 Error handling

- Read the **full path** an error takes before calling a catch "swallowed." Logging
  or a sink upstream may already surface it.
- Discarded results with no path to the user or logs are real defects.
- Expected outcomes (already-running service, empty list, idempotent no-op) are
  not internal errors. Do not log them as failures (Special Case / normal flow).
- Details and patterns: `skills/engineering/errors/SKILL.md`.

### 1.3 Comments

- Comments explain *why* (constraint, workaround, non-obvious invariant).
- Comments that narrate *what* the next line does are violations. Rename instead.
- Delete comment graveyards and commented-out code that source control already keeps.
- Full test for when a comment earns its keep: `skills/engineering/necessary-comments/SKILL.md`.

### 1.4 Magic numbers and constants

- Name values that carry meaning across call sites.
- Do not reuse an unrelated constant because the number happens to match.
- Prefer language builtins over hand-rolled tables for simple transforms.

---

## 2. Clean Architecture, the Dependency Rule

Source code dependencies point **inward**:

- Domain / core policy does not import frameworks, UI, or I/O adapters.
- Use cases depend on ports (interfaces, traits, exports contracts), not concrete drivers.
- Adapters implement ports and sit at the edge.
- Thin outer layers translate transport (HTTP, IPC, game events) into use-case calls.

**How to audit any repo:** find the innermost policy, list its imports, and ask
whether those imports are policy or infrastructure. Flag inward violations. Do not
assume a `domain/` or `application/` folder exists; look for the *dependency
direction*, not folder names copied from another project.

When a change is cheap, it is usually because a seam already exists. Prefer adding
a parameter or port over reaching through layers.

---

## 3. The Clean Coder (professionalism)

- **Say no** when a request conflicts with its own justification. Name the concern,
  then take direction once facts are clear.
- **"I don't know yet"** beats a guess stated as fact. Read-only first pass, then
  verify against the real call chain before "fixing."
- **Boy Scout Rule has a budget.** Leave the campground cleaner; do not rewrite
  unrelated modules mid-feature. Name deferred debt instead of expanding scope.
- **Estimates and scope.** Ship a bounded, reviewable slice. Name what is deferred.
  Do not silently take the maximum interpretation of an open-ended ask.

---

## 4. Three Laws of TDD

From *Agile Software Development*:

1. Do not write production code except to make a failing test pass.
2. Do not write more of a test than is sufficient to fail.
3. Do not write more production code than is sufficient to pass the current failing test.

Many codebases do not run strict TDD. Still aim for the *shape* TDD produces:

- Tests exercise behavior through public seams, not private internals.
- Fakes stand in for I/O and frameworks so signature changes stay local.
- Green tests before refactor; never refactor broken code first.

If the project has no test runner, say so and use the verification commands from
the project agent contract (`AGENTS.md` or equivalent). Do not invent a toolchain.

---

## 5. How to use this file in a review

1. Load project `AGENTS.md` (or README conventions) first.
2. Scope the review (files, report-only vs fix). See `skills/review/audit/SKILL.md`.
3. Apply sections 1-4 to *opened* code only.
4. For each finding: location, trigger, severity, whether verified end-to-end.
5. Separate real violations from patterns that only look wrong in isolation.

**Verdict style:** not zero findings, and nothing swept under the rug. Flag
unverified claims as unverified.
