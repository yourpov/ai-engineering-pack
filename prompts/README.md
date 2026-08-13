# prompts/ index

Paste-in task prompts. Not installable skills (see `../skills/`).

| Folder | Role | Pair with |
|---|---|---|
| **`reviews/`** | Audits, security, performance, production go/no-go, Tauri QC | Always `../skills/review/audit/SKILL.md` first |
| **`scaffolds/`** | Greenfield product builds + README voice templates | Matching `../languages/*` / `../frameworks/*` |
| **`domains/`** | Extension point. Add `domains/<your-stack>/` for threat models that only apply to one platform. Do not mirror generic Security/Error/Perf files that already live in `reviews/`. | Always `../skills/review/audit/SKILL.md` first |
| **`workflow/`** | How the agent should behave before and during a task, independent of stack | Any prompt above |

## Rules

1. **One prompt per task.** Do not paste the whole tree.
2. **Reviews default to report-only** until the human sets scope and mode `fix`.
3. **Scaffolds** implement only what was asked; prefer extending an existing repo.
4. **Paths** in these files assume this pack layout (or the same layout under project `docs/`).

## Quick picks

| Want | Open |
|---|---|
| Stop the agent guessing before it codes | `workflow/Before-Implementing.md` |
| Ship review | `reviews/PR-Review.md` |
| Security pass | `reviews/Security-Audit.md` |
| Discord bot | `scaffolds/Discord-Bot.md` |
| Website | `scaffolds/Website.md` |
| README voice | `scaffolds/Readme-Voice.md` + skill `Create-Readme.md` |
| Generic security (any stack) | `reviews/Security-Audit.md` |
