# Website Design Prompt

> **Agent load:** Design/UI tasks only. Do not dump this entire file for backend work.
> Open Philosophy, Visual Hierarchy, and the sections matching the task (type, color, motion, a11y).
> Build only what was asked. Prefer the project's existing design system when one exists.
> Craft: `skills/engineering/craft/SKILL.md`. Reviews of implemented UI: `skills/review/audit/SKILL.md` with explicit scope.
Design and build the UI. Every spacing value, type choice, and interaction should be one you can defend. If you cannot say why a number is what it is, it is a guess, and it will look like one.

---

## Philosophy

Design is not decoration. Every visual decision communicates something. A good website solves a problem clearly, creates trust instantly, and gets out of the user's way. Study Dieter Rams: your interface should be as honest and unobtrusive as good furniture.

**Before writing a single line of code, ask:**

- What is the one thing this page must accomplish?
- What does the user already know when they arrive?
- What is the first thing their eye should land on?
- Where does the page fall apart on mobile?

---

## Visual Hierarchy (Do This First)

**The eye always follows contrast, size, and whitespace, in that order.**

### DO

- Establish one dominant element per section (heading, image, or CTA, not all three at once)
- Use size to signal importance: H1 should feel 3-4× larger than body text
- Create breathing room with generous whitespace, it signals confidence, not emptiness
- Use a strict type scale: 12 / 14 / 16 / 20 / 24 / 32 / 48 / 64 / 96px
- Treat whitespace as a design element, not leftover space

### DON'T

- Compete with yourself: two bold CTAs cancel each other out
- Equalize everything, equal sizing creates visual monotony
- Fill every gap, cramped layouts signal low quality
- Use more than two font weights per section
- Bold random words in paragraphs (bold loses meaning when overused)

---

## Typography

**Good typography is invisible. Bad typography is all you see.**

### DO

- Choose fonts with personality that match the brand tone, no generic defaults
- Pair one display face with one legible body face, maximum
- Set body text between 16-18px for comfortable reading
- Line height: 1.5-1.65 for body, 1.1-1.2 for headlines
- Limit line length to 60-75 characters (use `max-width: 65ch`)
- Use real typographic punctuation in the product UI where it fits the brand (curly quotes, ellipsis). Engineering docs in this pack still avoid em dash as sentence punctuation.

### DON'T

- Default to Inter, Roboto, or Arial, they are the Times New Roman of the web
- Use more than two typefaces in one design
- Set body text below 14px or above 20px
- Justify text in web layouts (creates uneven rivers of whitespace)
- Use all-caps for anything longer than 3 words
- Letter-space body text (only appropriate for all-caps labels)

```
✓ GOOD type scale:
  Label, 12px, 0.08em letter-spacing, uppercase
  Body, 16px, 1.6 line-height
  Lead, 20px, 1.5 line-height
  H3, 24px, 1.2 line-height
  H2, 36px, 1.15 line-height
  H1, 56px, 1.05 line-height

✗ BAD type decisions:
  <p style="font-size: 13px; line-height: 1.3">, too tight, too small
  <h1 style="font-size: 28px">, H1 should dominate
  font-family: Arial, Helvetica, sans-serif, generic and boring
```

---

## Color

**Color communicates before the user reads a word.**

### DO

