# Clean Code Standards (all projects)

> **Your mandatory coding standards** for every project that uses this Architecture pack.  
> Derived from the personal clean-code lesson set (s4.codes-aligned), expanded into engineering rules with examples and review checklists.

## How to use

| Audience | Use |
|---|---|
| **You** | Read when forming habits; keep open during reviews |
| **Agents** | Load **one lesson file** that matches the task — never the whole folder |
| **New projects** | Copy `standards/Clean-Code/` (or symlink) into `docs/standards/Clean-Code/` and point `AGENTS.md` at it |
| **PR review** | Pick relevant numbers from the map below |

**Also load (subordinate to this folder):**

- `skills/engineering/uncle-bob/SKILL.md` — how to *apply* Clean Code / Clean Architecture / Clean Coder  
- `skills/engineering/errors/SKILL.md` — user-facing error *copy* (three laws) + extra examples; **must not contradict 03/12/13/34/37**  
- `skills/engineering/necessary-comments/SKILL.md` / `skills/engineering/craft/SKILL.md` — comment craft; subordinate to 05–10, 17, 19, 33, 38, 41  
- `standards/Principles.md` — SOLID, testing, security, concurrency; **craft sections defer to Clean-Code/**

### Authority (no contradictions)

When any Architecture doc disagrees with this folder on naming, functions, comments, null, exceptions, Demeter, DRY, CQS, or structure:

1. **`standards/Clean-Code/`** (these lessons) wins.  
2. Then Uncle Bob methodology (`skills/engineering/uncle-bob/SKILL.md`).  
3. Then other skills / `Principles.md`.  
4. Project `AGENTS.md` may name a **explicit** exception for that repo only.

Other packs were rewritten to follow this order. Prefer structure fixes (rename, extract, split, wrap) over comments or error-code returns.

---

## Agent load policy

1. Do **not** default-load all 41 files.  
2. Load the **single** `NN-*.md` that matches naming, errors, comments, structure, etc.  
3. For a full audit, load this README + scan the map, then open only files for findings.  
4. Prefer structure fixes (rename, extract, split, wrap) over adding comments or return codes.  
5. On conflict with `Principles.md` or `skills/*`, follow **this folder**.

---

## Full map (01–41)

| # | File | One-line rule |
|---|---|---|
| 01 | [01-side-effects-honest-functions.md](./01-side-effects-honest-functions.md) | No side effects beyond the name |
| 02 | [02-single-argument-patterns.md](./02-single-argument-patterns.md) | Argument *roles* — no flag-jobs (not “only one arg ever”) |
| 03 | [03-exceptions-over-error-codes.md](./03-exceptions-over-error-codes.md) | Exceptions over error codes (happy path flat) |
| 04 | [04-clarity-is-king.md](./04-clarity-is-king.md) | No mental mapping |
| 05 | [05-code-evolves-comments-may-not.md](./05-code-evolves-comments-may-not.md) | Stale comments are worse than none |
| 06 | [06-intention-revealing-names.md](./06-intention-revealing-names.md) | Names reveal intent |
| 07 | [07-leave-history-to-git.md](./07-leave-history-to-git.md) | No commented-out code / journals |
| 08 | [08-comments-that-protect.md](./08-comments-that-protect.md) | Warnings, amplification, real TODOs |
| 09 | [09-comments-clarify-not-confuse.md](./09-comments-clarify-not-confuse.md) | Specific, local, complete |
| 10 | [10-comments-must-communicate.md](./10-comments-must-communicate.md) | Instantly readable comments |
| 11 | [11-variables-close-to-use.md](./11-variables-close-to-use.md) | Locals near use; fields grouped |
| 12 | [12-define-contract-before-logic.md](./12-define-contract-before-logic.md) | Failure contract first |
| 13 | [13-stop-returning-null.md](./13-stop-returning-null.md) | Never return null — empty / Special Case / throw |
| 14 | [14-dry-single-source-of-truth.md](./14-dry-single-source-of-truth.md) | One authoritative representation |
| 15 | [15-one-thing-tell-a-story.md](./15-one-thing-tell-a-story.md) | One thing; top-down story |
| 16 | [16-not-everything-is-an-object.md](./16-not-everything-is-an-object.md) | Classes only when needed |
| 17 | [17-comments-tell-why.md](./17-comments-tell-why.md) | Why, not how |
| 18 | [18-stepdown-rule.md](./18-stepdown-rule.md) | Highest level first |
| 19 | [19-noise-comments.md](./19-noise-comments.md) | If code says it, comment is noise |
| 20 | [20-names-that-fail.md](./20-names-that-fail.md) | Call site must be enough |
| 21 | [21-expose-behavior-not-data.md](./21-expose-behavior-not-data.md) | Tell objects; don’t strip-mine fields |
| 22 | [22-messy-first-draft-then-clean.md](./22-messy-first-draft-then-clean.md) | Draft messy; ship clean |
| 23 | [23-low-argument-count.md](./23-low-argument-count.md) | Keep arity low |
| 24 | [24-bury-the-switch.md](./24-bury-the-switch.md) | Bury switches; keep policy clean |
| 25 | [25-let-tools-handle-types.md](./25-let-tools-handle-types.md) | Types/linters/formatters own ceremony |
| 26 | [26-extract-from-large-functions.md](./26-extract-from-large-functions.md) | Extract until the parent is an outline |
| 27 | [27-one-abstraction-level.md](./27-one-abstraction-level.md) | One abstraction level per function |
| 28 | [28-pairs-and-triads.md](./28-pairs-and-triads.md) | Careful pairs/triads; named bundles |
| 29 | [29-objects-vs-data-structures.md](./29-objects-vs-data-structures.md) | Object **or** data structure |
| 30 | [30-searchable-names.md](./30-searchable-names.md) | Searchable names; no magic numbers |
| 31 | [31-wrap-third-party-apis.md](./31-wrap-third-party-apis.md) | Wrap vendors; own errors |
| 32 | [32-law-of-demeter.md](./32-law-of-demeter.md) | No stranger chains |
| 33 | [33-code-explains-itself.md](./33-code-explains-itself.md) | Rename/extract before “what” comments |
| 34 | [34-catch-is-not-if.md](./34-catch-is-not-if.md) | Exceptions ≠ normal branches |
| 35 | [35-formatting.md](./35-formatting.md) | Vertical density + hierarchy |
| 36 | [36-honest-naming-no-disinformation.md](./36-honest-naming-no-disinformation.md) | No misleading names |
| 37 | [37-algorithm-breathes.md](./37-algorithm-breathes.md) | Happy path reads as pure steps |
| 38 | [38-comments-apologizing-for-structure.md](./38-comments-apologizing-for-structure.md) | Fix structure, not brace comments |
| 39 | [39-pronounceable-names.md](./39-pronounceable-names.md) | Speakable names |
| 40 | [40-command-query-separation.md](./40-command-query-separation.md) | Command **or** query |
| 41 | [41-docs-for-the-world.md](./41-docs-for-the-world.md) | Code for team; docs for the world |

---

## Topic clusters (load by concern)

| Concern | Lessons |
|---|---|
| **Naming** | 04, 06, 20, 30, 36, 39 |
| **Functions & structure** | 01, 02, 15, 18, 22, 23, 24, 26, 27, 28, 40 |
| **Errors** | 03, 12, 13, 34, 37 |
| **Comments** | 05, 07, 08, 09, 10, 17, 19, 33, 38, 41 |
| **Objects & design** | 16, 21, 29, 31, 32 |
| **DRY & tools** | 14, 25, 35 |
| **Variables / formatting** | 11, 35 |

---

## Suggested `AGENTS.md` snippet (drop into projects)

```markdown
## Coding standards
- Mandatory: docs/standards/Clean-Code/ (this Architecture pack).
- Agents: load only the Clean-Code lesson file(s) relevant to the task.
- Prefer rename/extract/split/wrap over comments or error-code returns.
- Full map: docs/standards/Clean-Code/README.md
```

---

## Editing a lesson

The 41 lesson files are hand-maintained. Edit the `NN-*.md` file directly, then update its
one-line rule in the map above so the two never drift apart.

Keep the shared trailing **Authority** block identical across files. If that block changes,
change it in all 41.
