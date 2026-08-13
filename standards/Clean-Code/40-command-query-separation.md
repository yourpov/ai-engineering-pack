# 40. Command–query separation

> **Rule:** A function either does something (command) or answers something (query) — never both. Queries are side-effect free. Commands must not smuggle a second question in their return value.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Mixing command and query means you cannot ask a question without mutating state. Callers cannot reason about safety. Split `attributeExists` from `setAttribute`.

**Returning a created entity is fine:** `user = createUser(...)` is a command that yields the thing it built — not a dual “did it exist?” query.

**Not fine:** `if setAttribute(...)` meaning both “I wrote” and “it already existed,” or `getOrCreate` that forces every “get” to possibly write. Split into query + command, or name the command honestly (`ensureUser`) and never use pure `get*` / `is*` / `has*` for mutation (01, 36).

---

## Rules

- Queries (`is`, `has`, `can`, `get`, `list`, `find`): no observable side effects; return info only.
- Commands: may mutate; prefer void, or return the artifact created/updated — not a second unrelated status question.
- Do not use a mutator’s return value to mean “already existed” vs “created” without an explicit, named type that is the command’s whole result — and still prefer split query + command when callers only needed to ask.
- Never name a command `get*` / `check*` / `is*` if it writes (01).
- Factory/create methods returning the new instance are commands, not queries — that is allowed.

---

## Bad vs good

### Bad
```text
if setAttribute(name, value):  // sets AND reports existence?
  ...
user = getOrCreate(id)        // every “get” may write
```

### Good
```text
if not attributeExists(name):
  setAttribute(name, value)

user = findUser(id)           // query
if user is missing:
  user = createUser(id)       // command returns User
```

---

## Review checklist

- [ ] Does asking the question change state?
- [ ] Can I mock/test the query without side effects?
- [ ] If a command returns a value, is it the artifact — not a hidden second query?

---

## Related standards

- 01 Side effects
- 02 Argument roles
- 36 Honest naming

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
