---
name: claude-folder
description: >
  Use when setting up or changing a project's Claude Code configuration: what
  belongs in CLAUDE.md versus CLAUDE.local.md, what each directory under .claude/
  actually does, how hooks get registered, and what a minimal new project needs.
  Trigger on "set up .claude", "add a hook", "what goes in settings.json", "make a
  slash command", "add a project skill", "why isn't my hook firing", "CLAUDE.md vs
  CLAUDE.local.md", or scaffolding agent config for a new repo. Tooling reference,
  not a code standard.
---

# The .claude/ Folder

What each part does, what to actually create, and the traps that waste an
afternoon. Harness details change between releases, so treat this as the map and
the official docs as the authority on exact key names.

**Reference:** [code.claude.com/docs](https://code.claude.com/docs), or `/help` in the tool.

## The two-file core

Most projects need exactly two files. Everything else is optional.

| File | Tracked? | Role |
|---|---|---|
| `CLAUDE.md` | Yes | Project instructions every agent loads. Your rules live here |
| `CLAUDE.local.md` | No | Personal overrides for your machine. Gitignored |

**Keep `CLAUDE.md` short.** Somewhere under 200 lines is a good ceiling. It loads
into every single session, so every line you add is rent paid forever. Push depth
into linked files and reference them by path; the agent will open what it needs.

`CLAUDE.md` is only part of that rent. Every installed skill's description, every
MCP server's tool schemas, and every hook's output ride along in the same
always-loaded block, billed on every call whether they fire or not.
[`../context-budget/SKILL.md`](../context-budget/SKILL.md) covers the whole floor
and how to measure it.

A common and good split: `AGENTS.md` at the root as the portable contract that any
harness reads, and `CLAUDE.md` for the parts specific to this tool. Cross-reference
rather than duplicating, or the two will drift.

## Directories under .claude/

| Path | What it is |
|---|---|
| `skills/` | Model-invokable. One folder per skill containing `SKILL.md`. Claude chooses them by matching the task against the `description`, so descriptions decide whether they fire |
| `agents/` | Sub-agent definitions, one markdown file each. Each runs in its own context window. See `Sub-Agents.md` before wiring these up |
| `commands/` | Slash commands, one markdown file each. User-invoked, not model-invoked. **Not legacy**, despite what some guides claim |
| `hooks/` | Where hook scripts live. Living here does **not** make them run (see below) |
| `rules/` | Additional rule files. Can be scoped to a glob so they load only for matching paths. **An unscoped rule file loads in every session** |
| `output-styles/` | Alternate response shapes, opt-in |
| `settings.json` | Tracked, shared config: permissions, env, model, statusline, and the hook registry |
| `settings.local.json` | Your machine only. Gitignore it |

At the repo root, `.mcp.json` declares MCP servers. Root only.

## The hook trap

**Dropping a script into `.claude/hooks/` does nothing.** Hooks fire only when
registered in `settings.json` under the `hooks` key with an event and a matcher.
The directory is a convention for where to keep the scripts, not a trigger.

If a hook is not firing, check registration before you debug the script.

Hooks are the only mechanism that *enforces* anything. A rule written in
`CLAUDE.md` is advisory: it holds while the agent remembers it. A hook is
mechanical and runs regardless. So put taste and context in `CLAUDE.md`, and put
the one or two rules you actually cannot afford to have violated into a hook.

Worth a hook: blocking edits that reintroduce a banned pattern, formatting on
write, refusing destructive shell commands, notifying you when a long run ends.

## Gitignore

Ignore `.claude/settings.local.json` **in the repo's own `.gitignore`**, not just
your global one. A global ignore protects the machine it is on and nobody else.
Check with `git check-ignore -v <path>`; it prints which ignore file matched, so
you can see whether the protection travels with the repo.

## Minimum viable setup

For a new project, in order:

1. `CLAUDE.md` with the project's rules. Short.
2. `.gitignore` entry for `.claude/settings.local.json`.
3. `.claude/settings.json` only once you have a permission or hook worth sharing.

Stop there. Add `skills/`, `agents/`, `commands/`, or `output-styles/` when a real
need appears, not because the directory exists. Empty scaffolding is a cost with
no return, and a folder full of half-written skills makes the agent worse at
picking the right one.
