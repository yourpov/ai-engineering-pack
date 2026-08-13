# 17. Good comments tell why — code tells how

> **Rule:** Code shows how. Comments, when needed, explain why: constraints, tradeoffs, history the code cannot express.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

“How” comments duplicate the next line and rot. “Why” comments capture intent that is not in the syntax: business rules, vendor quirks, performance tradeoffs, legal constraints.

---

## Rules

- Delete comments that restate the next statement.
- Keep comments that explain non-obvious motivation.
- Prefer better names so fewer whys are needed — but do not delete real constraints.

---

## Bad vs good

### Bad
```text
// increment i
i = i + 1
```

### Good
```text
// Vendor API rejects requests within 1s of the previous; space them out
sleep(1100)
```

---

## Review checklist

- [ ] Does this comment explain motivation the code cannot?
- [ ] Would a better name delete the need for a “how” comment?

---

## Related standards

- 08 Protective
- 33 Code explains itself
- 19 Noise

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
