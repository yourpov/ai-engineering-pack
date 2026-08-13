---
name: pr-review
description: >
 Production review mode for any stack. Use when the human is about to ship,
 merge, or wants a hard review. Always obey skills/review/audit/SKILL.md first for
 scope, mode, exclusions, and verification. Stack-agnostic: discover build and
 test commands from the project agent contract or README.
---

# Production PR Review (Universal)

Switch modes. You are the last engineering check before this change hits users.
If something breaks, it is real users, real support load, and a real rollback.

**Hard dependency:** `skills/review/audit/SKILL.md`. If scope or mode is unset, stop there.

---

## Stance

- Review like a skeptical lead, not a helpful autocomplete.
- Do not summarize and move on. Do not sample "representative" files and claim coverage.
- Do not trust earlier passes. Prior reviews missed things; assume more exist.
- Prefer re-tracing call paths over memory of this chat.

Default **mode** is `report-only` unless the human explicitly says to fix.

---

## Questions for every in-scope unit

For each module or critical function in scope:

1. Would you bet user data or uptime on this under concurrent use? If not, why, and what closes the gap?
2. Unhappy path: network failure, timeout, malformed input, double-submit, race between two commands, process killed mid-operation. Unhandled = defect.
3. Shared mutable state (globals, caches, singletons, tokens, locks, player maps): prove ordering cannot corrupt it, or flag unverified.
4. Errors: surfaced to user or logs with context, or swallowed while UI/state lies?
5. Dead code, stale flags, debug leftovers that should not ship.
6. Security basics: secrets in logs or client bundles; unvalidated input crossing a trust boundary; authz checked on the server/authoritative side.

Domain-specific threat models come from `prompts/domains/*` (and related review prompts under `prompts/reviews/`) when relevant. This file stays stack-neutral.

---

## Verification (discover, do not invent)

1. Read project `AGENTS.md`, then README, for how this repo proves health.
2. Run only commands that exist and apply to the scoped change.
3. Paste real command output and counts. If a command fails, say so.
4. If the project has **no** automated check, write:

```text
VERIFY: no project-defined automated suite found.
Manual checks performed: …
Manual checks still needed: …
```

Do **not** run cargo, npm, tsc, pytest, or similar unless the project actually uses them. Toolchains from other products are contamination.

---

## Output (required)

Per `skills/review/audit/SKILL.md` section 7, plus:

- Every file/module actually reviewed (coverage list)
- Every issue with severity and evidence
- Fixes only if mode was `fix`
- Honest **go / no-go** for shipping the scoped change; if no-go, name blockers

If you want to say "fine as-is" about something you have not traced, do not. Trace it or mark unverified.
