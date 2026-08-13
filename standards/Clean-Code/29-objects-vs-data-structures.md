# 29. Pick a side — object or data structure, not both

> **Rule:** Objects hide data and expose behavior. Data structures expose data and skip behavior. Hybrids lose both advantages.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Hybrids (getters everywhere **and** business methods) trap logic inside while also leaking representation. DTOs carry data. Active records may have save/delete — keep business rules elsewhere.

---

## Rules

- Object: private data + meaningful operations.
- Data structure / DTO: public fields (or simple accessors) + no business logic.
- Active record: persistence navigation only; pricing/rules live in domain services/objects.
- When you need a new type of behavior, polymorphism on objects beats adding switches on exposed fields.

---

## Bad vs good

### Bad
```text
class Product:
  getPrice / setPrice
  calculateDiscount(...)  // hybrid
```

### Good
```text
// DTO
ProductRecord { price, sku }

// Object
PricingPolicy.discount(product, user)
```

---

## Review checklist

- [ ] Is this type trying to be both a bag of fields and a behavior hub?
- [ ] Would splitting DTO + policy clarify change?

---

## Related standards

- 21 Expose behavior
- 16 Not everything is a class
- 24 Bury the switch

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
