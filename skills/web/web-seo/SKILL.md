---
name: web-seo
description: >
  Use this skill when making a website discoverable and shareable: meta tags, Open
  Graph and Twitter cards, canonical URLs, robots.txt, sitemap.xml, JSON-LD
  structured data, per-route metadata in a single-page app, and soft-404 handling.
  Covers the traps that silently break sharing (relative og:image, duplicated meta
  tags, SPA catch-all rewrites returning 200, CSP and inline JSON-LD) and the rule
  that every URL-bearing artifact must derive from one build-time constant. Trigger
  on "add SEO", "SEO audit", "og:image not showing", "link preview is broken",
  "add structured data", "rich results", "sitemap", "robots.txt", "canonical",
  "meta description", "share preview", or any pre-launch checklist for a public
  site. Framework-agnostic; build-tool notes for Vite, Next.js, and static hosts.
---

# Web SEO

SEO for a product site is not keyword work. It is a small set of machine-readable
facts — what this page is, where it canonically lives, what it looks like when
shared, and what kind of thing it describes — expressed so crawlers and social
scrapers can read them **without running your JavaScript**.

That last clause drives almost every decision here. Most social scrapers do not
execute JS. Anything that must survive a share has to be in the served HTML.

## Authority (read first)

This skill defines the SEO contract. Craft rules for the code that implements it
come from `standards/Clean-Code/`, and failure handling from `skills/engineering/errors/SKILL.md` —
notably: a build step that silently produces an incomplete sitemap is exactly the
swallowed-failure anti-pattern that skill forbids. Warn loudly instead.

---

## Rule 0 — one URL constant, derived not typed

Every artifact below embeds an absolute origin: canonical, `og:url`, `og:image`,
JSON-LD `url`, `robots.txt`'s sitemap line, and every `<loc>` in the sitemap. If
those are typed in more than once they will drift, and a wrong canonical is worse
than no canonical.

Define it **once**, in the build config, and prefer the host's own environment
variable so it is correct without anyone maintaining it:

| Host | Build-time variable | Notes |
|---|---|---|
| Vercel | `VERCEL_PROJECT_PRODUCTION_URL` | Stable production domain; switches to a custom domain automatically once attached. Not `VERCEL_URL` — that is per-deployment. |
| Netlify | `URL` | `DEPLOY_PRIME_URL` for branch previews. |
| Cloudflare Pages | `CF_PAGES_URL` | Per-deployment; set your own var for production. |
| Static / other | Your own env var | Fall back to a known-correct literal. |

```ts
const productionDomain = process.env.VERCEL_PROJECT_PRODUCTION_URL;
const SITE_URL = productionDomain
  ? `https://${productionDomain}`
  : "https://known-good-fallback.example";
