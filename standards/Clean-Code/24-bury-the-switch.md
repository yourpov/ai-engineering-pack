# 24. Keep high-level logic clean — bury the switch

> **Rule:** Do not let switch/if-else forests dominate high-level policy. Push polymorphism or lookup tables down.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Switches on type/status spread and duplicate when new cases appear. High-level code should read as a story; low-level modules own the branch tables or polymorphic dispatch.

---

## Rules

- One switch is better than the same switch copied in five files — bury it once (DRY).
- Prefer polymorphism / strategy / map of handlers when cases grow.
- High-level functions call `payment.complete()` rather than switching on `payment.type`.
- Exhaustive matching is fine when the language enforces completeness — still keep it out of the top-level narrative when large.

---

## Bad vs good

### Bad
```text
function process(p):
  switch p.type:
    case card: ...
    case wire: ...
    case crypto: ...
  // repeated in three other modules
```

### Good
```text
function process(p):
  p.complete()  // each type implements complete()
```

---

## Review checklist

- [ ] Is the same switch duplicated?
- [ ] Does high-level code still read like a story?

---

## Related standards

- 15 One thing
- 27 Abstraction levels
- 14 DRY

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
