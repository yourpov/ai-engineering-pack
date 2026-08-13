# 01. Side effects and honest functions

> **Rule:** A function must not do more than its name promises. Move side effects out of checks and queries.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

A **side effect** is any change outside the function's own scope: mutating globals, writing files, starting sessions, setting fields, network I/O, clearing carts.

When a name says *check*, *get*, *is*, *has*, or *validate*, callers trust it to **answer**. If it also mutates the world, reuse becomes unsafe (e.g. password check that restarts the session and empties the cart).

**Side effects make functions lie, and lies become bugs.** Keep the name honest; keep the effect at the call site or in a separately named command.

---

## Rules

- If the name is a question or check, return a value only — no hidden writes.
- If you need both answer and effect, split: `checkPassword` then `startSession` at the call site.
- Renaming to `checkPasswordAndStartSession` is more honest but still two jobs — prefer the split.
- Assigning a class/module field is still a side effect; treat it like one.
- Hooks that open sockets, write storage, or mutate module globals while “just reading” violate this standard.

---

## Bad vs good

### Bad
```text
function checkPassword(password):
  if password == secret:
    session = createSession()   // side effect hidden in a "check"
    return true
  return false
```

### Good
```text
function checkPassword(password):
  return password == secret

function startSession():
  session = createSession()

if checkPassword(input):
  startSession()
```

---

## Review checklist

- [ ] Can I call this twice safely if I only wanted information?
- [ ] Does the name match every mutation inside?
- [ ] Is there a pure alternative next to the effectful one?

---

## Related standards

- 40 Command–query separation
- 36 Honest naming
- 15 One thing / story

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