```

**Never guess a domain.** A plausible-looking `<project-name>.vercel.app` is very
often already taken by an unrelated project, and pointing your canonical and OG
tags at a stranger's site is an actively harmful "fix". Verify before you commit:

```bash
curl -s -L --max-time 15 "https://candidate.example" | grep -oiE "<title>[^<]*</title>"
```

If the title is not the product, it is not the domain.

---

## The checklist

### Tier 1 — breaks sharing if wrong

- [ ] **`og:image` is an absolute URL.** Relative paths are the single most common
      cause of "our link preview has no image". Same for `twitter:image`.
- [ ] `og:image` is reachable, ~1200×630, and declares `og:image:width` / `:height`
      so scrapers can lay out before download.
- [ ] `og:type`, `og:title`, `og:description`, `og:url`, `og:site_name`, `og:locale`
- [ ] `twitter:card` = `summary_large_image` (plus title / description / image)
- [ ] `<link rel="canonical">` on every route
- [ ] `<title>` and `<meta name="description">` unique per route
- [ ] `<html lang>` set

### Tier 2 — discoverability

- [ ] `robots.txt` exists and names the sitemap
- [ ] `sitemap.xml` exists, lists every real route, and is generated not hand-written
- [ ] JSON-LD describing what the page actually is
- [ ] Dynamic routes appear in the sitemap
- [ ] Non-canonical routes (404, search results, thin pages) are `noindex`

### Tier 3 — performance and polish

- [ ] `preconnect` / `dns-prefetch` to any origin serving above-the-fold assets
- [ ] `theme-color` matches the actual page background
- [ ] Heading hierarchy is one `h1` per page, then descending
- [ ] Meaningful `alt` on content images; `alt=""` on decorative ones

---

## Generate, never hand-maintain

`robots.txt` and `sitemap.xml` are derived data. Hand-written copies rot the first
time a route is added. Emit them from the build using the one URL constant.

Vite plugin sketch (Rollup's `emitFile` in `generateBundle`):

```ts
function seoAssets(): Plugin {
  return {
    name: "seo-assets",
    transformIndexHtml(html) {
      return html.replaceAll("__SITE_URL__", SITE_URL);
    },
    async generateBundle() {
      this.emitFile({
        type: "asset",
        fileName: "robots.txt",
        source: `User-agent: *\nAllow: /\n\nSitemap: ${SITE_URL}/sitemap.xml\n`,
      });
      // ...emit sitemap.xml the same way
    },
  };
}
```

A `__SITE_URL__` placeholder in `index.html` plus `transformIndexHtml` keeps the
static HTML on the same constant as everything else. Expose it to app code with
`define: { __SITE_URL__: JSON.stringify(SITE_URL) }` and declare it in your
ambient types.

### Dynamic routes in the sitemap

If routes come from a data source (CMS, changelog, catalog), fetch and enumerate
them at build time — and **reuse the app's own parser** rather than writing a
second one that can disagree with what the app renders.

Guard the network call, but do not swallow the failure:

```ts
try {
  const response = await fetch(SOURCE, { signal: AbortSignal.timeout(10_000) });
  if (!response.ok) {
    warn(`sitemap: source returned ${response.status}; dynamic routes omitted`);
    return [];
  }
  return parseRoutes(await response.text());
} catch (error) {
  warn(`sitemap: fetch failed (${String(error)}); dynamic routes omitted`);
  return [];
}
```

A build that quietly ships a two-entry sitemap is a bug you will not notice for
months.

---

## Structured data (JSON-LD)

Pick the type that honestly describes the page. Common ones:

| Page | Type |
|---|---|
| Downloadable app or tool | `SoftwareApplication` |
| FAQ / help page | `FAQPage` |
| Article, changelog entry, post | `Article` / `TechArticle` |
| Company or project site | `Organization` / `WebSite` |
| Store item | `Product` |

**Generate it from the same data the UI renders.** If the FAQ component maps over
a `faq` array, the `FAQPage` schema must map over that identical array at build
time. Schema hand-copied from the UI is schema that will contradict the UI.

```ts
function faqSchema(faq: readonly { question: string; answer: string }[]) {
  return JSON.stringify({
    "@context": "https://schema.org",
    "@type": "FAQPage",
    mainEntity: faq.map((item) => ({
      "@type": "Question",
      name: item.question,
      acceptedAnswer: { "@type": "Answer", text: item.answer },
    })),
  }).replaceAll("</", "<\\/");
}
```

That `replaceAll("</", "<\\/")` is not decoration — an answer containing `</script>`
would otherwise terminate the script tag early.

### Never fabricate

Do not emit `aggregateRating`, `review`, or invented counts to farm stars in search
results. It is against every search engine's guidelines, it is a manual-action risk,
and for a small product it is a trust problem you cannot undo. Mark up only facts
that are true and visible on the page.

---

## Single-page apps

### Per-route metadata: mutate, do not append

Frameworks that hoist metadata from components (React 19's native `<title>` /
`<meta>` support among them) **append** tags. If `index.html` already ships a
static `og:description` — and it must, for scrapers — a hoisted one produces two
conflicting tags.

Upsert instead, so there is exactly one of each:

```ts
function upsertMeta(attribute: "name" | "property", key: string, value: string) {
  let tag = document.head.querySelector<HTMLMetaElement>(`meta[${attribute}="${key}"]`);
  if (!tag) {
    tag = document.createElement("meta");
    tag.setAttribute(attribute, key);
    document.head.appendChild(tag);
  }
  tag.setAttribute("content", value);
}
```

Keep the static tags in `index.html` as the crawler-visible default, and let the
runtime hook specialise them per route for engines that do render JS.

### Soft 404s

An SPA catch-all rewrite (`/(.*) → /index.html`) returns **HTTP 200 for every
unknown URL**. Search engines call this a soft 404 and treat it as a quality
signal. Unless your host can return a real 404 status for unmatched routes, the
standard mitigation is to set `noindex, follow` on the 404 route so rendering
engines drop it.

### Static-file precedence

Confirm your catch-all rewrite does not swallow `robots.txt` and `sitemap.xml`.
Most hosts (Vercel included) check the filesystem before applying rewrites, so
real emitted files win — but verify rather than assume, because the failure is
silent and looks like a working site.

---

## CSP and JSON-LD

A strict `script-src 'self'` blocks inline scripts, which looks like it should
block `<script type="application/ld+json">`. It does not: a non-JavaScript MIME
type makes the element a **data block**, not executable script, so `script-src`
does not govern it. Strict-CSP sites ship JSON-LD normally.

Still verify in a real browser console after adding the CSP header — a
misconfigured `require-trusted-types-for` or a proxy rewriting the type can
change the outcome.

---

## Framework notes

| Stack | Approach |
|---|---|
| **Vite SPA** | `transformIndexHtml` for static tags, a plugin emitting robots/sitemap, `define` for the app-side constant, a runtime hook for per-route meta. |
| **Next.js (App Router)** | `metadata` / `generateMetadata` exports; `app/robots.ts` and `app/sitemap.ts` generate both files. Set `metadataBase` so relative OG images resolve to absolute — this is the framework's answer to Rule 0. |
| **Astro / SSG** | Emit per-page tags at build; every page already has its own HTML, so most SPA caveats do not apply. |
| **Plain static** | Hand-write per-page tags but still generate robots/sitemap from a script. |

---

## Verification

Check the **built output**, never the source — placeholders that fail to
substitute are invisible in source and obvious in `dist/`.

```bash
# no unreplaced placeholders survived
grep -c "__SITE_URL__" dist/index.html          # expect 0

