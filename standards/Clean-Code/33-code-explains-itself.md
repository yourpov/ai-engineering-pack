# 33. Stop commenting what code does — rename and extract instead

> **Rule:** Before writing a “what” comment, try better names, extracted variables, extracted functions, or named constants.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Every comment is a second artifact to maintain. Complicated conditions become English when pieces are named. Magic numbers become constants. Blocks become functions whose names are the old comment.

---

## Rules

- Extract explaining variables for complex conditions.
- Extract functions when a block answers one question.
- Replace magic numbers with named constants.
- Only then consider a why-comment (17, 08).

---

## Bad vs good

### Bad
```text
// check if employee is eligible for full benefits
if employee.flags & HOURLY and age > 65: ...
```

### Good
```text
if employee.isEligibleForFullBenefits(): ...
```

---

## Review checklist

- [ ] Am I about to write a comment that a rename would delete?
- [ ] Is there a magic number with a domain name waiting?

---

## Related standards

- 19 Noise
- 17 Why
- 15 One thing

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
