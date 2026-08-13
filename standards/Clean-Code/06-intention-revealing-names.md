# 06. Intention-revealing names (written once, read hundreds of times)

> **Rule:** Names answer why it exists, what it does, and how it is used — without a comment.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Code is written once and read hundreds of times. A single letter (`d` — days? data? distance?) fails the readability test and forces every developer into a scavenger hunt.

**Intention-revealing names** make filtering, pricing, and domain rules obvious at the point of use.

---

## Rules

- The name answers why / what / how without a comment.
- Filtering variables describe the criterion: `activeAccounts`, not `list2`.
- Calculations use domain words, not single letters.
- Scope-length rule: wider scope → more descriptive name (see 30).

---

## Bad vs good

### Bad
```text
const d = items.filter(x => x.a)
const t = p * q * (1 + r)
```

### Good
```text
const activeItems = items.filter(item => item.isActive)
const lineTotal = unitPrice * quantity * (1 + taxRate)
```

---

## Review checklist

- [ ] Does the name explain intent without opening other files?
- [ ] Would I be happy reading this at 2 AM in an incident?

---

## Related standards

- 04 Clarity is king
- 30 Searchable names
- 39 Pronounceable names

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
