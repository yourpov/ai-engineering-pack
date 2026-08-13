# 14. DRY — one authoritative representation of each piece of knowledge

> **Rule:** Every piece of knowledge has a single, unambiguous, authoritative home. Change once.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Duplication is the root of many defects: twenty fetch helpers, twenty timeouts, miss one → silent bug. Extract shared policy (timeouts, auth headers, validation, CORS helpers, project-type lists) into one function/module.

---

## Rules

- If you must change the same idea in two places, extract a third place both call.
- Copy-paste “just for now” needs a ticket or immediate extraction.
- DRY is about **knowledge**, not about forcing unrelated code into a wrong abstraction.
- Config values (limits, endpoints, magic numbers) live in named constants/modules.

---

## Bad vs good

### Bad
```text
// twenty endpoints each open-code fetch + timeout + error check
```

### Good
```text
function apiGet(path):
  // timeout, auth, error mapping once
getUsers = () => apiGet("/users")
getPosts = () => apiGet("/posts")
```

---

## Review checklist

- [ ] If this constant/rule changes, how many files must I edit?
- [ ] Is the abstraction named after the knowledge, not a vague `Utils` dumping ground?

---

## Related standards

- 31 Wrap third parties
- 26 Extract
- 30 Searchable names

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
