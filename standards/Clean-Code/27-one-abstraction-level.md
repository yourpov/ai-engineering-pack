# 27. Do not mix high-level and low-level details

> **Rule:** One level of abstraction per function. Mixing policy with bit-twiddling obscures intent.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

High-level steps (“charge customer, send receipt”) next to low-level details (string index math, raw bytes) force context switching. Readers cannot skim.

---

## Rules

- A function’s lines should all be at roughly the same altitude.
- Push encoding, parsing, and I/O into lower helpers.
- Keep orchestration high; keep mechanisms low.

---

## Bad vs good

### Bad
```text
function onboard(user):
  charge(user)
  // low-level suddenly
  bytes = user.email.charCodeAt(0) & 0xff
  sendReceipt(user)
```

### Good
```text
function onboard(user):
  charge(user)
  sendReceipt(user)

function emailPartitionKey(email): ...
```

---

## Review checklist

- [ ] Do any lines feel like a sudden zoom into machinery?
- [ ] Can I move that machinery down a level?

---

## Related standards

- 15 One thing
- 26 Extract
- 18 Stepdown

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
