# 10. Comments must communicate instantly

> **Rule:** If the team cannot understand a comment immediately, rewrite or delete it. No mumbles, novels, or HTML-in-source.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Mumbled comments (“// not sure if works — RJ edge case??”) dump confusion. Epic history novels for one-liners are noise. HTML tags in source comments punish editors.

---

## Rules

- Unsure? Write a clear TODO that states the constraint and what to verify.
- Delete novel comments; the code should carry the meaning.
- Plain text only in line comments — no HTML intended for a doc site.
- Names over essays.

---

## Bad vs good

### Bad
```text
// RJ ??? maybe broken for that edge case from the old RFC and the 2018 library...
```

### Good
```text
// TODO: confirm behavior when region is missing; currently falls back to STANDARD_RATE
```

---

## Review checklist

- [ ] Can a teammate act on this comment without asking the author?
- [ ] Is it shorter than a paragraph unless the constraint is truly deep?

---

## Related standards

- 08 Protective comments
- 19 Noise
- 07 Git for history

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
