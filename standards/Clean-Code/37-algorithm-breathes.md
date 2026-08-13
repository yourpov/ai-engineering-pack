# 37. Use exceptions so the algorithm can breathe

> **Rule:** Separate the happy-path algorithm from error handling so each can be read alone.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Return-code nesting makes you read two programs at once: success and failure. Exceptions pull failure aside so the success algorithm is four clear steps again.

---

## Rules

- Happy path is a straight sequence of domain steps.
- Error policy lives in catch blocks or a dedicated handler function.
- Do not interleave logging/retry/status checks between every domain line when a boundary can own them.
- Still do not use catch as ordinary branching (34).

---

## Bad vs good

### Bad
```text
// every step wrapped in if err != nil / else pyramids
```

### Good
```text
function moderateMessage(...):
  verifySession(...)
  resolveChannel(...)
  moderateContent(...)
  broadcast(...)
```

---

## Review checklist

- [ ] Can I quote the algorithm in four bullets from the function body?
- [ ] Is error policy separable?

---

## Related standards

- 03 Exceptions
- 34 Catch not if
- 15 Story

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
