# Code Review and Security Audit

The deep pass over code that is about to ship. Security first, then architecture, then
craft. That order is not a preference. A clean abstraction sitting on top of an auth hole
is still an auth hole.

If you want a pre-merge go or no-go rather than a full sweep, use
[`PR-Review.md`](./PR-Review.md) instead.

## Before you open a file

1. Read `skills/review/audit/SKILL.md`. Scope is required, and the mode is `report-only` until someone
   says otherwise.
2. Read the `languages/*` and `frameworks/*` file that matches this stack.
3. Open only the sections of `standards/Principles.md` you need. Loading the whole handbook
   spends the context you need for the code.
4. Use `skills/engineering/craft/SKILL.md` for naming, comments, and prose. Do not run AI-detector scoring.
5. Take verify commands from the project's `AGENTS.md` or README. If you cannot find them,
   ask. Do not invent a toolchain.

## The four phases

| Phase | What you are looking for |
|---|---|
| **1. Security** | Anything an attacker can reach |
| **2. Architecture** | Whether the shape of this code survives the next feature |
| **3. Quality** | Whether the next person can read it |
| **4. Remediation** | The exact fix, applied only when the mode is `fix` |

Work them in that order. Do not start proposing refactors while there is still unreviewed
auth code.

---

## Phase 1: Security

Assume every input is hostile and every client is lying. The question is never whether
someone would do this. It is whether they can.

### Authentication and authorization

- [ ] Hardcoded credentials, API keys, or tokens
- [ ] Weak password rules
- [ ] Endpoints with no authentication
- [ ] Endpoints with authentication but no authorization check behind it
- [ ] Session fixation, or JWTs with a weak secret or no expiry

### Input validation and injection

- [ ] SQL and NoSQL injection
- [ ] Command injection
- [ ] Code injection through `eval` or `Function`
- [ ] Path traversal
- [ ] XSS, reflected, stored, and DOM-based
- [ ] CSRF: missing tokens, or state-changing requests behind a GET
- [ ] File upload handling
- [ ] Unvalidated redirects

### Data protection

- [ ] Sensitive data in logs or shipped in the client bundle
- [ ] Missing encryption at rest or in transit where it is required
- [ ] API responses returning more PII than the caller needs
- [ ] Insecure deserialization

### Trust boundaries

- [ ] Client-side checks treated as the only validation
- [ ] IDOR: object IDs accepted without checking who owns them
- [ ] SSRF, open proxies, or webhooks accepted without verification

---

## Phase 2: Architecture

Look for the change that has not happened yet. Most architecture problems cost nothing
today and everything the first time someone tries to extend them.

- [ ] Dependencies point inward. Policy does not import drivers
- [ ] Every module has one job you can name in a sentence
- [ ] No god objects, and no simple change that requires editing six files
- [ ] Shared mutable state is either safe under concurrency or flagged as unsafe

---

## Phase 3: Quality

- [ ] Names match the domain and the conventions already in the file (`skills/engineering/craft/SKILL.md`)
- [ ] Comments say why, not what
- [ ] Errors carry enough context to debug from a log line alone (`skills/engineering/errors/SKILL.md`)
- [ ] No dead code, debug leftovers, or commented-out blocks
- [ ] Tests exercise public seams. When the mode is `fix`, the verify commands actually run

---

## Phase 4: Remediation

Write every finding as: location, trigger, effect, severity, fix, and how it was verified.

| Severity | Means |
|---|---|
| **Blocker** | Do not ship |
| **High** | Ship only with a named owner and a date |
| **Medium** | Next sprint |
| **Low** | Fix it while you are in there |

Be specific about effect. "Could be exploited" is not a finding. "An unauthenticated GET to
`/api/orders/:id` returns another user's shipping address" is a finding, because someone can
act on it.

Close with the report shape from `skills/review/audit/SKILL.md`: what you covered, what you excluded and
why, findings by severity, verification, and a go or no-go.
