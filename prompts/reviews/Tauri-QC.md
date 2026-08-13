> **Harness (required):** Obey `skills/review/audit/SKILL.md`. Scope must be explicit (paths or flows).
> Default mode is `report-only`. Do not edit unless mode is `fix`.
> Read project `AGENTS.md` and only the sections of `frameworks/Tauri.md` you need.
> Discover verify commands from the project (cargo/npm only if this repo uses them).

# Tauri QC (usability + bug audit)

You are auditing a Tauri desktop app for usability issues and bugs.

1. Roleplay as 3 distinct user personas (e.g. first-time user, daily power user, user on a slow/offline connection) and walk through the app's core flows: [list flows, e.g. onboarding, file import, settings sync].
2. For each persona, log specific friction points: confusing UI states, inconsistent behavior, things that feel off, anything that breaks expectations of a native desktop app (window controls, file dialogs, drag-and-drop, system tray, auto-updater, permissions prompts).
3. For each issue logged, search the actual codebase to find the relevant frontend component and/or Rust command/IPC handler.
4. Classify each issue as:
   - **CONFIRMED BUG**, code clearly causes this behavior (cite file + line)
   - **LIKELY BUG**, strong circumstantial evidence, but could not fully trace
   - **NOT REPRODUCIBLE IN CODE**, looked but found no supporting evidence; flag as possibly perception/expectation
   - **WORKING AS INTENDED**, code shows this is deliberate; note if it is still a UX problem worth revisiting

## Output format

A table: Persona | Issue | Severity (low/med/high or harness blocker/high/medium/low) | Status | File/line reference | Notes

End with harness report shape: coverage list, exclusions, go/no-go for shipping the scoped surface.

## Constraints

- Do not speculate about Rust/JS code you have not opened.
- Prioritize Tauri-specific failure modes (permission scope errors, IPC serialization mismatches, window lifecycle bugs, asset loading paths) over generic web UI nits.
- No silent full-repo rewrites. Mode `fix` only changes in-scope confirmed defects.
