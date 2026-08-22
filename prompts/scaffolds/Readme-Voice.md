# README Prompt

Voice and structure templates for README.md. For a **discovery-first** README written only from the real tree (no guessing), also load `skills/writing/create-readme/SKILL.md`.

This file owns tone and section layouts by project type. The skill owns evidence rules.


## Before you write

1. Prefer `skills/writing/create-readme/SKILL.md` for discovery (open the tree; no guessing).
2. Read `skills/writing/prose/SKILL.md` for the prose rules. **This file overrides it on
   casing and formality only.** The lowercase, contraction-friendly voice below is a
   deliberate choice for personal repos. Everything else in that skill still applies:
   no filler, no em dash punctuation, no binary contrast or rhetorical setup, no list
   padded to three.
3. Pick a template below only after you know project type and whether the repo is public or private.
4. **Private repos / small teams:** skip badge spam, generic Contributing, and license sections unless the repo already has them. Use Small / Personal or a trimmed Full template.
5. Never paste secrets, tokens, or live connection strings.
6. Match tone to `AGENTS.md` when present.

---

You are writing a README.md for a project. Match the tone and structure below exactly.

---

## Voice

- Casual, lowercase descriptions. not corporate.
- Don't over-explain. if it's obvious, skip it.
- Short sentences. no filler paragraphs.
- Write like a dev talking to another dev, not a marketing page.
- First person is fine ("i've been working on this", "hit me up").
- Contractions and informal grammar are fine.

---

## Structure by Project Type

Pick the right template based on the project. Don't mix them.

---

### Small / Personal Projects

For simple bots, scripts, personal tools. No badges, no center alignment. Just straight to the point.

```markdown
# project-name

one-line description of what it does and who it's for (if relevant)


## commands

- `/command` - what it does
- `/command @user [optional]` - what it does


## permissions

what the bot/app needs to work:
- permission 1
- permission 2

---

contact line or "hit me up" note
```

**Rules:**
- No badges, no shields, no centered title
- Just h1 â†’ description â†’ sections â†’ done
- Include permissions/requirements only if the user needs to know
- End with a casual contact note if it's for someone specific

---

### Full Projects

For apps, desktop tools, published repos. Badges, centered title, screenshots, roadmap.

```markdown
<div align="center" id="top">

# Project Name

</div>
<p align="center">
  <img alt="Top language" src="https://img.shields.io/github/languages/top/USER/REPO?color=56BEB8">
  <img alt="Language count" src="https://img.shields.io/github/languages/count/USER/REPO?color=56BEB8">
  <img alt="Repository size" src="https://img.shields.io/github/repo-size/USER/REPO?color=56BEB8">
  <img alt="License" src="https://img.shields.io/github/license/USER/REPO?color=56BEB8">
</p>

---

short description of what it is. one or two sentences max.

![Screenshot](assets/img/screenshot.png)

## What is this

a paragraph explaining the project. what problem it solves, why it exists.
keep it casual. mention how long you've been working on it if relevant.
mention if it's beta/WIP.

## Features

- feature one
- feature two
- feature three

## Stack

- Language/Runtime
- Framework
- Key library
- Key library

## Install

\```bash
git clone https://github.com/USER/REPO.git
cd REPO
<install command>
<run command>
\```

## Build

\```bash
<build command>
\```

Output goes to `dist/` (or wherever)

## Screenshots

![Name](path/to/screenshot.png)

## Known Issues

- issue one (and when it happens)
- issue two

## Roadmap

- [ ] planned feature
- [ ] planned feature
- [ ] planned feature

## Contact

- Platform: [@handle](url)
- Platform: [@handle](url)

## License

<license>
```

**Rules:**
- Always center the title with `<div align="center">`
- Always include the 4 shield badges (top language, language count, repo size, license) with `?color=56BEB8`
- "What is this" instead of "About" â€” more natural
- Stack section is a flat bullet list, link each item
- Screenshots go in their own section
- Roadmap uses `- [ ]` checkboxes
- Known Issues only if there are actual known issues
- License at the bottom, just the name or a link

---

### Templates / Starter Projects

For boilerplates, templates, libraries. Focus on usage, API, and getting started fast.

```markdown
# project-name

one-line description of what it is.

## what's included

- feature / thing included
- feature / thing included
- feature / thing included

## install

needs [runtime](url) version+

\```bash
git clone https://github.com/USER/REPO.git
cd REPO
<install command>
<run command>
\```

runs on :PORT

## example

\```language
// example code showing the main usage pattern
\```

short explanation of what the example does.

## commands

\```bash
<dev command>       # dev mode
<build command>     # build
<start command>     # prod
<test command>      # tests
\```

## usage patterns

### basic usage

\```language
// code example
\```

### with validation / options

\```language
// code example
\```

### advanced pattern

\```language
// code example
\```

## other stuff

### environment

\```bash
KEY=value
KEY=value
\```

### before committing

\```bash
<lint + test command>
\```

---

<license> license
```

**Rules:**
- No badges, no centered title â€” templates should feel lightweight
- "what's included" instead of "features"
- Show a real code example early â€” don't make people scroll
- Commands section with inline comments explaining each
- "other stuff" for env vars, contributing notes, misc
- Keep it scannable â€” someone should understand the template in 30 seconds

---

### Mid-Size Projects (Bots, Tools, APIs)

For projects that are more than personal but not full apps. Badges + centered title, but lighter than full projects.

```markdown
<div align="center" id="top">

# Project Name

</div>
<p align="center">
  <img alt="Top language" src="https://img.shields.io/github/languages/top/USER/REPO?color=56BEB8">
  <img alt="Language count" src="https://img.shields.io/github/languages/count/USER/REPO?color=56BEB8">
  <img alt="Repository size" src="https://img.shields.io/github/repo-size/USER/REPO?color=56BEB8">
  <img alt="License" src="https://img.shields.io/github/license/USER/REPO?color=56BEB8">
</p>

---

## About

**Project Name** is a [language] [type] that does [thing]. short description.

## Tech Stack

- [Language](url)
- [Library](url)
- [Library](url)

---

## Setup

### Configuration

what to edit and where:

\```json
{
  "key": "value"
}
\```

### Requirements

- what's needed before running (accounts, tokens, permissions)
- step-by-step if there's a setup flow (developer portal, etc.)

### Running

\```bash
git clone https://github.com/USER/REPO
cd REPO
<install deps>
<run command>
\```

---

## Example

![](screenshot-url)

```

**Rules:**
- Centered title + badges like full projects
- "Tech Stack" with linked items
- Setup split into Configuration â†’ Requirements â†’ Running
- Example section with a screenshot if available
- No roadmap unless there's actual planned work
- Lighter than full projects â€” don't add sections just to fill space

---

## General Rules

1. **Never write "Table of Contents"** â€” if the README needs one, it's too long
2. **No "Getting Started" as a header** â€” just call it "Install" or "Setup"
3. **No "Prerequisites"** â€” fold it into Install ("needs [bun](url) 1.0+")
4. **No "Contributing" section** unless it's an open source project expecting PRs
5. **No "Acknowledgments"** â€” if you want to credit someone, do it in Contact or a one-liner
6. **Stack/Tech Stack is always a bullet list** â€” never a table, never a paragraph
7. **Badge color is always `56BEB8`**
8. **Screenshots are just `![Name](path)`** â€” no HTML image tags unless sizing is needed
9. **License goes last** â€” just the name or `<license>` as placeholder
10. **Contact uses `[@handle](url)` format** â€” platform name before the link
