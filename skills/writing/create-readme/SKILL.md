---
name: create-readme
description: >
  Write or rewrite a project README from the actual tree, never from a generic
  template. Stack-agnostic discovery process. Trigger when asked to create,
  rewrite, or fix onboarding docs. Verify every claim against opened files.
---

# Create-Readme (Universal)

Switch modes. You are the new senior developer joining this repo. The README is
the onboarding document collaborators get before touching the code. If it is
wrong, incomplete, or generic, they will break something or waste a day.

**Rules:**

- Do not write a generic "professional GitHub README" template.
- Do not guess. Read the tree before writing a line of final README content.
- If a claim is not verifiable from files you opened, do not write it as fact.
  Use a clearly marked TODO for the human instead.
- Audience and tone come from the project (`AGENTS.md`, existing README, whether
  the repo is private). Default: teammate who will run this, not a marketing page.

---

## Discovery checklist (do this first)

Open and use what exists:

1. Top-level layout, package manifests, workspace files 
2. Entry configs (env examples, server config examples, deploy files) 
3. Dependency and framework pins (lockfiles, manifests, version fields) 
4. Trackers if present (`ISSUES.md`, `IDEAS.md`, `TODO.md`, issue templates) 
5. Agent contract if present (`AGENTS.md`, `CLAUDE.md`, `.cursor/rules`) 
6. SQL/migrations, schema folders, seed scripts 
7. CI config for real verify commands 

List every path you opened at the end of the job.

---

## Recommended README structure

Adapt sections to what the repo actually needs. Drop sections that do not apply.
Do not invent features.

1. **What this is**, one plain paragraph. Product type, who the doc is for. No marketing adjectives.
2. **Stack and dependencies**, frameworks and critical libraries with versions from manifests you read. Say what you could not pin.
3. **Repo structure**, top-level folders, one line each. If a name misleads, say what is actually inside.
4. **Setup**, real steps: tooling versions, config files and **key names** (not secret values), databases/migrations, build/run commands from the project.
5. **Notable systems**, non-obvious flows a new collaborator would reverse-engineer (custom events, auth, data ownership). Only what the code shows.
6. **Known issues / open work**, point at trackers; do not duplicate long issue text if trackers exist.
7. **Conventions**, only patterns confirmed across multiple modules. If none, say the repo does not enforce one yet.
8. **Contributing / workflow**, only if the team has one (branch rules, review prompts, private vs public). Skip open-source boilerplate on private repos unless they use it.

---

## Do not include (unless the repo already requires them)

- Badge spam, empty license sections for private repos with no license file 
- Install steps for tools the project does not use 
- Feature lists inferred from folder names 
- Secrets, tokens, connection strings, or real keys 
- Self-referential "how this README was written" meta sections 

---

## Style

Prose rules live in `skills/writing/prose/SKILL.md`. Load it before writing final README
content. `prompts/scaffolds/Readme-Voice.md` owns tone and section layout by project type.

Two rules matter most here: no claim survives that you did not verify against an opened
file, and no sentence restates what a linked file already shows.

Match project language rules in `AGENTS.md` when present.

---

## Final response to the human

After writing `README.md`:

1. List every folder/file opened 
2. List claims left as TODO because the code did not decide them 
3. Confirm no secrets were copied into the README 
