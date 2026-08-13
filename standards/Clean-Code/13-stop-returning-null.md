# 13. Do not return null — empty values, Special Case, or throw

> **Rule:** Never return null from APIs you control. Use empty collections, zero/no-op values, Special Case / Null Object, or throw. Do not pass null either — fail fast at the boundary.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Null return forces every caller into a pyramid of guards. Miss one → NPE far from the cause. Uncle Bob: **don’t return null; don’t pass null.**

When absence is **normal success**:

- no orders → **empty list** (not null, not an error)
- no discount → **0** or a no-op policy object
- missing optional entity → **Special Case / Null Object** with the same interface, or a typed optional only if the type system forces explicit handling everywhere (`Optional`, `Option`, `Maybe`) — not raw null soup

When absence is **failure** (invariant broken, missing required data): **throw** (03) with context.

**Never** use empty list / zero / null to hide an operational failure (failed fetch → `[]`). That is catch-as-if (34).

---

## Rules

- Public and internal APIs you own: no `return null`.
- Collections: empty, not null.
- Numeric absence with identity value: 0 / 1 / no-op value.
- Missing collaborator with safe defaults: Special Case object (same interface).
- True failure: throw with context (03). Do not return null “so the caller can check.”
- Do not pass null into callees; reject at the trust boundary with a clear error.
- Do not paper over bad producers with more null checks — fix the producer.
- Successful empty ≠ failed call: failures use exceptions (or the language error channel per 03), never silent empty.

---

## Bad vs good

### Bad
```text
orders = getOrders(user)     // may null
if orders != null:
  for o in orders: ...

// also bad: failure disguised as empty
try: return fetchOrders()
catch: return []
```

### Good
```text
orders = getOrders(user)     // always a list
for o in orders: ...

// required user missing → throw
// optional display name → Special Case or Optional type, not raw null
```

---

## Review checklist

- [ ] Does any function I own return raw null?
- [ ] Are callers stacked with defensive null checks a better return type would delete?
- [ ] Can a failed I/O path still look like successful empty data?

---

## Related standards

- 03 Exceptions over error codes
- 34 Catch is not if
- 37 Algorithm breathes

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
