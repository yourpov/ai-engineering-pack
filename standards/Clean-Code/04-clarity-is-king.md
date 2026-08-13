# 04. Clarity is king — no mental mapping

> **Rule:** Names must not require a mental dictionary. Never force readers to decode private abbreviations.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Every cryptic name is another entry on the reader's mental stack: “this is the URL… this is the user… this is permissions…” Four lines, four translations — the brain becomes a lookup table instead of following logic.

**Avoid mental mapping.** No one should have to remember what your abbreviations stand for.

**Acronyms the team already speaks as words** (URL, ID, HTTP, API) are fine — see 39. Private shorthands (`tx`, `mgr`, `ctx2`) are not.

---

## Rules

- Use full intention-revealing words: `transaction`, not `tx`, unless the team already says that abbreviation as a domain word.
- Loop variables in long methods still need meaning when the body is non-trivial.
- Function parameters (`price`, `tax`, `quantity`) must make formulas readable as English.
- If you need a comment to explain a name, the name is wrong.
- Pronounceable team-spoken acronyms: 39. Searchable scope-length names: 30.

---

## Bad vs good

### Bad
```text
for t in list:
  if valid(t): process(t)   // t? time? tax? text?
total = p + t * q
```

### Good
```text
for transaction in pendingTransactions:
  if isValid(transaction): process(transaction)
lineTotal = price + tax * quantity
```

---

## Review checklist

- [ ] Can a new teammate read this block without asking what abbreviations mean?
- [ ] Would saying the name out loud make sense in a review?

---

## Related standards

- 06 Intention-revealing names
- 39 Pronounceable names
- 30 Searchable names

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