# og:image is absolute
grep -o 'og:image" content="[^"]*"' dist/index.html

# structured data landed, with the expected item count
grep -o '"@type":"Question"' dist/index.html | wc -l

# sitemap contents
grep -o "<loc>[^<]*</loc>" dist/sitemap.xml

# files actually serve, with correct content types
npx vite preview --port 4173 &
for p in / /robots.txt /sitemap.xml /og-image.png; do
  curl -s -o /dev/null -w "$p %{http_code} %{content_type}\n" "http://localhost:4173$p"
done
```

Then, before announcing launch:

- Google Rich Results Test — validates JSON-LD and previews eligibility
- Facebook Sharing Debugger / X Card Validator — forces a re-scrape and shows
  exactly what a scraper resolved, which is the fastest way to catch a relative
  `og:image`
- Submit the sitemap in Google Search Console

---

## Anti-patterns

| Anti-pattern | Why it hurts |
|---|---|
| Relative `og:image` | Preview renders with no image on most platforms |
| Domain typed into several files | Guaranteed drift; a wrong canonical de-indexes pages |
| Guessing the production domain | The plausible subdomain is frequently someone else's live site |
| Hand-maintained `sitemap.xml` | Stale the first time a route is added |
| Schema hand-copied from UI copy | Contradicts the page as soon as either changes |
| Fabricated ratings or review counts | Guideline violation and a manual-action risk |
| Hoisted per-route meta on top of static tags | Duplicate, conflicting tags |
| Keyword-stuffed `<title>` | Truncated in results and reads as spam |
| `noindex` shipped from a staging config | Silently removes the whole site from search |
