# 22. First drafts may be messy — do not ship them that way

> **Rule:** It is OK if the first draft is messy. It is not OK to leave it that way.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Exploration produces knots. Professionalism is the cleanup pass: names, extracts, tests, deletion of dead ends — before merge, or as a fast follow with a named debt item.

---

## Rules

- Separate spike branches from production paths when possible.
- Boy Scout Rule: leave the area cleaner than you found it — within the change’s budget.
- Do not merge “temporary” without a ticket and a deadline.
- Refactor only with a way to verify (tests, manual script, or project verify commands).

---

## Bad vs good

### Bad
```text
// WIP dump merged to main with debug prints and dead branches
```

### Good
```text
// Spike → rewrite clean → PR with clear story and no dead code
```

---

## Review checklist

- [ ] Would I be proud if this were the first file a new hire opened?
- [ ] Is temporary code labeled and scheduled for removal?

---

## Related standards

- 15 One thing
- 26 Extract
- 07 Delete dead code

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
