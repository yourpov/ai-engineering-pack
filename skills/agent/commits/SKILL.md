---
name: commits
description: >
  Commit discipline for agent-assisted work: one commit per completed task, staging
  only that task's files, conventional message types, and never pushing without being
  asked. Use when an agent is about to commit, when a session has produced a pile of
  unstaged changes across several tasks, or when deciding who owns commits between a
  parent session and its sub-agents. Trigger on "commit this", "should I commit",
  "write a commit message", "squash these", "git add", or any task finishing with
  changes on disk. Harness-agnostic.
---

# Commits

An agent can produce a week of changes in an hour. Without a commit policy you end up
with one enormous uncommitted diff that nobody can review, bisect, or roll back, and
the only way out is to read all of it at once.

The fix is boring and it works: commit each task as it finishes.

## The rule

**One commit per completed task, made as soon as the task is done.** Not batched at
the end of the session, not held until the human asks.

Give the agent standing authorization for this in your `CLAUDE.md`. Approval per commit
defeats the point, because the cost of the policy is entirely in the interruptions.

## Staging

**Stage the files that task touched.** `git add <paths>`, never `git add .`.

Agent sessions leave debris: scratch files, a log the agent wrote to think, a config it
edited while diagnosing something unrelated. `git add .` sweeps all of it into a commit
that claims to be one change. Naming paths is also a last check on scope, since a task
that touches files you did not expect is a task that did something you did not expect.

## Messages

```
<type>: <what changed>
```

| Type | For |
|---|---|
| `feat` | New behavior a user can observe |
| `fix` | Corrected behavior that was wrong |
| `refactor` | Same behavior, different structure |
| `docs` | Documentation only |
| `test` | Tests only |
| `chore` | Tooling, deps, config, housekeeping |
| `style` | Formatting with no logic change |

`feat: add user auth endpoint`. `fix: resolve null check in parser`.

Describe what changed, not what you did. "Update files" and "address feedback" are not
commit messages, they are the absence of one.

## Pushing

**Never push automatically.** Commits stay local until the human asks for them.

Local commits are cheap to amend, reorder, or drop. A pushed commit is a fact other
people build on, and undoing it is their problem as much as yours. That decision belongs
to the person who knows what else is in flight.

## Splitting

If a task touched several concerns, make several commits in sequence rather than one
that needs a paragraph to explain. The test is whether you can describe the change in
one line without using "and". If you cannot, it is more than one commit.

## Who commits

The session that owns the work owns the commits. See
[`sub-agents`](../sub-agents/SKILL.md): by default a sub-agent does not commit, because the
parent is the only one that can see whether the change is really finished. Never grant
commit authority to parallel agents sharing a worktree.

Verify before every commit. Read the diff you are about to stage and run the project's
verify gate. A clean commit history full of broken commits is worse than no history,
because it looks trustworthy.

---

## Drop this in your CLAUDE.md

```markdown
## Commits

Commit after each completed task, immediately. Do not batch tasks into one commit.
This is standing authorization, do not ask per commit.

- Stage only that task's files: `git add <paths>`, never `git add .`
- Message format: `<type>: <what changed>`
- Types: feat, fix, refactor, docs, test, chore, style
- Never push. Commits stay local until I ask
- If a task touched several concerns, split it into sequential commits
- Read the diff and run the verify gate before staging
```
