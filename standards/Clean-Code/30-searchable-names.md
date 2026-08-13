# 30. Searchable names make the code navigable

> **Rule:** The length of a name matches the size of its scope. Magic numbers and single letters are unsearchable.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Searching for `7` or `e` is useless. Searching for `MAX_CLASSES_PER_STUDENT` or `WORKDAYS_PER_WEEK` is instant. Single letters are only for tiny local scopes.

---

## Rules

- Replace magic numbers with named constants at meaningful scope.
- Single-letter names only in short loops/lambdas with obvious meaning.
- Wider scope → longer, more specific names.
- Avoid Unicode lookalikes and near-duplicate names (see 36).

---

## Bad vs good

### Bad
```text
pay = hours * 5 * 4
```

### Good
```text
pay = hours * HOURLY_RATE * WORKDAYS_PER_WEEK
```

---

## Review checklist

- [ ] Can I find every use of this concept by searching its name?
- [ ] Are raw numbers domain-meaningful without a name?

---

## Related standards

- 06 Intention-revealing
- 04 Clarity
- 36 No disinformation

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
