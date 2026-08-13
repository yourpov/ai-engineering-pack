> **Harness (required):** Obey `skills/review/audit/SKILL.md` before this prompt.
> Scope must be explicit (paths/packages). Default mode is `report-only`.
> Do not edit code unless mode is `fix`. Exclude vendored/minified/binary unless scoped in.
> Read project `AGENTS.md` for owned code and verify commands.
> End report: coverage list, findings with severity, verification, go/no-go.

---

# Domain: Performance and scalability

Assume traffic or player count grows **10x** with no infra change. Within scope, flag:

- N+1 queries
- Unbounded loops/recursion over user-controlled input
- Missing pagination
- Sync work that should be async/batched
- Unbounded caches or listeners never removed
- O(n²) or worse on data that scales with users/players

For each: file, load condition, failure mode, severity, fix (applied only if mode is `fix`), and approximate load level where it becomes real.

**End report:** hotspots reviewed, issues, go/no-go under 10x.
