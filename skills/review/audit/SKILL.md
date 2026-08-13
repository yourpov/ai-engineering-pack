---
name: audit
description: >
  Universal operational harness for any review, security pass, or audit prompt.
  Always load before running prompts/reviews/* or prompts/domains/* or a full PR
  review. Defines scope,
  mode (report vs fix), exclusions, severity, verification, and output shape.
  Stack-agnostic. Project-specific threat models live in prompts; this file is
  the safety contract that keeps agents from thrashing an entire monorepo.
---

# Audit Harness (Universal)

Every audit prompt and production review **must** obey this harness. Domain
prompts add threat models and checklists on top of it. They do not override
scope, mode, or verification rules here.

---

## 1. Before any file is opened

Collect or ask for:

| Field | Required | Default if human is silent |
|---|---|---|
| **Scope** | Yes | Stop and ask. Never invent "the whole repo" unless explicitly told. |
| **Mode** | Yes | `report-only` |
| **Exclusions** | Strongly preferred | Third-party/vendored trees, minified bundles, generated code, binary/stream assets, lockfile-only noise |
| **Verify commands** | Yes if mode is `fix` | Discover from project `AGENTS.md` / README. If none exist, state "no automated verify" and list manual checks. |
| **Approval for risky edits** | When needed | Do not force-push, rewrite history, edit secrets, or mass-change vendored code without explicit approval |

**Scope examples (good):** one resource folder, one package, a PR diff, a named file list. 
**Scope examples (bad):** "everything under src," "all resources," with no exclusions and mode `fix`.

If scope is missing, output:

```text
BLOCKED: audit scope not set.
Provide: paths or package names, mode (report-only | fix), and any exclusions.
```

Do not proceed past that message.

---

## 2. Mode

### `report-only` (default)

- Trace and document findings.
- Do not edit production code.
- Fixes are recommendations with enough detail to implement later.

### `fix`

- Only after scope is explicit.
- Fix in-scope defects only. No drive-by refactors, renames, or style passes.
- One concern per change set when practical.
- After fixes, run project verify commands and paste real output (or honest failure).

### `fix` is never implied by "review," "audit," or "check."

---

## 3. Exclusions (apply unless the human overrides)

Do **not** treat as first-class audit targets:

- Vendored / third-party dependencies and plugins not owned by the project
- Minified or bundled `dist` output (unless the task is specifically "patch this known vendored bug" and approval is clear)
- Generated code, caches, lockfile-only diffs
- Binary assets, maps, media, compiled streams
- Secrets files and live config (report exposure risk; do not print secret values)

Own project code and thin wrappers around vendors **are** in scope when listed.

---

## 4. Evidence rules

- Do not sample "representative" files and claim full coverage.
- Do not rely on earlier sessions as proof. Re-open files when asserting.
- Unhappy paths count: null/malformed input, double-submit, disconnect mid-op, permission denied, timeout.
- Shared mutable state: prove ordering safety or flag as unverified.
- "Client already checks this" / "UI prevents this" is not validation on a hostile client or untrusted caller.
- If you have not traced a path, label it **unverified**, not safe.

---

## 5. Severity

| Level | Meaning |
|---|---|
| **blocker** | Data loss, security break, economy break, crash on common path, ship-stopper |
| **high** | Exploitable or likely production incident under realistic load/abuse |
| **medium** | Real defect, limited blast radius or harder to trigger |
| **low** | Smell, missing polish, dead code, docs drift |

---

## 6. Finding format (every issue)

For each finding:

1. **Location**, path and symbol/event/handler name 
2. **Trigger**, exact scenario (who/what calls it, with what inputs) 
3. **Effect**, what goes wrong 
4. **Severity**, blocker / high / medium / low 
5. **Evidence**, what you opened and what you traced 
6. **Fix**, concrete change (applied if mode is `fix`, proposed if `report-only`) 
7. **Verify**, how you confirmed, or "not run" with reason 

---

## 7. Required end report

Always end with:

1. **Scope actually covered**, paths/modules reviewed (honest list) 
2. **Skipped / excluded**, and why 
3. **Findings**, table or list with severity 
4. **Fixes applied**, only if mode was `fix` 
5. **Verification**, commands run and outcomes, or explicit "none" 
6. **Go / no-go**, ship recommendation for the scoped change; name blockers 

Never "all green" without the coverage list and evidence.

---

## 8. How domain prompts use this file

1. Read this harness. 
2. Read project `AGENTS.md` if present. 
3. Apply the domain prompt's checklist **inside** the agreed scope and mode. 
4. Domain prompt may tighten severity (e.g. treat money dupes as blocker). It may not widen scope or switch to `fix` silently.

---

## 9. Pairing

| Doc | Role |
|---|---|
| `prompts/reviews/PR-Review.md` | Production-minded full review posture |
| `prompts/reviews/*` | Generic audits (security, perf, errors, Tauri QC) |
| `prompts/domains/*` | Domain threat models you add for your own stack |
| `prompts/scaffolds/*` | Greenfield product builds (not audits) |
| `skills/engineering/errors/SKILL.md` | Failure handling and user-facing copy |
| `skills/engineering/uncle-bob/SKILL.md` | Craft methodology |
| `standards/Principles.md` | Long optional handbook (section-load only) |
| Project `AGENTS.md` | What is owned code, secrets, verify commands |
