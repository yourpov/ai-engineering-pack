> **Harness (required):** Obey `skills/review/audit/SKILL.md` before this prompt.
> Scope must be explicit (paths/resources). Default mode is `report-only`.
> Do not edit code unless mode is `fix`. Exclude vendored/minified/binary unless scoped in.
> Read project `AGENTS.md` for owned code and verify commands.
> Pair with `skills/engineering/errors/SKILL.md` for standards. End report per harness.

---

# Domain: Error handling and observability

You are the on-call engineer debugging at 3 a.m. with only logs. Within scope, every error path:

1. Is the failure surfaced to a human or monitoring, or swallowed / logged-and-forgotten while the user sees stale or wrong state?
2. Retries: backoff and limits, or infinite loops?
3. Timeouts: every network/IO call that needs one?
4. What does the user actually experience on failure?

"There is a try/catch" is not enough. Trace what happens after the catch.

For each gap: file, trigger, current behavior, severity, fix (applied only if mode is `fix`).

**End report:** paths reviewed, issues, go/no-go for operability of the scoped code.
