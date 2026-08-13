# 32. Stop chaining through strangers (Law of Demeter)

> **Rule:** Talk to friends, not strangers. Do not reach through returned objects’ objects’ objects.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

A method may call: itself, its fields, objects it created, or arguments — not the innards of what those return (when those are true objects).

**Data structures** may expose fields; Demeter is about objects with behavior. Getters on data can create false ambiguity — prefer clear DTOs or tell-don’t-ask APIs.

---

## Rules

- Replace `a.getB().getC().getD()` with a single intention-revealing call on a friend.
- Do not fix Demeter by inventing `getB_C_DPath()` that still encodes the chain — ask for the outcome you need.
- Flatten external payloads into view models at a boundary (see 31).

---

## Bad vs good

### Bad
```text
path = context.options.scratchDir.absolutePath
```

### Good
```text
file = context.createScratchFile(name)
```

---

## Review checklist

- [ ] Does this line know too much about foreign structure?
- [ ] Is the chain data (OK) or behavior objects (not OK)?

---

## Related standards

- 21 Expose behavior
- 31 Wrappers
- 29 Objects vs data

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
