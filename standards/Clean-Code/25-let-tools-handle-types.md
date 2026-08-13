# 25. Let tools handle types — you focus on logic

> **Rule:** Use the type system, linters, and formatters. Do not re-encode what the compiler already guarantees.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Manual type tags, redundant runtime checks that the type system already enforces, and hand-formatted wars waste attention. Configure tools; write domain logic.

---

## Rules

- Enable strict typing where the language supports it; do not disable it to “move faster.”
- Do not maintain parallel shadow type systems in comments or string tags.
- Trust exhaustiveness checks; fix the model instead of casting everything to `any` / untyped.
- Format with the project’s automatic formatter — no style bikeshedding in review.

---

## Bad vs good

### Bad
```text
// @type {string} name
function f(name):  // untyped language surface when types exist
  if typeof name != "string": ...
```

### Good
```text
function f(name: string): void
  // logic only
```

---

## Review checklist

- [ ] Are we fighting the typechecker with casts instead of modeling correctly?
- [ ] Is formatting automated?

---

## Related standards

- 35 Formatting
- 04 Clarity
- 12 Contracts

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
