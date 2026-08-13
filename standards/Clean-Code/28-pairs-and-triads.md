# 28. Argument pairs, triads, and naming

> **Rule:** Unrelated pairs confuse order. Triads are harder. Encode order in names or own the relationship.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Some pairs are one idea (x,y). Many pairs are unrelated peers — force an owner. Triads (especially assert-style message/expected/actual) need rigid conventional order or named parameters.

---

## Rules

- If two args form one concept, pass a point/range/object.
- If two args are unrelated, make one the receiver/owner method.
- Avoid three-arg calls when a struct reads clearer.
- Use argument names (named params / options objects) when order is not obvious.

---

## Bad vs good

### Bad
```text
assertEqual(expected, actual, message)  // easy to swap
```

### Good
```text
assertEqual({ expected, actual, message })
// or language-native named arguments
```

---

## Review checklist

- [ ] Could a caller swap two args by mistake?
- [ ] Does the name read as a sentence with its arguments?

---

## Related standards

- 02 Single-arg patterns
- 23 Low arity
- 04 Clarity

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
