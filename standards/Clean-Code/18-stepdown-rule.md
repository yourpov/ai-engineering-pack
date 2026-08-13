# 18. Stepdown rule — highest level first

> **Rule:** Highest-level function at the top; every definition below its first caller; related functions grouped. Read top-to-bottom like a newspaper.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Readers start at the top. Put the abstract story first, then descend into details. Callers above callees. Cluster helpers that serve the same narrative section.

---

## Rules

- Export / public entry points near the top of the file (or as the language convention dictates — stay consistent).
- After a function, place the helpers it calls before unrelated later chapters.
- Avoid scatter where you must jump randomly around the file to follow one story.

---

## Bad vs good

### Bad
```text
// 200 lines of helpers, then main at the bottom
```

### Good
```text
function handleRequest(...):  // story
  validate(...)
  execute(...)
  respond(...)

function validate(...): ...
function execute(...): ...
function respond(...): ...
```

---

## Review checklist

- [ ] Can I read the file top-down without jumping up for callees?
- [ ] Are related helpers adjacent?

---

## Related standards

- 15 One thing
- 27 Abstraction levels
- 35 Formatting

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
