# 11. Declare variables close to use; group properties by team convention

> **Rule:** Locals live next to first use. Shared fields live in one agreed place. Formatting is a team decision — then stick to it.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Variables declared too early are baggage on the mental stack. Move them to the block that needs them.

Class/module properties are shared by design — do not scatter them “near the one method that uses them.” Pick top-or-bottom by team standard and stay consistent. Same for braces, quotes, tabs.

---

## Rules

- Declare locals at the smallest scope that needs them.
- Group fields in one place (usually top of type/module).
- Follow the project’s formatter and style config — no personal wars in PRs.
- Do not declare “just in case” variables for code that might never run.

---

## Bad vs good

### Bad
```text
function run():
  skippedLog = []
  report = null
  // ... 40 lines ...
  if cond: skippedLog.append(...)
  // ... 40 lines ...
  report = build()
```

### Good
```text
function run():
  // ...
  if cond:
    skippedLog = []
    skippedLog.append(...)
  // ...
  report = build()
```

---

## Review checklist

- [ ] Am I carrying unused names across large spans of code?
- [ ] Are shared fields discoverable in one place?

---

## Related standards

- 35 Formatting
- 15 One thing
- 26 Extract large functions

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
