> **Harness:** Obey `skills/review/audit/SKILL.md` for reviews. For **builds**, still: read project
> `AGENTS.md` if present; prefer extending an existing repo; discover verify commands
> from the project; default to implementing only what the human asked (not the full
> deliverables list unless they want a full scaffold).

# Discord Bot Prompt

Build a Discord bot. Keep command handling, business logic, and the Discord client separate, so the logic can be tested without a gateway connection.

---

## Language selection

Choose language and load the matching file (section-load: structure, errors, project prompt):

- **Bun/Node.js** → `languages/Bun-Node.md`
- **Python** → `languages/Python.md`
- **Go** → `languages/Go.md`

Also open relevant sections of `standards/Principles.md` and `skills/engineering/errors/SKILL.md` if failure handling is in scope. Craft: `skills/engineering/craft/SKILL.md`.

---

## Scope rules

1. If the human named a single feature (e.g. standup reminders), implement **that**, not a full moderation suite.
2. If the repo already exists, extend it. Do not `bun init` / re-scaffold unless greenfield is explicit.
3. Secrets only in env / config examples with placeholders. Never commit real tokens.

---

## Architecture (when scaffolding or reviewing)

- Follow structure from the language architecture file
- Command handler pattern (no giant if/else chains)
- Event-driven registration
- Service layer for business logic
- Data access behind a thin boundary

**Typical layout (adapt to language):**
```
bot/
├── commands/
├── events/
├── services/
├── data/
├── models/
└── index
```

**Security (always)**

- Validate all user input
- Permission checks **before** privileged actions (server-side / bot process, not only UI)
- Rate limiting / cooldowns on expensive commands
- Token in env only
- Sanitize content before DB write

---

## Setup (greenfield only)

**Bun/Node.js:**
```bash
bun init
bun add discord.js dotenv
mkdir commands events services data
```

**Python:**
```bash
python -m venv venv
# activate venv for your OS
pip install discord.py python-dotenv
mkdir commands events services data
```

**Go:**
```bash
go mod init botname
go get github.com/bwmarrin/discordgo
mkdir -p cmd/bot internal/commands internal/events internal/services internal/data
```

---

## Environment (placeholders)

```
TOKEN=
PREFIX=!
DATABASE_URL=
```

---

## Deliverables (full scaffold only)

When the human asked for a full bot scaffold:

1. Modular structure per language file
2. Command registration system
3. Commands the human specified (if none, ask; do not invent a full mod suite)
4. Event handlers needed for those features
5. Permission checks where actions are privileged
6. Cooldowns on expensive commands
7. Config via env
8. Error logging with context
9. README with setup

---

## Validation checklist

- [ ] Scoped to what was asked
- [ ] Permission checks on privileged paths
- [ ] Rate limiting where abuse is realistic
- [ ] Token only in environment
- [ ] Names and structure match language file + Craft
- [ ] Verify commands from project AGENTS.md / README (or listed manual checks)
