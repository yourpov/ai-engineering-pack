# 31. Wrap third-party APIs — own your exception types

> **Rule:** Do not catch library exceptions all over the codebase. Wrap vendors so your design and error types stay yours.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Library exception shapes are not your domain model. Catching three AWS errors the same way couples you to AWS. A wrapper translates vendor errors into **your** failures and gives one seam to swap vendors.

---

## Rules

- One adapter/client module per external system (HTTP API, SDK, WS protocol).
- UI and domain code never import vendor SDKs directly when a wrapper exists.
- Map vendor errors → your error types at the wrapper boundary.
- Wrappers also isolate URL building, auth headers, retries, and timeouts (DRY).

---

## Bad vs good

### Bad
```text
try:
  s3.put(...)
catch S3ErrorA, S3ErrorB, S3ErrorC:
  // same handling
```

### Good
```text
// storage.put hides S3; throws StorageFailure
storage.put(...)
```

---

## Review checklist

- [ ] How many files import the vendor SDK?
- [ ] Could we swap the vendor by editing one module?

---

## Related standards

- 14 DRY
- 12 Contracts
- 03 Exceptions

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
