# 20. If you must read the implementation to understand the name, the name failed

> **Rule:** A name that requires opening the body has failed. Rename until the call site is enough.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Call sites are the product. If every use of `process`, `handle`, `doStuff`, or `manager` requires reading the body, navigation is broken.

---

## Rules

- Prefer domain verbs and nouns over vague `process` / `handle` / `data` / `manager` / `util`.
- Encode important outcomes in the name (`archiveExpiredSessions`, not `run`).
- If the function does two things, split — do not invent a baggy name that covers both (see 01).

---

## Bad vs good

### Bad
```text
result = process(data)
```

### Good
```text
invoice = applyLateFees(invoice)
```

---

## Review checklist

- [ ] Do I understand the call without Go-to-Definition?
- [ ] Is the name specific enough to search (see 30)?

---

## Related standards

- 04 Clarity
- 06 Intention-revealing
- 36 No disinformation

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
