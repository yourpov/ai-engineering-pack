# 12. Define the error contract before the logic

> **Rule:** When a function can fail, define its public failure type and boundary try/catch (or language equivalent) first — then fill the happy-path steps inside. Domain steps stay free of catch-as-if (34, 37).

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

If `withdraw` breaks, everything depending on it breaks. Writing the failure contract first (tests + translation into a stable type like `TransactionFailed`) freezes what callers can rely on. Logic added later stays contained.

**Where the try lives:** at the **function/boundary that owns the contract**, not around every low-level line. The algorithm inside remains a straight list of steps (37). Catch translates and rethrows or reports — it does not implement normal branches (34).

---

## Rules

- Decide what the function may throw before implementing steps (03).
- Translate lower-level failures into the function’s public error at that boundary.
- Add tests for the failure mode first when practical.
- Callers should not depend on incidental library error types (31).
- Do not use the contract catch for expected alternate business paths (34).

---

## Bad vs good

### Bad
```text
// logic first, ad-hoc errors leak everywhere
function withdraw(...):
  account = db.fetch(...)  // throws DbError, Timeout, ...
  account.balance -= amount
```

### Good
```text
function withdraw(...):
  try:
    account = loadAccount(...)
    account.debit(amount)
    save(account)
  catch any as cause:
    throw TransactionFailed(cause)  // translate once; still a failure

// callers use try only where they can handle or report TransactionFailed
```

---

## Review checklist

- [ ] Do callers know the stable failure type?
- [ ] Are lower-level exceptions translated, not leaked?

---

## Related standards

- 03 Exceptions over codes
- 31 Wrap third parties
- 37 Algorithm breathes

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
