# 36. Honest naming — no disinformation

> **Rule:** Never name something a list if it is a map. Never use near-identical names or lookalike characters.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Trust starts with names that do not mislead. `accountList` that is actually a map is a lie. `FooForEfficientHandling` vs `FooForEfficientStorage` guarantees autocomplete mistakes. `l` vs `1`, `O` vs `0` are disinformation.

---

## Rules

- Type-shaped names must match the structure (`Map`, `Set`, `List` only when accurate).
- Make similar concepts obviously distinct in spelling.
- Avoid visually confusable identifiers.
- If the implementation changed structure, rename in the same change.

---

## Bad vs good

### Bad
```text
accountList = { id -> account }  // it's a map
```

### Good
```text
accountsById = { id -> account }
```

---

## Review checklist

- [ ] Does the name’s data-structure word match reality?
- [ ] Could two names be swapped by accident in review?

---

## Related standards

- 01 Honest functions
- 04 Clarity
- 20 Names that fail

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
