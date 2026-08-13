# 39. Pronounceable names — code is social

> **Rule:** If you cannot say the name in a review, rename it. Names should sound like human language.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Programming is social. Unpronounceable soup blocks onboarding and discussion. Prefer words you can speak in a standup.

---

## Rules

- No compressed vowel-stripped identifiers (`genymdhms` → `generationTimestamp`).
- Speak the name out loud once when inventing it.
- Acronyms only when the team already says them as words (URL, ID, HTTP).

---

## Bad vs good

### Bad
```text
genymdhms = ...
```

### Good
```text
generationTimestamp = ...
```

---

## Review checklist

- [ ] Can I say this name cleanly in a code review?
- [ ] Would a new hire pronounce it the same way?

---

## Related standards

- 04 Clarity
- 06 Intention-revealing
- 30 Searchable

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
