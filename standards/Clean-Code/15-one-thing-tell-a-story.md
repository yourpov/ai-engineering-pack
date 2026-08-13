# 15. One thing — functions that tell a story

> **Rule:** A function does one thing. It cannot be split into differently named sections. It reads top-down as a short story.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Multiple things ⇒ multiple reasons to change, break, test, and confuse. One thing ⇒ one purpose, one test focus, a trusted building block.

**Tests for “one thing”:**

1. You cannot divide the body into sections that deserve different names.  
2. You cannot extract a helper whose name is more than a restatement of the implementation.

---

## Rules

- If you can comment section headers inside a function, extract those sections.
- Keep one level of abstraction per function (see 27).
- Order steps so the reader’s eye follows the story: setup → work → result.
- Prefer many small named functions over one multi-act epic.

---

## Bad vs good

### Bad
```text
function handleOrder(order):
  // validate
  // charge card
  // send email
  // update analytics
  // write audit log
```

### Good
```text
function handleOrder(order):
  validate(order)
  charge(order)
  notifyCustomer(order)
  recordAnalytics(order)
  writeAudit(order)
```

---

## Review checklist

- [ ] Can I describe the function without using “and”?
- [ ] Are section comments trying to apologize for size?

---

## Related standards

- 18 Stepdown rule
- 26 Extract large functions
- 38 Apology comments

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
