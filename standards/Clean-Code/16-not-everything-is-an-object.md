# 16. Not everything needs to be an object

> **Rule:** Before creating a class, ask whether a function, data record, or module is enough. Avoid ceremony for its own sake.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Classes are tools, not virtues. A one-method class with no real identity is often a function. Over-objectifying simple transforms adds noise and digs inheritance holes.

---

## Rules

- Prefer plain functions/modules for stateless transforms.
- Use objects when they hide data and expose behavior over time (see 21, 29).
- Use data structures/DTOs when you are only carrying fields.
- Do not invent hierarchies for a single use site.

---

## Bad vs good

### Bad
```text
class StringCapitalizer:
  function capitalize(s): return s.upper()
```

### Good
```text
function capitalize(s): return s.upper()
```

---

## Review checklist

- [ ] Does this type have a real domain identity or lifecycle?
- [ ] Would a function module be clearer?

---

## Related standards

- 29 Pick a side
- 21 Expose behavior
- 16 (this)

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
