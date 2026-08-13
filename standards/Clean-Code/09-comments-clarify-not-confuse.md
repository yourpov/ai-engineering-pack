# 09. Comments should clarify, not confuse

> **Rule:** Be specific, obvious, and local. Misleading or incomplete comments are worse than none.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Technically true comments can still be dangerous: they give false confidence to stop reading. Comments that raise more questions than they answer fail their job. Comments about code you do not control go stale when the other file changes.

---

## Rules

- If a comment describes behavior, it must cover the full behavior (not only the happy/premium path).
- Do not write comments that need their own comments.
- Only comment on the code immediately next to the comment.
- Never document another module’s defaults from afar.

---

## Bad vs good

### Bad
```text
// Returns 20% discount for premium users
return user.premium ? 0.2 : 0.05  // comment hides non-premium path
```

### Good
```text
function discountRate(user):
  return user.premium ? PREMIUM_RATE : STANDARD_RATE
```

---

## Review checklist

- [ ] Could this comment make someone skip reading and get the wrong idea?
- [ ] Is every claim local and complete?

---

## Related standards

- 05 Comments go stale
- 10 Communicate clearly
- 33 Explain with code

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
