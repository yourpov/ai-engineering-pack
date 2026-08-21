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
- [ ] **The declared `og:image:width` / `:height` match the file's real pixels.**
      Scrapers reserve layout from the declaration and then fit the actual image
      into it. Declaring 512×512 for a 372×279 file produces the "huge awkward
      crop" that looks like a broken card. Measure, do not assume:
      `ffprobe -v error -show_entries stream=width,height -of csv=p=0 og-image.png`
- [ ] `og:image:type` and `og:image:alt` set
- [ ] **The OG image is not the favicon.** Check they are not the same file —
      `cmp -s public/og-image.png public/favicon.png && echo "SAME FILE"`. An icon
      reused as a share card is the most common cause of an ugly preview.
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

### Tier 2.5 — the favicon is a search result element

Google renders a favicon beside every result. It silently rejects icons that do
not meet its requirements, and falls back to a generic globe — which reads to the
site owner as "my image is missing" while every tag validates fine.

- [ ] **Square.** A non-square favicon is rejected outright. This is the usual cause.
- [ ] **A multiple of 48px** — 48×48, 96×96, 192×192. Ship several via `sizes`.
- [ ] **Visible on both light and dark chrome.** A black-on-transparent glyph
      disappears against a dark tab bar and in dark-mode search results. Give the
      icon an opaque background in the brand colour rather than shipping alpha.
- [ ] `apple-touch-icon` is 180×180 and **opaque** — iOS composites transparency
      onto black.
- [ ] Reachable at a stable URL and not blocked by `robots.txt`.

```bash
# square? multiple of 48?
for f in public/favicon*.png public/apple-touch-icon.png; do
  printf "%-28s " "$f"; ffprobe -v error -show_entries stream=width,height -of csv=p=0 "$f"
done
```

### Tier 2.75 — the tags are 20% of the job

A perfect technical layer on a thin site ranks for the brand name and nothing
else. Before congratulating yourself on a clean audit, check the things markup
cannot fix:

- [ ] **There is something to rank.** Nobody searches your product name until they
      already know it. If no page answers the query you want to win, no tag will
      make one.
- [ ] **The content is in the served HTML.** A client-side `fetch` that renders
      your richest keyword surface — a project list, a catalogue, a changelog —
      is invisible to everything that does not run JS, and unreliable even for
      Google. Snapshot it at build time.
- [ ] **`<meta name="keywords">` is deleted.** Every major engine has ignored it
      since 2009. It only signals that someone copied a 2006 checklist.
- [ ] **Payload is not the bottleneck.** Core Web Vitals are ranking signals.
      Measure the built bundle, not the vibe:
      `npm run build` and read the chunk table. Split vendors that change on a
      different cadence than your app, so a copy tweak does not invalidate the
      framework chunk.
- [ ] **Authority is acknowledged, not assumed.** Sitelinks, rich results, and
      competitive rankings need inbound links and traffic. Clean markup removes
      the excuses; it does not substitute for them.

Say this plainly to whoever asked for the audit. A technically perfect, contentless
site is a common and expensive misunderstanding of what SEO is.

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

## The description you write is a suggestion, not a guarantee

Google frequently ignores `meta name="description"` and synthesises a snippet from
on-page text instead — it does this when the description looks thin, keyword-ish,
or less relevant to the query than the body copy. The result is often two
unrelated fragments stitched together, reading like keyword salad:

> Full-stack developer and audio engineer. Bots, APIs, automation, Roblox systems,
> and products. Full-stack development with Go, TypeScript, and Python. Discord
> bots, Telegram bots, REST…

Neither half of that came from a meta tag; both were scraped from the page.

**So when a snippet reads badly, fix the on-page copy — editing the meta tag alone
will not change it.** Grep the site for the exact fragment to find its real source:

```bash
grep -rn "the exact phrase from the SERP" src/
```

Two practical consequences:

- **The first substantive paragraph is SEO copy**, whether you intended it or not.
  Write it as a sentence a human would read aloud, not a comma-separated skills list.
- **A charming one-liner description is a trade-off.** "Learn more about me, connect,
  or work with me." is excellent for a Discord embed and gives Google nothing to
  prefer over body text. If you want the description honoured, it needs enough
  substance to beat the page copy on relevance — pair the hook with the specifics.

### Sitelinks cannot be requested

The indented sub-links under a result (Login, Services, Blog…) are generated
algorithmically from site structure, internal linking, and traffic. There is no
markup that produces them and no way to opt in. What you can do is remove the
reasons Google withholds them: unique `<title>` per route, every route in the
sitemap, and real internal links to each. Duplicate titles across routes — the
default failure in an SPA where only some pages set metadata — are the most
common blocker.

---

## Single-page apps

### Serving real HTML to non-rendering crawlers

Google renders JS, so a runtime metadata hook is enough *for Google*. Discord,
Slack, Twitter, and most other scrapers do not — they read the served HTML, which
in an SPA is one `index.html` with one set of tags. Every route then shares the
homepage's title and description, and every share of `/games` looks like a share
of `/`.

On a serverless host you can close this without a framework migration: route the
content paths through a small handler that returns per-route tags to bots and the
normal shell to everyone else.

```ts
// vercel.json → { "source": "/games", "destination": "/api/preview?route=/games" }
const meta = ROUTE_SEO.find((r) => r.path === route);
if (meta && isBot(req.headers['user-agent'])) {
  return res.send(renderTags(meta));       // built from the same table the app uses
}
return res.send(readFileSync('dist/index.html', 'utf8'));
```

