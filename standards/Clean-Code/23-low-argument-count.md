# 23. Keep argument count low

> **Rule:** Fewer arguments reduce mental weight. Prefer zero–two; treat three as a smell; four+ usually needs a structure.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Each argument is another thing to order, remember, and misuse. Bundle related params into a typed object/record when the list grows. Flag args that only select behavior are worse (see 02).

---

## Rules

- 0–2 arguments preferred for most functions.
- 3 arguments require a natural, documented order (see 28).
- 4+ → introduce a parameter object / options type with named fields.
- Do not pass booleans that fork the function into two programs.

---

## Bad vs good

### Bad
```text
createUser(name, email, role, active, sendEmail, tenantId, meta)
```

### Good
```text
createUser(CreateUserRequest{ name, email, role, active, tenantId })
// sendEmail decided by a separate policy or command
```

---

## Review checklist

- [ ] Can I reduce arity by bundling or splitting functions?
- [ ] Is any argument a feature flag in disguise?

---

## Related standards

- 02 Single-arg patterns
- 28 Pairs and triads
- 14 DRY

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
