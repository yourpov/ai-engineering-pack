# 05. Code evolves; comments may not

> **Rule:** Treat outdated comments as worse than no comments. Prefer code that does not need them.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Code moves, splits, and merges. Comments often stay behind and tell the **old** story. An inaccurate comment does not merely fail to help — it actively deceives.

You will still write comments (see 08–10, 17). Know which help and which hurt.

---

## Rules

- When you change behavior, update or delete the adjacent comment in the same commit.
- Never leave a comment that contradicts the current code.
- Prefer renaming/extracting so the comment becomes unnecessary.
- During review, flag comments that look older than the surrounding code.

---

## Bad vs good

### Bad
```text
// Returns the active user only
return users.filter(u => u.role == "admin")  // comment lies
```

### Good
```text
return users.filter(u => u.role == "admin")  // or rename to listAdmins()
```

---

## Review checklist

- [ ] Does every comment still match the code after this change?
- [ ] Could a rename make the comment deletable?

---

## Related standards

- 07 Leave history to git
- 19 Noise comments
- 33 Code explains itself

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
