# 08. Comments that protect from well-meaning mistakes

> **Rule:** Keep comments that warn, amplify non-obvious importance, or track blocked work — not narration.

**Status:** mandatory coding standard for every project that pulls this Architecture pack.  
**Source:** personal clean-code lesson pack (s4.codes-aligned).  
**Stack:** language-agnostic. Translate examples to the repo's language.

---

## Principle

Some comments protect the next developer:

- **Warnings** — slow tests, do not run in quick cycles  
- **Concurrency / safety** — “do not share this instance”  
- **Amplification** — a `+1 day` that looks random but is required for timezone edge cases  
- **TODOs with a real constraint** — waiting on a vendor bug, not “TODO: clean this later”

---

## Rules

- Warn when an “obvious optimization” would break production.
- Amplify rules that look arbitrary but are load-bearing.
- TODOs must name the external blocker or ticket — not delay craft indefinitely.
- Do not use protective comments as an excuse for bad structure (see 38).

---

## Bad vs good

### Bad
```text
// TODO: fix later
expiry = day + 1
```

### Good
```text
// +1 day so users west of UTC do not expire before local midnight
expiry = day + 1

// TODO(chrome#12345): remove once upstream ships fix for ...
```

---

## Review checklist

- [ ] Does this comment prevent a real, documented class of mistake?
- [ ] Is the TODO tied to something outside our control or a tracked issue?

---

## Related standards

- 17 Why not how
- 10 Comments must communicate
- 38 Structure over apology comments

---

## Agent notes

- Load **this file only** when the task is about this topic — do not stream the whole `Clean-Code/` folder.
- Prefer fixing structure (rename, extract, split, wrap) over adding comments or return codes.
- **Authority:** `standards/Clean-Code/` and Uncle Bob craft rules outrank `Principles.md`, `skills/engineering/errors/SKILL.md`, and other skills on these topics. Only a named exception in project `AGENTS.md` may override.
