# 26. Extract from massive functions

> **Rule:** Large functions bury business logic under low-level detail. Extract until the parent is a short story.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

When a function scrolls for screens, readers lose the plot. Extract cohesive blocks into named helpers. The parent becomes an outline; children own details.

---

## Rules

- If you need comments as section headers, extract instead (15, 38).
- Extract until each function fits one abstraction level (27).
- Do not extract blindly without a way to verify behavior.
- UI “god components” count — split by view/responsibility.

---

## Bad vs good

### Bad
```text
function renderPage():
  // 400 lines of fetch, parse, map, JSX, analytics, error UI
```

### Good
```text
function renderPage():
  data = loadPageData()
  return view(data)

function loadPageData(): ...
function view(data): ...
```

---

## Review checklist

- [ ] Does the top function fit on one screen as an outline?
- [ ] Does each extract have a name that is not just restating the code?

---

## Related standards

- 15 One thing
- 18 Stepdown
- 27 Abstraction levels

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
