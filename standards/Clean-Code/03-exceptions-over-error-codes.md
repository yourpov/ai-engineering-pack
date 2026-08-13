# 03. Prefer exceptions over error codes

> **Rule:** Operational failures use exceptions (Uncle Bob default). Do not return status codes or dual success/error bags that every caller must check. Keep the happy path a flat list of steps.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Error codes start as one `if` and grow into nested pyramids. Every caller must check or the error vanishes. Shared error enums couple the whole codebase.

Exceptions keep the algorithm as a **list of steps**. Extract try/catch into its own function so normal flow and error flow are not interleaved line-by-line.

**Languages without exceptions (Go, etc.):** use the language’s single idiomatic error channel (`error` as a second return **only where the language requires it**), always with context, and still keep the happy path free of nested status pyramids. Do not invent parallel `{ ok, error }` objects in TypeScript/Java/C# when exceptions exist.

**Not the same as error codes:** pure queries returning `boolean`, empty collections, or Special Case objects (13, 34).

---

## Rules

- Happy path: sequential steps without nesting for status checks.
- In exception languages: throw on operational failure; catch at a boundary that can translate or recover.
- Do not return `{ success: false, error }` / error enums from domain logic in languages that have exceptions.
- HTTP/API edges map exceptions (or the language error channel) to status codes **once** — not at every internal step.
- When try and catch both grow large, extract each into its own named function (12, 37).
- Every thrown/returned failure carries **context** (what failed, which ids, cause) — bare empty throws are forbidden.
- Wrap third-party failures into **your** types (31). Callers do not catch vendor exceptions all over the tree.

---

## Bad vs good

### Bad
```text
account = openAccount(data)
if account.error: return account.error
profile = addProfile(account)
if profile.error: return profile.error
```

### Good
```text
function createUser(data):
  account = openAccount(data)
  addProfile(account, data)
  sendWelcome(account)
  return account

function runCreateUser(data):
  try:
    return createUser(data)
  catch error:
    handleCreateUserError(error)  // boundary: log, translate, rethrow or user message
```

---

## Review checklist

- [ ] Is the main algorithm readable without scanning every step for status codes?
- [ ] Is there a single boundary that translates failures to user/API responses?
- [ ] Does every failure carry operation + identifiers + cause?

---

## Related standards

- 37 Let the algorithm breathe
- 34 Catch is not if
- 12 Define the contract first
- 31 Wrap third parties

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
