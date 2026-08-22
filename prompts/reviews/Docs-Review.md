# Documentation Review

An audit of the written material in a repo, run the same way a code review is run: explicit
scope, `report-only` until someone says otherwise, and every claim checked against a file
you opened.

Docs rot silently. A wrong command in a README does not fail CI, so nobody finds it until a
new person loses an afternoon to it. This pass finds those before they do.

For writing a README from scratch, use `skills/writing/create-readme/SKILL.md` instead. This
file reviews what already exists.

## Before you open a file

1. Read `skills/review/audit/SKILL.md`. Scope is required. The mode is `report-only` until
   the human sets `fix`.
2. Read `skills/writing/prose/SKILL.md`. It holds the prose rules this review applies.
3. Read the project `AGENTS.md` if there is one. It sets tone and names the real verify
   commands.
4. Do not run AI-detector scoring, and do not rewrite a doc to lower a detector score.

**Scope for this prompt is a path list, usually globs.** `README.md`, `docs/**/*.md`,
`*/README.md`. A whole repo of markdown with mode `fix` is the same mistake as an unbounded
code refactor.

## The four phases

| Phase | What you are looking for |
|---|---|
| **1. Truth** | Claims that are wrong or no longer true |
| **2. Structure** | Whether a reader can find what they came for |
| **3. Prose** | Whether it reads like someone wrote it for a person |
| **4. Remediation** | The exact edit, applied only when the mode is `fix` |

Work them in that order. A beautifully written install section with the wrong package
manager is worse than an ugly one that works.

---

## Phase 1: Truth

Every checkable claim gets checked. Open the file, run nothing that mutates state, and note
what you verified.

- [ ] Commands exist and match the project's real toolchain (`package.json` scripts, the
      `Makefile`, CI config)
- [ ] File paths, directory names, and links resolve. Include relative links between docs
- [ ] Version numbers and pins match the manifests and lockfiles
- [ ] Environment variable names match what the code actually reads
- [ ] Ports, URLs, and default hosts match the config
- [ ] Described behavior matches current code, not the version the doc was written against
- [ ] Screenshots and output blocks show the current UI or the current CLI output
- [ ] No secrets, tokens, connection strings, or internal hostnames. Report exposure without
      reprinting the value
- [ ] Features listed actually ship. Anything aspirational is labeled as planned

An unverifiable claim is a finding. Mark it `unverified` rather than assuming it is fine.

---

## Phase 2: Structure

- [ ] The first paragraph says what this is and who it is for, with no marketing adjectives
- [ ] Setup is reachable in one screen from the top
- [ ] One concept lives in one place. Two files stating the same rule will drift into two
      different rules
- [ ] Headings describe content, so a reader scanning the sidebar can route themselves
- [ ] Sections nobody needs are gone rather than kept for symmetry
- [ ] Cross-references point at files instead of restating them
- [ ] An index or README that lists siblings actually lists all of them. An unlisted file is
      a file nobody loads

---

## Phase 3: Prose

Apply `skills/writing/prose/SKILL.md`. Report by pattern, not line by line, and quote one
example per pattern rather than every instance.

- [ ] Filler adjectives, verb inflation, discourse glue
- [ ] Adverbs standing in for numbers
- [ ] Em dash used as sentence punctuation
- [ ] Binary contrast, negative listing, rhetorical setup, false agency, narrator distance
- [ ] Fragments written for rhythm, punchy section closers
- [ ] List lengths padded to three
- [ ] Passive voice that hides who acts

Style findings are `low` unless they make an instruction ambiguous. A tidy sentence is worth
less than a correct one, and a prose pass must never change a factual claim.

---

## Phase 4: Remediation

Write every finding as: location, what the doc claims, what is actually true, severity, the
edit, and how you verified it.

Severity for documentation:

| Severity | Means |
|---|---|
| **Blocker** | Following the doc breaks something or leaks a secret. Wrong destructive command, exposed credential, install step that corrupts state |
| **High** | Following the doc fails. Dead command, broken required link, wrong env var name |
| **Medium** | The doc misleads without failing. Stale behavior, duplicated rule that has drifted, missing setup step a reader can work around |
| **Low** | Style, prose tics, formatting, an unlisted file in an index |

Be specific about effect. "The README is outdated" is not a finding. "`README.md:42` says
`npm install`, but the repo has `bun.lockb` and no `package-lock.json`, so a new contributor
installs a second lockfile" is a finding, because someone can act on it.

When the mode is `fix`, edit only in-scope files, keep the project's existing voice, and
leave a marked `TODO` for anything you could not verify rather than guessing at it.

Close with the report shape from `skills/review/audit/SKILL.md`: what you covered, what you
excluded and why, findings by severity, verification, and a go or no-go.
