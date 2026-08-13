# 41. Team reads the code; world reads the docs

> **Rule:** Internal code should rarely need comments. Public APIs need docs: what, inputs, outputs, failures, example. Licenses stay standard and collapsible.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Two audiences: teammates reading source, and external users reading documentation. Well-named internal code is the teammate document. Libraries and public endpoints need proper docs and license headers without drowning the file.

---

## Rules

- Public functions/modules: document purpose, inputs, outputs, errors, example.
- Do not substitute huge comment banners for real docs sites / README sections when the audience is external.
- Use standard license text at file tops when required; let IDEs collapse them.
- Internal helpers: prefer self-explanatory code over mini-manuals.

---

## Bad vs good

### Bad
```text
// no README, no error docs, 40-line essay above every private helper
```

### Good
```text
// README + API docs for public surface
// private helpers named clearly, minimal comments
```

---

## Review checklist

- [ ] Could an external consumer use this without reading the implementation?
- [ ] Are licenses standard and non-noisy?

---

## Related standards

- 17 Why
- 19 Noise
- 07 Git for history

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