- Start with a neutral base (near-white or near-black, not pure #000/#fff)
- Choose one primary accent color, one secondary, and stick to them
- Use `oklch()` or HSL for predictable color manipulation
- Reserve bright color for actions and focus states only
- Test contrast ratios: 4.5:1 for body text, 3:1 for large text (WCAG AA)
- Use color to communicate meaning, not decoration

### DON'T

- Use more than 4-5 colors in a palette (plus neutrals)
- Use pure black `#000000` for text, use `#111` or `#1a1a1a`
- Use purple-to-blue gradients as your primary background, it's 2019
- Make interactive elements indistinguishable from decorative ones
- Rely on color alone to convey information (consider color-blind users)

```
✓ GOOD palette structure:
  Background:   #f8f7f4  (warm off-white)
  Surface:      #ffffff
  Text primary: #1a1917
  Text muted:   #6b6860
  Accent:       #d4521a  (one strong color)
  Accent light: #fce8dc  (tint for backgrounds)
  Border:       #e5e3de

✗ BAD palette decisions:
  background: linear-gradient(135deg, #667eea, #764ba2), overused
  color: #333333, boring; use an intentional near-black
  Using 8+ colors with no system
```

---

## Spacing

**Spacing is the single most underrated design tool.**

### DO

- Use a base-8 spacing scale: 4 / 8 / 12 / 16 / 24 / 32 / 48 / 64 / 96 / 128px
- Give sections generous vertical padding (80-160px on desktop)
- Keep related elements close together, unrelated elements far apart
- Use asymmetric padding to create visual tension and interest
- Increase whitespace as content importance increases

### DON'T

- Use arbitrary values: `padding: 23px`, pick from your scale
- Apply equal spacing everywhere, it erases hierarchy
- Pack elements into corners out of fear of empty space
- Use the same vertical gap between every section
- Treat padding and margin as the same concept

---

## Layout & Grid

**The grid is a constraint that creates freedom.**

### DO

- Establish a max content width (1200-1400px) and center it
- Use CSS Grid for two-dimensional layouts, Flexbox for one-dimensional
- Break the grid deliberately for impact, hero images, pull quotes
- Design mobile-first: start at 375px, then expand
- Use `clamp()` for fluid typography and spacing
- Align to an invisible baseline grid for vertical rhythm

### DON'T

- Make columns too narrow, below 280px, text becomes unreadable
- Center-align body paragraphs (only acceptable for 1-2 line captions)
- Use 12-column grids with all 12 columns filled, use only 2-4 columns meaningfully
- Ignore the fold, the first 600px of vertical scroll determines if users stay
- Nest grids more than two levels deep

```
✓ GOOD layout decisions:
  .container { max-width: 1280px; margin: 0 auto; padding: 0 24px; }
  section { padding: clamp(64px, 10vw, 128px) 0; }
  h1 { font-size: clamp(2.5rem, 5vw, 5rem); }

✗ BAD layout decisions:
  width: 1000px; margin: 0 auto;, fixed width breaks at 1000px exactly
  padding: 15px;, off-scale, arbitrary
  Centered paragraphs of 5+ lines, unreadable
```

---

## Navigation

**Navigation is a promise. Every link is a commitment.**

### DO

- Keep primary navigation to 5-7 items maximum
- Make the current page/state visually distinct
- Ensure tap targets are at least 44×44px on mobile
- Use a sticky nav only if it adds real value (long pages, frequent jumping)
- Keep logo on the left, primary CTA on the right
- Make the mobile menu feel native, not like an afterthought

### DON'T

- Hide navigation behind a hamburger menu on desktop
- Use dropdown menus with 15+ items, restructure the information instead
- Make the logo a decorative image with no link to home
- Use hover-only interactions on mobile (no hover state exists)
- Animate the nav logo on every scroll event (nauseating)

---

## Calls to Action

**One primary action per view. Everything else is secondary.**

### DO

- Make primary buttons visually dominant, full color, solid fill
- Use secondary buttons only when genuinely offering a choice
- Write CTA copy that describes the outcome: "Start Free Trial" not "Submit"
- Give buttons enough padding: `px-6 py-3` minimum, `px-8 py-4` for hero CTAs
- Maintain 3:1 minimum contrast between button and page background

### DON'T

- Place two equal-weight CTA buttons next to each other
- Use "Click Here", "Learn More", or "Submit" as CTA text
- Make buttons too small to tap (under 44px height on mobile)
- Use ghost/outline buttons as your primary action
- Disable buttons without explaining why they're disabled

```
✓ GOOD CTA copy:
  "Start Building Free"
  "See Pricing"
  "Download the Guide"
  "Book a Demo"

✗ BAD CTA copy:
  "Submit"
  "Click Here"
  "Learn More"
  "Get Started" (with no context for what you're starting)
```

---

## Imagery & Icons

**Every image either earns its place or costs you trust.**

### DO

- Use images that show real outcomes, real people, or real product
- Compress all images: WebP for photos, SVG for illustrations and icons
- Set explicit `width` and `height` on images to prevent layout shift
- Use `loading="lazy"` on below-the-fold images
- Ensure icon sizes are consistent within a set (16px or 24px, not mixed)
- Use `alt` text that describes the image's purpose, not its appearance

### DON'T

- Use stock photos of people in offices pointing at whiteboards
- Mix icon styles (filled, outlined, duotone) in the same UI
- Use bitmap icons when SVGs are available
- Use decorative images without `alt=""`
- Scale images up beyond their natural resolution
- Place text directly on busy photographs without a proper contrast layer

---

## Interactions & Animation

**Animation should clarify, not entertain.**

### DO

- Use transitions for state changes: `transition: all 0.15s ease`
- Animate opacity and transform, never width/height (triggers layout)
- Use `prefers-reduced-motion` media query to disable animations when asked
- Make hover states immediate (0-100ms) and exits slightly slower (100-200ms)
- Use micro-interactions to confirm actions: button press, form submit, copy

### DON'T

- Animate on page load without user intent (unless it's < 200ms)
- Use `animation: spin infinite` on anything the user must read
- Animate more than 2 properties simultaneously without a clear reason
- Use JavaScript for animations that CSS can handle
- Make animations that loop without a clear purpose

```css
✓ GOOD transitions:
  button { transition: background-color 0.15s ease, transform 0.1s ease; }
  button:hover { transform: translateY(-1px); }
  button:active { transform: translateY(0); }

✗ BAD transitions:
  * { transition: all 0.5s ease; }, too broad, too slow
  .card:hover { width: 105%; }, triggers layout recalculation
```

---

## Forms

**Every field you add costs you a submission.**

### DO

- Label every input above the field (not inside as placeholder)
- Show validation errors inline, immediately after the user leaves the field
- Use `type="email"`, `type="tel"`, `type="number"` for appropriate mobile keyboards
- Group related fields logically
- Show progress indicators on multi-step forms
- Make the submit button reflect the action: "Send Message" not "Submit"

### DON'T

- Use placeholder text as the only label (it disappears on focus)
- Show all validation errors only after submission
- Use dropdowns for fewer than 5 options (use radio buttons instead)
- Ask for information you don't actually need
- Use CAPTCHA as the first line of spam defense
- Create required fields that aren't obviously required

---

## Performance & Loading States

**A slow website is a broken website.**

### DO

- Target < 3 seconds Time to Interactive on mobile 4G
- Show skeleton screens or shimmer loaders while content loads
- Preload critical fonts and hero images
- Use `font-display: swap` to prevent invisible text during font load
- Implement loading states for every async action
- Paginate or virtualize lists over 50 items

### DON'T

- Show a blank white screen while JavaScript hydrates
- Use spinner icons for operations that take > 2 seconds (use progress bars)
- Load all page data before showing any content
- Ship unoptimized images (WebP + appropriate dimensions)
- Block rendering with synchronous scripts in `<head>`

---

## Accessibility

**Accessible design is good design for everyone.**

### DO

- Ensure all interactive elements are keyboard accessible
- Use semantic HTML: `<nav>`, `<main>`, `<article>`, `<button>` over `<div>` everywhere
- Maintain visible focus indicators (don't just `outline: none`)
- Provide `aria-label` on icon-only buttons
- Use `<h1>` → `<h2>` → `<h3>` in proper order (never skip levels)
- Test with a screen reader at least once

### DON'T

- Use `div` and `span` for clickable elements
- Remove focus outlines without providing an alternative
- Create color contrast below WCAG AA (4.5:1 for text)
- Auto-play video or audio
- Use `title` attribute as the primary tooltip mechanism
- Create hover-only interactions (menus that only appear on hover)

---

## Mobile Design

**Mobile is not a reduced desktop. It's a different context entirely.**

### DO

- Design mobile-first, then scale up
- Stack elements vertically on mobile, avoid horizontal scroll
- Use touch-friendly targets (44px minimum height/width)
- Simplify or collapse complex navigation
- Test real device sizes: 375px, 390px, 430px (common iPhones)
- Use `env(safe-area-inset-*)` for devices with notches

### DON'T

- Design desktop-first and "make it responsive" at the end
- Show identical navigation on mobile and desktop
- Use hover-dependent interactions (no hover on touch)
- Make modals that don't scroll on mobile
- Use small font sizes to fit more content (min 16px for body on mobile)
- Assume landscape orientation is primary

---

## AI Detection: Design Edition

**Avoid these patterns that signal template-generated, AI-default design:**

```
✗ AI DEFAULT PATTERNS:
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%)
  font-family: 'Inter', sans-serif, or Roboto, Poppins as the only choice
  Three cards with icons on white background as the "features" section
  Hero: centered h1 + paragraph + two buttons + hero image
  Testimonial section: photo circle + star rating + quote
  Pricing: three tiers, middle one "highlighted" with a badge
  Footer: four columns of links + social icons + copyright

✓ HUMAN DESIGN CHOICES:
  A type-led design where the headline does the heavy lifting
  Off-white or tinted backgrounds instead of pure white
  Intentional asymmetry: text left, image bleeding off-right
  A single feature shown in depth instead of six shown shallowly
  Honest product screenshots instead of mockup devices
  Unexpected section breaks: full-bleed color, diagonal dividers
  Typography as the primary visual element, not just decoration
```

---

## Validation Checklist

Before delivering:

- [ ] One dominant visual element per section
- [ ] Type scale is consistent and logical
- [ ] Color palette has a clear system (primary, accent, neutral)
- [ ] Whitespace feels generous, not cramped
- [ ] All CTAs describe an outcome, not an action
- [ ] Mobile layout tested at 375px width
- [ ] Tap targets are at least 44×44px
- [ ] All images have alt text
- [ ] Keyboard navigation works throughout
- [ ] No pure black (#000) or pure white (#fff) for primary surfaces
- [ ] Visual defaults are intentional for the brand, not generic template chrome
- [ ] No stock photo of people in offices
- [ ] Contrast ratios pass WCAG AA
- [ ] Loading states exist for every async operation
- [ ] Fonts are not Inter, Roboto, or Arial
- [ ] No placeholder-only form labels
- [ ] Animations use transform/opacity only
- [ ] `prefers-reduced-motion` is respected

---

## Quick Reference: The 10 Principles (After Dieter Rams)

1. **Useful**, solve one problem clearly
2. **Honest**, don't make things look more capable than they are
3. **Unobtrusive**, the UI should never compete with the content
4. **Understandable**, the user should never have to wonder
5. **Consistent**, the same affordance always behaves the same way
6. **Thorough**, every edge case considered, every error state designed
7. **Long-lasting**, avoid trendy design that dates immediately
8. **Accessible**, designed for the full range of human ability
9. **Minimal**, less, but better, every element earns its place
10. **Good**, leave the user better off than when they arrived

---

## Remember

Reference the best visual designs that exist: read Refactoring UI for tactics, absorb Dieter Rams for philosophy, test your assumptions against Nielsen's heuristics. Every design decision must answer the question: *does this serve the user, or does it serve the designer?*

The result must be production-ready, visually distinguished design that:

- Passes a senior designer's review on craft and intentionality
- Works flawlessly across all devices and input types
- Communicates clearly before the user reads a word
- Feels intentional and brand-specific, not generic template chrome
- Treats accessibility as a baseline, not a feature
