# skills/ index

One folder per skill, grouped by what the skill is for. Each folder holds a `SKILL.md`
carrying YAML frontmatter: a `name`, and a `description` full of trigger phrases. The
agent reads those descriptions and decides on its own when to load one. That is the
difference between this folder and [`../prompts/`](../prompts/README.md), which you paste
in by hand.

**The folder name is the skill's `name`.** That is not cosmetic. It is what lets you copy
a skill straight into `.claude/skills/` without renaming anything.

```bash
cp -r skills/review/audit ~/.claude/skills/
```

---

## `agent/` — how the agent runs

| Skill | `name` | Fires when |
|---|---|---|
| **[sub-agents](./agent/sub-agents/SKILL.md)** | `sub-agents` | Delegating work: model tiers, setting the model explicitly, writing a brief, commit authority, and when inline beats fan-out. Ends with a paste-ready block for your global `CLAUDE.md` |
| **[context-budget](./agent/context-budget/SKILL.md)** | `context-budget` | A session feels expensive or keeps hitting limits: the always-loaded floor, prompt-cache mechanics and what silently breaks them, and why output is the costliest token |
| **[commits](./agent/commits/SKILL.md)** | `commits` | One commit per task, staging only that task's files, message types, and never pushing unasked |
| **[claude-folder](./agent/claude-folder/SKILL.md)** | `claude-folder` | Setting up `.claude/`: CLAUDE.md vs CLAUDE.local.md, what each directory does, why a hook is not firing |

## `engineering/` — how code gets written

| Skill | `name` | Fires when |
|---|---|---|
| **[errors](./engineering/errors/SKILL.md)** | `errors` | How code should fail, and how the failure should read to a human. Uncle Bob's error handling plus user-facing copy |
| **[uncle-bob](./engineering/uncle-bob/SKILL.md)** | `uncle-bob` | Structure, seams, architecture, TDD discipline, professional scope calls. Methodology, not a report about your repo |
| **[craft](./engineering/craft/SKILL.md)** | `craft` | Naming, comments, structure, and prose judged for the human who has to debug it later |
| **[necessary-comments](./engineering/necessary-comments/SKILL.md)** | `necessary-comments` | Deciding whether a comment earns its keep, or whether a rename or extraction should replace it |

## `review/` — how work gets checked

| Skill | `name` | Fires when |
|---|---|---|
| **[audit](./review/audit/SKILL.md)** | `audit` | Any review, security pass, or audit. Sets scope, mode, exclusions, severity, and output shape. **Required before every `prompts/reviews/*` file** |

## `web/` — shipping websites

| Skill | `name` | Fires when |
|---|---|---|
| **[web-seo](./web/web-seo/SKILL.md)** | `web-seo` | Meta and OG tags, canonical URLs, robots, sitemap, JSON-LD, per-route metadata in an SPA |

## `writing/` — documentation

| Skill | `name` | Fires when |
|---|---|---|
| **[create-readme](./writing/create-readme/SKILL.md)** | `create-readme` | Writing a README from the real tree instead of a template. Every claim verified against an opened file |

---

[`../design/apple-design/`](../design/apple-design/SKILL.md) is built the same way but stays
opt-in, because an aesthetic should never fire on its own.

## Installing

Copy the skill folder. The name is already correct, so nothing gets renamed:

```bash
cp -r skills/review/audit .claude/skills/
cp -r skills/engineering/errors .claude/skills/
```

Use `~/.claude/skills/` instead of `.claude/skills/` to install for every project on the
machine. Category folders are for browsing this repo. Claude Code reads a flat
`skills/` directory, so copy the individual skill folders rather than the category.

If you would rather not install anything, copy the files under `docs/skills/` and point
your `AGENTS.md` at them.

Exact paths and frontmatter keys are documented at
[code.claude.com/docs](https://code.claude.com/docs), and they do change between releases.
[`claude-folder`](./agent/claude-folder/SKILL.md) covers the surrounding `.claude/` layout.

## Adding a skill

1. Pick the category it belongs to, or add one when nothing fits.
2. Create `skills/<category>/<name>/SKILL.md`, where `<name>` is kebab-case and matches
   the `name` in the frontmatter. Those two disagreeing is the most common reason an
   install silently does nothing.
3. Copy the frontmatter shape from [`engineering/errors`](./engineering/errors/SKILL.md).
4. Write the `description` for the matcher, not for a human. It should contain the phrases
   someone would actually type. A vague description means the skill never fires.
5. Keep the body portable. If it only makes sense on one stack, it belongs in
   [`../prompts/`](../prompts/README.md).
6. Add a row to the table above.

## Authority

Skills lose to [`../standards/Clean-Code/`](../standards/Clean-Code/README.md) on naming,
functions, comments, null, exceptions, Demeter, DRY, CQS, and structure. See
[`../standards/README.md`](../standards/README.md).
