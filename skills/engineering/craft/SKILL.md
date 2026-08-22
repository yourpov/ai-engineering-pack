---
name: craft
description: >
  Code and prose craft for maintainability and clarity. Use when reviewing naming,
  comments, structure, or written docs. Optimizes for humans who must debug and
  extend the system. Does not score "AI detection" and does not encourage fake
  inconsistency. Language-agnostic.
---

# Craft, Code and Prose Quality (Universal)

Goal: code and docs that a teammate can trust at 2 a.m. 
Non-goal: looking "less AI" on a detector. Detectors are not quality metrics.
Correctness, security, and clarity always beat stylistic camouflage.

**Authority:** naming and comment craft defer to `standards/Clean-Code/` (especially
04, 06, 17, 19, 20, 30, 33, 36, 39, 41). If this skill and those lessons disagree,
**Clean-Code wins**.

---

## 1. Naming

- Prefer domain language over empty CS suffixes when the domain term is clear.
- Names should make the next line obvious. If you need a comment to explain the
  name, rename.
- Consistency within a module beats clever variety. Do not introduce intentional
  inconsistency to "look human."
- Match existing project conventions from `AGENTS.md` and neighboring files.
  Framework idioms (e.g. event handlers, controllers) are fine when they match the stack.

**Smell:** `handleUserDataValidationResultProcessor` 
**Better:** `validateUser` or the verb the domain already uses 

**Smell:** renaming a clear `InventoryHandler` to something vague mid-audit for style points 
**Better:** leave idiomatic names; fix real bugs 

---

## 2. Comments

- Keep *why*: constraints, workarounds, invariants, hazard warnings.
- Remove *what*: narration of the next statement, step lists that restate code.
- If removing a comment makes code unclear, rename symbols; do not put the comment back.
- TODOs need a condition or owner context when possible: what blocks removal.
- Full test for when a comment earns its keep: `skills/engineering/necessary-comments/SKILL.md`.

---

## 3. Structure

- Flat until complexity demands a layer. Do not add repository/service/factory
  stacks for a three-line operation.
- One module, one job. Dumping-ground `utils/` with unrelated helpers is a smell.
- Prefer the same abstraction level inside a function (Clean Code).
- Asymmetry across modules is fine when complexity differs. Symmetry for its own
  sake is not a goal, and breaking symmetry on purpose is not a goal either.

---

## 4. Error handling and defense

- Guard boundaries and real failure modes. See `skills/engineering/errors/SKILL.md`.
- Do not wrap every line in try/catch. Do not null-check values the type system
  or caller already guarantees, unless you are at an untrusted boundary.
- Untrusted input (network, client, user, file) is always a boundary.

---

## 5. Prose (docs, PR text, commit messages)

Full rules live in `skills/writing/prose/SKILL.md`. Load it when writing or reviewing any
written material. The short version:

- Short sentences. Concrete verbs. No filler adjectives or discourse glue.
- No em dash as punctuation. Ordinary hyphens in compound words are fine.
- Watch the structural tells, not just the word list: binary contrast, negative listing,
  rhetorical setup, fragments written for rhythm.
- Do not restate what a linked file already shows; link it.
- Commit/PR body: why and risk, not a narrative of every keystroke.

---

## 6. Review checklist (craft only)

- [ ] Names match domain and local convention 
- [ ] Comments are why-only 
- [ ] No extra layers without a proven need 
- [ ] Failure paths have context; no silent swallow 
- [ ] Docs/PR text would pass a "senior engineer wrote this" bar without detector games 

If craft conflicts with security or correctness, craft loses.
