# 07. Leave history to Git (no commented-out code, attributions, or changelogs)

> **Rule:** Delete dead code. Do not keep museum pieces, author banners, or edit journals in source. Required license/copyright headers (41) are allowed.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Commented-out code starts as “we might need it” and becomes a half-alive file. Version control already stores every line ever written.

Author names at the top of files go stale as ownership moves. Dated “changelog” comments duplicate Git, poorly.

**Exception (not history):** standard **license / copyright** headers required by policy belong in source (41). They are legal, not changelogs or “who wrote this line.”

---

## Rules

- Delete commented-out code entirely — restore from Git if needed.
- No file-header author attributions as a substitute for `git blame`.
- No journal comments (`// 2024-01-03 fixed bug`). Use commit messages.
- Source files contain what runs **today** only, plus required license headers when policy demands them.
- Do not invent custom license essays — use a standard license text (41).

---

## Bad vs good

### Bad
```text
// function oldBilling() { ... }
// Fixed by Dave 3/12/2019
// Author: Alice
```

### Good
```text
// SPDX-License-Identifier: MIT   (or standard license block if required)
// (no commented-out code, no author journal — history lives in Git)
```

---

## Review checklist

- [ ] Is there commented-out code in this diff?
- [ ] Are there changelog or author banners that Git already covers?
- [ ] If a license header is present, is it standard — not a custom essay?

---

## Related standards

- 05 Comments go stale
- 19 Noise comments
- 41 Docs for the world

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
