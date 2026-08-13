> **Harness:** Read project `AGENTS.md` if present. Prefer extending an existing repo.
> Implement only the pages/features requested. Discover verify commands from the project.
> Reviews: `skills/review/audit/SKILL.md`. Craft: `skills/engineering/craft/SKILL.md`.

# Website Prompt

Build a website with TypeScript and Tailwind CSS. Use the major version the project already has. Do not downgrade it, and do not introduce a second CSS system alongside it.

---

## Instructions

Section-load:

- `languages/TypeScript.md` (structure, errors, validation)
- `frameworks/React-Tailwind.md` (components, Tailwind patterns)
- Relevant `standards/Principles.md` sections only

**Architecture**

- Component-based structure from the framework file
- TypeScript throughout
- Tailwind for styling unless the project already uses something else
- Router only if multi-page is in scope
- Context only for truly global state (auth, theme)

**If greenfield structure is needed:**
```
website/
  src/
    components/
    pages/
    hooks/
    context/
    types/
    main.tsx
  public/
  index.html
```

**Styling**

- Prefer utilities already used in the repo
- Mobile-first responsive design
- Consistent spacing scale (e.g. 4/8/12/16/24/32/48/64)

**Credits / footer**

- Do **not** hard-code personal handles unless the human or `AGENTS.md` / existing site already define them.
- If a footer is requested without copy, use a neutral placeholder and ask.

**Responsive**

- Mobile-first
- Touch targets ~44px minimum where primary actions are tappable

---

## Setup (greenfield only)

Use the project's package manager and the framework file's current setup. Example shape:

```bash
npm create vite@latest project-name -- --template react-ts
cd project-name
npm install
# install Tailwind per current official Vite guide for this major version
npm install react-router-dom
```

Do not paste obsolete Tailwind init commands if they conflict with the major version in use. Prefer official docs + lockfile.

---

## Deliverables (only if full site requested)

1. Structure matching language/framework files
2. Responsive layout
3. TypeScript with no `any` without justification
4. Pages the human named (if none, ask)
5. Forms with loading and error states when forms exist
6. README with setup
7. No secrets in the repo

---

## Validation checklist

- [ ] Scoped to requested pages/features
- [ ] No `any` without a documented reason
- [ ] Styling matches project system
- [ ] Mobile-usable primary flows
- [ ] Loading and error states on async UI
- [ ] Typecheck/build per project scripts
- [ ] Names/structure per Craft + framework file
