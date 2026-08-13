# Default Stack

> **Agent load: optional, personal defaults only.** Do **not** treat this file as required
> architecture for every project. Project `AGENTS.md`, README, and lockfiles win.
> Use this when starting something new and the human has not named a stack yet.

Services and tools I reach for by default. Not every project needs all of them.
Start with the essentials, add the others when you actually feel the need.

---

## Essentials (almost every project)

| Concern | Pick | Why |
|---|---|---|
| **Coding (AI)** | Claude Opus | Long-context reasoning, best at large refactors |
| **Version control** | GitHub | Ubiquitous, great PR/Actions |
| **Backend / DB / Auth** | Supabase | Postgres + realtime + RLS + auth in one box |
| **Deploy (web)** | Vercel | Zero-config for Next.js/SvelteKit; preview URLs per PR |
| **Domain** | Namecheap | Cheap renewals, decent UI |
| **DNS / edge** | Cloudflare | Free tier is the best; proxy, WAF, R2 if needed |

---

## When the product ships to real users

| Concern | Pick | Why |
|---|---|---|
| **Auth (non-Supabase)** | Clerk | If I need social login + orgs + passwordless without writing it |
| **Payments** | Stripe | Checkout, subscriptions, invoicing. Sandbox is painless |
| **Emails (transactional)** | Resend | Simple API, good DX, inbox delivery is solid |
| **Uptime monitoring** | UptimeRobot | Free tier covers most side projects |
| **Error tracking** | Sentry | Source-map support, releases, performance |
| **Product analytics** | PostHog | Self-host or cloud; funnels + session replay + flags |

---

## When the product gets AI features

| Concern | Pick | Why |
|---|---|---|
| **Vector DB** | Pinecone | Managed, serverless tier is enough to start |
| **LLM API** | Anthropic (Claude) | Primary. Fall back to OpenAI for niche cases |

---

## AI design + dev tooling (for building with AI)

Install/configure these in your Claude Code environment. Most live in `.mcp.json`.
Setup syntax for all of it: [code.claude.com/docs](https://code.claude.com/docs).

| Tool | What it does | Link |
|---|---|---|
| **Google Stitch MCP** | UI mockup generation via MCP | https://stitch.withgoogle.com/docs/mcp/setup |
| **Gemini Nano Banana 2** | Visual asset / mockup generation | https://gemini.google/overview/image-generation/ |
| **UI/UX Pro Max** | 67 styles, 96 palettes, 57 font pairings, design-system intelligence | https://github.com/nextlevelbuilder/ui-ux-pro-max-skill |
| **21st.dev** | Reactive / 3D component asset library | https://21st.dev |
| **n8n-MCP** | 1000+ n8n node knowledge for production automations | https://github.com/czlonkowski/n8n-mcp |
| **Obsidian Skills** | AI-powered second brain (markdown / canvas / CLI) | https://github.com/kepano/obsidian-skills |
| **Claude-Mem** | Persistent memory compression across Claude Code sessions | https://github.com/thedotmack/claude-mem |
| **Get Shit Done (GSD)** | Spec-driven dev: interview → plan → execute → verify | https://github.com/gsd-build/get-shit-done |
| **Superpowers** | Auto-enforces TDD, debugging, code review, planning | https://github.com/obra/superpowers |
| **Awesome Claude Code** | Curated reference of 200+ Claude Code tools | https://github.com/hesreallyhim/awesome-claude-code |
| **MengTo/Skills** | Skills collection worth reading before writing your own | https://github.com/MengTo/Skills |
| **Skills CLI** | Installs skills from any GitHub repo across 75+ agents; `npx skills find` searches the community registry | https://github.com/vercel-labs/skills |
| **Token Shield** | Measures real token usage from your session transcripts: cache hit ratio, first-request cost, subagent output share | https://github.com/khalilmaaouni/token-shield |

For components, icons, illustrations, gradients, and 3D assets, see
[`design/README.md`](./design/README.md).

---

## Picking a stack for a new project

Work through this in order; stop as soon as you have enough:

1. **What language fits the problem?**
   - Systems / performance / Windows native → C++ or Rust
   - Web / desktop (Tauri) / CLI tooling → TypeScript
   - Data / ML / scripting → Python
   - Concurrent services → Go
   - Roblox game → Luau (`languages/Luau.md`)
2. **What's the surface?** Web app, desktop, service, bot, game, library.
3. **Do I need persistence?** If yes → Supabase by default (unless the domain already picked another DB).
4. **Do I need auth?** If yes → Supabase Auth, escalate to Clerk if complex.
5. **Do I need to take money?** If yes → Stripe.
6. **Do I need AI features?** If yes → Anthropic + Pinecone if there's retrieval.
7. **Where does it live?** Vercel for web, GitHub Releases for desktop binaries, npm / crates / PyPI for libraries.
8. **How do I know it's healthy in prod?** Sentry + UptimeRobot + PostHog (if there are users).

---

## Things I deliberately don't default to

- **Firebase**, vendor lock is painful, cost curve is steep. Supabase covers it.
- **Heroku**, Vercel + Supabase is cheaper and better for my usual web stack.
- **AWS from day one**, start on managed Supabase/Vercel; only move to AWS when you hit real scale.
- **Custom auth**, never roll your own. Clerk or Supabase.
- **Mongo**, unless there's a specific reason, Postgres (via Supabase) wins.
- **Monorepos from day one**, one repo per project until coupling makes it painful.

---

## Notes

- These are **defaults**, not rules. If a project needs something else, use it.
- Reassess every 6 months. The landscape moves; loyalty to a tool is worth nothing.
- Cost discipline: every managed service is a monthly bill. Turn off what you're not using.
- Agents: if `AGENTS.md` or an existing lockfile conflicts with this file, **ignore this file**.
