# 19. If the code already says it, the comment is noise

> **Rule:** Delete comments that restate the obvious on private code. Noise trains people to ignore all comments. Public API docs (41) and protective why-comments (08, 17) are not noise.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

`// set name` above `setName`, or `{/* Logo */}` above a logo component adds nothing. Noise dilutes the value of real comments.

**Not noise (keep):**

- Public exported API documentation: what / inputs / outputs / failures / example (41)
- Why / warning / amplification / constrained TODO (08, 17)
- Required license headers (07, 41)

---

## Rules

- No comments that only restate the next private line or a private method name.
- No section banners that only restate structure (38).
- Keep comments that add information the code cannot (why, constraint, warning).
- Exported signature-level docs for public APIs are required by 41 — do not delete them as “noise.”

---

## Bad vs good

### Bad
```text
// User service for users
class UserService:
  // constructor
  constructor() {}
```

### Good
```text
class UserService:
  constructor() {}

// Public API (41) — keep:
// Creates a billing invoice for the given subscription. Throws BillingFailed if the gateway rejects the charge.
function createInvoice(subscription): Invoice
```

---

## Review checklist

- [ ] If I delete this comment, is any knowledge lost?
- [ ] Is this public API surface (keep docs) or private narration (delete/rename)?

---

## Related standards

- 33 Code explains itself
- 41 Docs for the world
- 08 Protective comments
- 17 Why not how

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
