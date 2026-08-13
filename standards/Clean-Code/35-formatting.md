# 35. Formatting — vertical density and hierarchy

> **Rule:** Blank lines separate concepts; related lines stay dense; indentation shows hierarchy. Automate the rest.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Even good names fail inside a wall of text. Vertical spacing groups ideas. Over-spacing disconnects related lines. Indentation makes scope scannable. Use the project formatter for horizontal rules.

---

## Rules

- One blank line between conceptual paragraphs inside a function/file.
- No blank lines between tightly coupled statements (e.g. successive field assigns that form one unit).
- Rely on automatic formatters for width, commas, and import order.
- Team conventions win (11).

---

## Bad vs good

### Bad
```text
// either zero blank lines for 80 lines, or a blank line after every statement
```

### Good
```text
// group: inputs
// group: compute
// group: result / return
```

---

## Review checklist

- [ ] Can I scan the file by eye via vertical groups?
- [ ] Is the formatter enforced in CI or pre-commit?

---

## Related standards

- 18 Stepdown
- 11 Team conventions
- 25 Tools

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
