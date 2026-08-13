# 02. Argument roles (ask, transform, handle) — no flag-jobs

> **Rule:** Each argument has one clear role: data for a question, data for a transform, or an event to handle. Flags that switch which algorithm runs are two jobs in one function. This is about roles, not a hard “only one argument ever” limit (see 23, 28).

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Booleans and multi-args become harmful when they **select which algorithm runs** rather than supply data. `process(true)` forces the reader into the implementation.

**Output arguments** (mutate inputs instead of returning) break the expectation that data flows in via arguments and out via returns.

Clean **roles** (often one argument, sometimes a small natural set — see 23/28):

1. **Asking** — pass X, get an answer about X  
2. **Transforming** — pass X, get a new value derived from X  
3. **Handling an event** — something happened; react to it

---

## Rules

- Do not use boolean flags to merge two unrelated behaviors into one function — split the function.
- Passing `true` as real data (e.g. `setVisible(true)`) is fine; the value *is* the data.
- Prefer return values over output parameters.
- Arity: prefer few arguments (23). Pairs that are one concept (point, range) and careful triads are covered in 28 — still no flag-jobs.

---

## Bad vs good

### Bad
```text
render(user, includeAdminTools: true)  // flag chooses a second job
```

### Good
```text
renderUser(user)
renderAdminTools(user)  // separate, or compose at call site
```

---

## Review checklist

- [ ] Does each argument mean data, or a control flag for a second algorithm?
- [ ] Can the call site be understood without opening the callee?

---

## Related standards

- 23 Argument count
- 28 Pairs and triads
- 40 Command–query separation

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