Derive the table from the same constant the client hook reads — two hand-kept
lists will disagree, and the disagreement is invisible until someone shares a link.

Costs to weigh: every request to those paths becomes a function invocation, and
user-agent sniffing is a heuristic that new crawlers fall outside of. Prefer real
SSG or SSR when the project can afford it; this is the retrofit.

### One route table, every consumer

Per-route metadata has at least four consumers — the `<title>` the browser shows,
the sitemap, the HTML served to crawlers, and the robots rules. Hand-maintaining
them separately guarantees a page that describes itself one way to users and
another to search engines, and nothing fails loudly when they diverge.

Define the routes once and derive everything:

```ts
export const ROUTE_SEO = [
  { path: "/", title: "…", description: "…", changefreq: "weekly", priority: 1.0 },
  { path: "/demos", title: "…", description: "…", changefreq: "monthly", priority: 0.8 },
];
```

Then `buildSitemap()` maps it, the bot handler looks up from it, and a single
`useRouteSeo(path)` hook applies it client-side. Adding a page becomes one edit.

**The failure this prevents is silent.** An SPA where only *some* routes set their
own title leaves the rest inheriting the homepage's — duplicate titles across
routes, which splits relevance and is the most common reason sitelinks never
appear. Verify by enumerating, not spot-checking:

```js
for (const route of ROUTES) {
  await page.goto(base + route);
  console.log(route, await page.title(), await canonicalOf(page));
}
```

Two identical titles in that output is a bug, not a style choice.

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

**Check the body, never the status code.** A dev server's SPA fallback answers
`200` for `/robots.txt` while serving `index.html`, so a status check passes on a
file that does not exist:

```bash
curl -s -o /dev/null -w "%{http_code} %{content_type}\n" localhost:5173/robots.txt
# 200 text/html   ← the 200 is a lie; text/html is the tell
curl -s localhost:5173/robots.txt | head -1
# <!doctype html> ← proof
```

Assert on content type and first line. `text/html` for `robots.txt` means the
fallback ate it.

### Generated files must also exist in dev

`generateBundle` runs at build only, so a plugin that emits `robots.txt` and
`sitemap.xml` leaves both missing during development — you cannot verify locally,
and the SPA fallback disguises it as a 200. Serve them from the same builders via
dev middleware so the two environments cannot disagree:

```ts
const SEO_FILES = {
  "/robots.txt": { type: "text/plain", body: () => buildRobots(SITE_URL) },
  "/sitemap.xml": { type: "application/xml", body: () => buildSitemap(SITE_URL) },
};

configureServer(server) {
  server.middlewares.use((req, res, next) => {
    const file = SEO_FILES[req.url?.split("?")[0] ?? ""];
    if (!file) return next();
    res.setHeader("Content-Type", file.type);
    res.end(file.body());
  });
},
generateBundle() { /* emit the same builders */ },
```

### Making the sitemap readable (optional)

An `<?xml-stylesheet type="text/xsl" href="/sitemap.xsl"?>` instruction after the
XML declaration renders the sitemap as a table for humans. Crawlers ignore the
instruction entirely, so it costs nothing at crawl time — it is purely for the
person who opens the file and sees the browser's "no style information" notice.

Namespace it correctly or the transform silently produces an empty page: the
sitemap lives in `http://www.sitemaps.org/schemas/sitemap/0.9`, so bind a prefix
(`xmlns:s=…`) and select `s:urlset/s:url`, never bare `urlset/url`.

### Crawling rules that are actually about security

`Disallow` and `noindex` solve different problems and are routinely swapped:

| Goal | Use | Why not the other |
|---|---|---|
| Keep a URL out of results | `noindex` | `Disallow` blocks the crawl, so the `noindex` is never seen — and a linked URL can still be indexed bare |
| Stop a crawler fetching at all | `Disallow` | `noindex` requires the fetch to happen |

**Where the URL is the credential — one-time links, OAuth callbacks, magic-link
tokens — you want `Disallow`.** A crawled token lands in logs and caches and may
be spent before its owner clicks it:

```
Disallow: /*?token=
Disallow: /*?state=
Disallow: /*?code=
```

Wildcards are honoured by Google, Bing, and Yandex. And remember `robots.txt` is
public: listing `/admin` advertises it. That is usually fine for a path already
visible in your JS bundle, but never treat the file as concealment.

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
| Favicon reused as `og:image` | An icon crops into a share card badly; also usually the wrong aspect |
| Declared OG dimensions that do not match the file | Scraper reserves the wrong box; preview looks broken |
| Non-square favicon | Google rejects it and shows a generic globe instead |
| Only some SPA routes setting their own title | Duplicate titles suppress sitelinks and split relevance |
| Editing `meta description` to fix a bad SERP snippet | Google may be synthesising from body copy; fix the copy |
| `<meta name="keywords">` | Ignored since 2009; signals a copied 2006 checklist |
| Checking a generated file with a status code only | SPA fallback returns 200 with `index.html`; assert on content type |
| `Disallow` on a page you wanted de-indexed | Blocks the crawl, so the `noindex` is never read |
| Crawlable one-time-token URLs | The URL is the credential; it ends up in logs and caches |
| Declaring the audit done at Tier 2 | Tags are ~20% of ranking; thin content is the real ceiling |
