# 34. Stop using catch as an if

> **Rule:** Exceptions are for breakage, not for normal alternate paths. Never use empty catch to turn failure into fake success. Model expected cases in the main flow (Special Case / default).

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

If both branches are normal business outcomes (e.g. regional tax vs default tax), do not throw to select the default. Return a default rule or Special Case object. Catch-as-if hides control flow and trains fear of exceptions.

**Swallowing:** `catch (e) {}` and `catch (() => [])` are forbidden. They violate honest failure (03, 13). If work is truly best-effort, you still **log with context** and document **why** ignore is safe (08) — or rethrow after translation. Prefer designing so no catch is needed.

---

## Rules

- Never use try/catch for expected branching — use defaults / Special Case / explicit `if`.
- Reserve exceptions for invariants broken, I/O failure, impossible states (03).
- Empty catch is always wrong.
- Catch that returns empty/success-on-failure is always wrong.
- Best-effort paths: log + short why-comment, or rethrow domain error — never silent.
- Boundary handlers (12) catch to **translate and rethrow/report**, not to pretend success.

---

## Bad vs good

### Bad
```text
try:
  rule = lookup(region)
catch:
  rule = STANDARD

try:
  return fetchOrders()
catch:
  return []   // failure looks like “no orders”
```

### Good
```text
rule = lookupOrDefault(region, STANDARD)

orders = fetchOrders()  // throws on failure; empty list only when fetch succeeded with zero rows
```

---

## Review checklist

- [ ] Is this catch handling a planned path or a real failure?
- [ ] Would a default value or Special Case remove the try?
- [ ] Can any failure path look identical to successful empty?

---

## Related standards

- 03 Exceptions over error codes
- 37 Algorithm breathes
- 13 Stop returning null
- 12 Define the contract first

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
