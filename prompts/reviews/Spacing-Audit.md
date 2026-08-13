> **Harness (required):** Obey `skills/review/audit/SKILL.md` before this prompt.
> Scope must be explicit (components, routes, or frames). Default mode is `report-only`.
> Do not change anything unless mode is `fix`.
> Read project `AGENTS.md` for owned code. If the project already defines a spacing scale,
> that scale wins over the one below.
> End report: coverage list, before/after table, values left untouched with reasons.

---

# Spacing Audit

Check a layout against a 4/8pt spacing scale, report every value that is off the grid, and
snap it to the nearest correct step. The goal is consistent, predictable rhythm that is easy
to hand off and maintain.

Inconsistent spacing is the most common reason a competent layout looks amateur. Nobody can
point at what is wrong, because no single value is wrong. The rhythm is.

## The scale

Base unit is 8. Use 4 only as a half-step for tight, component-internal spacing.

| Band | Values | Where it belongs |
|---|---|---|
| **Structural** | 8, 16, 24, 32, 48, 64, 80, 96 | Section gaps, card padding, layout margins |
| **Fine** | 4, 12, 20 | Inside a component: icon-to-label, chip padding, tight stacks |
| **Zero** | 0 | Deliberate flush edges |

- **On-grid** is any multiple of 4.
- **Off-grid** is anything that is not a multiple of 4. Hard flag.
- A **4-only step used structurally** (a section gap of 20, 28, or 36) is on-grid but out of
  band. Soft flag it as "consider an 8pt step" and let the human decide. The same value
  inside a component is fine and needs no flag.

## Which surface you are auditing

State it before you start, because the vocabulary differs.

| | In code | In a design file |
|---|---|---|
| Padding | `padding`, `p-*`, `px-*`, `py-*` | Frame padding, per side |
| Gap | `gap`, `row-gap`, `column-gap`, `space-y-*` | Item spacing in an auto-layout frame |
| Margin | `margin`, `m-*`, `mt-*` | Spacing between grouped blocks |
| Off-limits | Anything driven by a token or a CSS custom property | Anything bound to a variable, or set by hug/fill |

**In a Tailwind codebase the audit mostly reduces to one thing.** Tailwind's default spacing
scale is already this system: `1` is 4px, `2` is 8px, `3` is 12px, `4` is 16px, `6` is 24px,
`8` is 32px, `12` is 48px, `16` is 64px. Every value in the table above has a named step. So
named classes are on-grid by construction, and the off-grid values are almost always
arbitrary ones: `p-[14px]`, `gap-[18px]`, `mt-[30px]`. Grep for `-\[` and start there.

## What to check

- Padding inside frames and containers, all four sides.
- Gap between children, and between grouped blocks.
- Gaps between sections.
- Margins around content within a container.

## How to audit

1. Walk the scoped frame or component tree top-down. If nothing is scoped, ask before
   walking the whole page.
2. Compare each spacing value against the scale.
3. For each off-grid value, pick the nearest allowed step. When a value sits exactly between
   two steps, round toward the surrounding rhythm of its group rather than up by default.
4. **Prefer consistency within a group over per-element correctness.** If three cards use
   14, 16, and 16, take all three to 16. A group that is uniform at a slightly wrong value
   reads better than a group that is individually correct and visually ragged.

## Rules

- Change spacing values only. Do not touch layout structure, sizing behavior (hug/fill,
  flex-grow), content, or styling.
- **Leave tokens alone.** If a value is already bound to a variable, token, or custom
  property, note it and move on. Overwriting a token's output hides the real fix, which is
  the token's value.
- Preserve intentional exceptions without flagging them: 1px borders and hairlines, optical
  alignment offsets, and any spacing that is a consequence of hug/fill rather than a set
  value.
- Round icon-to-label and other fine gaps to 4 or 8. Never promote a fine gap to a
  structural step because it was closer on the number line.
- If a whole group is uniformly off-grid, that is usually a deliberate system (a 6pt scale,
  or a design that predates yours). Flag it once, ask, and do not mass-change it.

## Output

Report first. Change nothing until the summary is on screen and, in `report-only` mode,
change nothing at all.

1. **Coverage.** What you walked, and total elements checked.
2. **Findings**, grouped by frame or component, as `Card padding: 14 -> 16`.
3. **Left untouched**, each with its reason: token, hairline, optical, hug/fill.
4. **Soft flags**, the 4-only structural steps, listed separately from hard fixes.
5. **Rhythm note**, one line: the dominant step the layout now uses, for example "layout on
   an 8pt rhythm, fine spacing on 4pt."

Apply fixes only after the summary, and only when the mode is `fix`.

## Do not

- Do not resize elements or reflow content to hit a number.
- Do not invent a new scale. Use the steps above, or the project's own if it has one.
- Do not touch corner radius, stroke, type, or color unless asked.
- Do not "fix" a group by making it uniformly wrong in a new way.
