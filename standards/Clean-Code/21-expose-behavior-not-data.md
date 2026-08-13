# 21. Expose behavior, not data

> **Rule:** Objects hide data and offer operations. Do not leak guts through getters for others to reimplement your logic.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Asking for fields and computing outside the object spreads business rules. Tell the object to do the work. Data structures are the opposite (see 29) — pick a side.

---

## Rules

- Prefer `order.total()` over `order.getItems()` + external sum.
- Do not grow public getters just so other modules can re-encode your invariants.
- When many consumers need the same derived value, put it on the object/module that owns the data.

---

## Bad vs good

### Bad
```text
if user.getAccount().getPlan().getName() == "pro":
  ...
```

### Good
```text
if user.hasProPlan():
  ...
```

---

## Review checklist

- [ ] Are outsiders implementing rules that belong on the type?
- [ ] Is this a Demeter violation chain (see 32)?

---

## Related standards

- 29 Pick a side
- 32 Law of Demeter
- 15 One thing

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
