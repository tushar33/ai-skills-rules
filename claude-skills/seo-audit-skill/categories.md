# Technical-SEO categories — generic, portable

Framework-agnostic. This file has **no hardcoded paths** — it defines what
to check and why, plus discovery hints for finding the relevant
implementation in an unfamiliar codebase. Copy this file (and `seo-audit.md`)
into any project's `skills/seo-audit/` unchanged. The project-specific
instance — actual file paths, current state, verification recipes for
*this* codebase — lives in a separate `project-checklist.md` that starts
empty (or nearly empty) in a new project and gets built up as the skill runs.

Each category: what it is → why it matters → generic detection approach →
discovery hints (where this typically lives, across a few common stacks) →
when it doesn't apply.

---

## SEO-01 — Search engine verification
**What:** Site-ownership proof (meta tag or static file) letting a search
engine's webmaster console (Google Search Console, Bing Webmaster Tools,
Yandex Webmaster, etc.) attribute crawl/index data to this property.
**Why:** Without it, that engine's indexing/crawl-error data is invisible —
issues go undetected until they show up as traffic loss.
**Generic detection:** Search the app's root HTML template for verification
meta tags (`msvalidate.01`, `google-site-verification`, `yandex-verification`,
etc.) and the static-assets root for verification stub files.
**Discovery hints:** CRA/Vite → `public/index.html` + `public/`; Next.js →
`app/layout.tsx`/`pages/_document.tsx` + `public/`; plain server app →
the base layout template + static file root.
**N/A when:** the property isn't meant to be publicly indexed at all (pure
internal tool, staging-only deploy).

---

## SEO-02 — Crawlable pagination for infinite-scroll / paginated listings
**What:** Any listing whose full content only reveals itself through
JS-driven infinite scroll needs a parallel, crawlable path: visible
`<a href>`/`<Link>` Prev/Next pagination (e.g. `?page=N`) plus a canonical
tag that reflects the actual page, not always page 1.
**Why:** Crawlers don't scroll. Without crawlable links, only the first
batch of any infinite-scroll listing is ever indexed.
**Generic detection:** Find listing/grid components that fetch page 2+ only
in response to a scroll or "load more" event handler, then check whether
each also renders a real `<a>`/`<Link>` to the next/prev page and sets its
canonical tag accordingly.
**Discovery hints:** grep for infinite-scroll library usage or hook names
(`useInfiniteScroll`, `react-infinite-scroll`, `IntersectionObserver` on a
sentinel element) and cross-check against grep for `?page=` link generation
nearby.
**N/A when:** the listing's total item count always fits on one screen (no
real pagination need).

---

## SEO-03 — Responsive images (srcset/sizes, modern formats)
**What:** Serving multiple image resolutions (`srcSet`/`sizes`) and, where
possible, modern formats (WebP/AVIF) so the browser picks the smallest
adequate image for the viewport/DPR.
**Why:** Oversized images are a leading cause of poor LCP/CLS — both Core
Web Vitals ranking factors — especially on mobile and image-dense grids.
**Generic detection:** Find the shared image component/wrapper every page
uses, and check whether it ever sets `srcSet`/`sizes` (or the framework's
equivalent — e.g. Next.js `<Image>`, Gatsby `gatsby-image`) versus always
requesting one fixed size.
**Discovery hints:** grep the shared image wrapper for `srcSet`, `sizes`,
`<picture>`, or a framework-native responsive-image component; check
whether the CDN/asset pipeline even supports arbitrary-width requests (a
prerequisite before `srcSet` can be generated).
**N/A when:** the CDN/build pipeline has no way to request alternate widths
at all (fix that first).

---

## SEO-04 — Canonical tag correctness
**What:** Every page emits exactly one `<link rel="canonical">`, and it
reflects the actual resolved URL (including relevant query params like
pagination), not a hardcoded or path-only guess.
**Why:** A missing, duplicated, or wrong canonical causes search engines to
pick their own — often the wrong page for indexing/ranking.
**Generic detection:** Find the component/hook that injects canonical tags
site-wide and check: (a) it runs on every route, (b) it's query-param-aware
where the route has meaningful query state, (c) there's no second, competing
canonical injection elsewhere (SSR template + client-side both writing one).
**Discovery hints:** grep for `rel="canonical"` or a helmet/head-management
library usage (`react-helmet`, `next/head`, framework-native `<Head>`).
**N/A when:** page has no indexable content variant (admin/auth-only route).

---

## SEO-05 — Structured data (JSON-LD) coverage & validity
**What:** Schema.org JSON-LD (`Organization`, `BreadcrumbList`, `Event`,
`Article`, `Product`, `VideoObject`, etc.) matching each page's actual
content type.
**Why:** Missing/wrong structured data forfeits rich-result eligibility
(breadcrumbs, event dates, video thumbnails in search results) and a wrong
`@id`/`url` can misattribute content to a different entity.
**Generic detection:** Find where JSON-LD is injected (DOM script tag or
head-management library) and verify it fires per-route with the right
`@type` and that dynamic fields (`@id`, `url`, `name`) match the current
entity, not a stale/shared default.
**Discovery hints:** grep for `application/ld+json` or `schema.org`.
**N/A when:** route has no content type with a meaningful schema.org
mapping.

---

## SEO-06 — Meta title & description coverage
**What:** Every indexable page has a unique, descriptive `<title>` and
`<meta name="description">` — not a generic app-wide default repeated
everywhere, not a title template that's technically unique per entity
(e.g. `{name} | {brand}`) but carries zero descriptive/category context
(role, content type, location) that would help a snippet stand out or
signal topical relevance, and not a tag that's simply too thin to be
descriptive even when it's non-empty and technically unique — e.g. a
description of only 3-8 words, or a title that's just a bare name/number
with no other word in it. A present-but-thin value passes a naive
"is this field empty?" check while still failing the actual intent of the
category.
**Why:** Duplicate/missing titles-descriptions read as low-quality/
boilerplate to search engines and produce poor search-result snippets. A
title that's unique-by-interpolation-only still under-communicates topic —
the same class of harm as a duplicate title, just harder to catch by diffing
alone. A thin-but-present value is a third, distinct failure mode from
either of those: it won't show up in a duplicate-content diff (it may be
unique per page) and won't necessarily show up as a template-wide pattern
(it can be a one-off, data-dependent shortfall — e.g. an entity whose
source description field itself only ever had a few words), so it needs its
own explicit check rather than being assumed to be caught by the other two.
**Generic detection:** Sample several distinct entity/content pages and
diff their rendered `<title>`/description — if they're identical across
clearly different content, that's the gap. Separately, check whether the
title template itself ever varies its *shape* per entity type/category, or
is always exactly `{name} | {brand}` for every entity regardless of type —
the latter is a template-wide content gap, not a one-off. For descriptions,
measure the rendered length against the intended budget (~155-160 chars) —
a description consistently landing well under budget (e.g. 15-20+ chars
short) across many pages is a symptom worth investigating: check whether
more than one truncation/formatting step runs on the same string before it
reaches the page (see the sequential-double-truncation bug signature in
`/audit-mistakes` if this codebase has one). Independently of the
budget-shortfall check, sample rendered title/description word counts
directly (not just character totals against the budget) and flag any that
are thin in absolute terms — roughly under 5-8 words for a description or
under 2-3 words for a title beyond the entity name itself — since a value
can clear a lenient char-count check while still reading as too sparse to
be a genuine descriptive snippet; trace thin hits back to whether the
*source data* for that entity is itself sparse (a content gap, likely
recurring across many similar entities) vs. a one-off rendering bug.
**Discovery hints:** grep the head-management usage for where
title/description props are set (or not set, falling through to a
default); grep for every function that truncates/slices a description
string and trace whether more than one such function's output feeds into
the other for the same field.
**N/A when:** the page genuinely has no distinguishing content (e.g. a
generic "coming soon" stub).

---

## SEO-07 — Heading hierarchy
**What:** Each page has exactly one `<h1>`, headings nest in logical order
(no `h1` → `h3` skip, no multiple competing `h1`s), adjacent heading levels
don't repeat the exact same text (a real `h2` shouldn't be word-for-word
duplicated by a nested `h3` — that's the same label at two levels, not two
levels of real structure), and heading tags are used exclusively for actual
section/document structure — not as a styling shortcut for repeated
non-heading content (e.g. wrapping every individual data-field label or
every item in a long repeated list — bio fields, catalog/venue entries,
timeline entries — in an `h4` just because it happened to have the right
font size/weight).
**Why:** Confuses both accessibility tooling (screen-reader users navigate
by heading level, expecting each one to mark real structure) and search
engines' page-topic signal extraction. A heading tag repeated hundreds of
times for non-heading list items dilutes whatever legitimate topic signal
the real headings on the page carry.
**Generic detection:** For a sample of *fully-rendered* page templates
(scroll/expand everything a real user would — async tabs, accordions,
lazy-loaded sections — not just what's in the initial viewport), count
heading tags per level, diff adjacent levels' text for exact duplicates, and
sample a handful of each heading level's actual content: is each one really
a section title, or is it a per-item label inside a repeated list/grid?
**Discovery hints:** grep component library for typography wrapper
components (`H1`/`H2`/etc.) and check how many render per page template;
grep list/grid-rendering components (`.map(...)` over an array) for a
heading-level wrapper applied to every item — that's the styling-shortcut
smell. Note: a fully-rendered manual/browser count can differ from what a
crawler's snapshot actually contains — cross-check against SEO-15 before
concluding the *indexed* page has the same heading profile as what a human
sees.
**N/A when:** N/A doesn't really apply — every page should pass this.

---

## SEO-08 — Image alt text coverage
**What:** Every content-bearing `<img>` has a non-empty, descriptive `alt`.
**Why:** Accessibility + image-search visibility signal.
**Generic detection:** grep the shared image component for whether `alt` is
a required prop with no default fallback to empty string, and sample actual
call sites for missing/generic (`alt="image"`) values.
**Discovery hints:** grep the shared image wrapper's prop list.
**N/A when:** purely decorative images (should have `alt=""`, which is
correct, not a gap).

---

## SEO-09 — Robots.txt & sitemap health
**What:** `robots.txt` doesn't accidentally block indexable content, its
`Sitemap:` directive resolves, and the sitemap's URLs are live and return
200 (not stale 404s/redirects).
**Why:** A silently broken or stale sitemap starves crawlers of new/updated
URLs; an overly broad `Disallow` can de-index entire sections by accident.
**Generic detection:** Fetch `robots.txt`, confirm every `Disallow` is
intentional (not an accidental prefix match), fetch the sitemap URL it
points to, and spot-check a sample of its entries for 200 status.
**Discovery hints:** `public/robots.txt`, `public/sitemap.xml`, or a
backend-generated sitemap route.
**N/A when:** no public content is meant to be indexed at all.

---

## SEO-10 — hreflang / i18n signals
**What:** For multilingual sites, `hreflang` alternates linking each
language variant of a page to the others (including `x-default`).
**Why:** Without it, search engines may serve the wrong language variant to
users, or treat variants as duplicate content.
**Generic detection:** Check whether pages with a language/locale switcher
also emit `<link rel="alternate" hreflang="...">` tags for each variant.
**Discovery hints:** grep for `hreflang`, or the i18n library's routing
config for locale-prefixed routes.
**N/A when:** site is genuinely single-language with no locale routing.

---

## SEO-11 — Redirect chains & internal broken links
**What:** Internal links point directly at final canonical URLs (no
A→B→C redirect chains), and no internal link 404s.
**Why:** Redirect chains waste crawl budget and dilute link equity; broken
internal links are dead ends for both users and crawlers.
**Generic detection:** Sample a set of internal links, follow each to its
final destination, and count redirect hops; separately check for any
generated link pointing at a route that doesn't resolve.
**Discovery hints:** grep for `<Link to=`/`<a href=`/`router.push` patterns
that build URLs from data rather than static routes (highest-risk for
staleness after entity renames/merges).
**N/A when:** site has no legacy-URL history (nothing to have accumulated
redirects from).

---

## SEO-12 — HTTPS / mixed content
**What:** Every asset (images, scripts, iframes) loads over HTTPS on an
HTTPS page — no `http://` sub-resource requests.
**Why:** Browsers block or warn on mixed content; it's also a minor ranking
signal and an outright broken-page risk.
**Generic detection:** grep rendered output / source for hardcoded
`http://` URLs in asset-loading contexts.
**Discovery hints:** CDN base-URL env vars, hardcoded legacy asset URLs in
CMS-authored HTML blocks.
**N/A when:** site is intentionally HTTP-only internal tooling.

---

## SEO-13 — Core Web Vitals signals (LCP priority, CLS sources, lazy-load strategy)
**What:** The largest above-the-fold image/text block is prioritized
(preload hint / `fetchpriority="high"`, not lazy-loaded), layout-shifting
elements (ads, late-loading embeds, web fonts) reserve their space
up-front, and below-the-fold media is lazy-loaded.
**Why:** LCP and CLS are direct Google ranking factors as well as UX
metrics.
**Generic detection:** Identify the page's LCP candidate (usually a hero
image or heading) and confirm it isn't lazy-loaded; check for
explicit width/height (or aspect-ratio) on images/embeds to prevent shift.
**Discovery hints:** grep the shared image component for a
priority/eager-load prop; check root HTML template for LCP preload
scripting.
**N/A when:** page has no above-the-fold media (rare).

---

## SEO-14 — URL structure & duplicate-content-prone query params
**What:** Clean, stable, human-readable URLs; any query param that doesn't
change page content (session IDs, tracking params, sort-order-only params
on an otherwise-identical listing) is either canonicalized away or excluded
from indexing signals.
**Why:** Unbounded param combinations create near-infinite crawlable
variants of the same content, diluting crawl budget and ranking signal.
**Generic detection:** Identify which query params actually change
rendered content vs. which are cosmetic/tracking-only, and confirm the
canonical tag strips the latter.
**Discovery hints:** grep the canonical-tag-building logic (see SEO-04) for
whether it includes `location.search` wholesale or selectively.
**N/A when:** site has no query-param-driven views at all.

---

## SEO-15 — JS-render / prerender coverage for crawlers
**What:** For any client-side-rendered (CSR) app, whether crawlers actually
receive a fully-rendered snapshot of each route's content — not a raw JS
shell that never executes, a stale snapshot, or a snapshot missing async
data/structured data that loads after initial paint.
**Why:** Some crawlers render JS reasonably well; many (Bing, most social
previewers, some Google edge cases) don't — or a rendering/prerender service
can silently regress per-route. A CSR app with no rendering fallback is
invisible to whichever crawlers can't execute its JS, and content that loads
async after the initial snapshot (JSON-LD, main text) may be missing even
when the crawler does render.
**Generic detection:** Identify whether the app is CSR, SSR, SSG, or
CSR-with-prerender-fallback. If CSR-with-fallback, fetch a sample of routes
with a crawler user-agent (or via the prerender service directly) and
confirm the response contains fully-rendered content — main text,
structured data, and meta tags all present — not an empty root div or a
loading skeleton. Don't stop at "is a page-ready flag wired up" — verify
the flag's own readiness condition is actually sufficient: a common,
easy-to-miss gap is a readiness check that's satisfied by top-level data
plus a coarse heuristic (e.g. "has the image count stopped growing"), while
a *specific subsection* of the same page — commonly one behind its own
code-split/lazy import and a loading-fallback boundary — is still resolving
independently when the snapshot is captured. This produces an intermittent,
page-dependent bug: some entities/routes render fully, others (hitting the
race) get captured mid-load showing their loading-fallback markup instead
of content, for no code-level reason specific to those entities.
**Discovery hints:** grep for a prerender/rendering-service integration
(middleware, edge function, or a "is this a bot" user-agent check), and for
any signal the app itself exposes to tell the renderer "the page is ready"
(e.g. a `window`-scoped ready flag flipped after async data + structured
data are in the DOM). Then grep the pages using that readiness signal for
any child rendered via a lazy/code-split import (`lazy(() => import(...))`
or framework equivalent) inside a loading-fallback boundary (`Suspense` or
similar) — cross-reference whether the readiness condition actually depends
on that specific import resolving, or only on unrelated top-level state. To
confirm live: fetch a sample of affected-template routes with a crawler
user-agent and grep the response for the shared loading-fallback component's
markup/text still present inside a named container — if found on some
routes but not others of the same template, that's this race, not a
one-off content issue.
**N/A when:** the app is server-rendered or statically generated — content
is already present in the initial HTML response for every crawler.

---

## SEO-16 — Open Graph / Twitter Card social meta tags
**What:** `og:title`, `og:description`, `og:image`, `og:url`, `og:type`,
and `twitter:card`/`twitter:image` (or the platform-appropriate equivalent)
set per page, distinct from the plain `<title>`/meta description (SEO-06).
**Why:** These control how a link looks when shared on social platforms,
in chat apps, and in some search/aggregator surfaces. Missing or generic
(app-wide default) tags produce a blank or wrong preview image/title
site-wide, hurting click-through on shared links.
**Generic detection:** Find where `<head>` meta tags are set per route and
check whether Open Graph/Twitter tags are included alongside the plain
title/description, and whether `og:image`/`twitter:image` resolve to a
real, entity-specific image rather than one shared fallback for every page.
**Discovery hints:** grep for `og:`/`twitter:` meta tag literals, or the
head-management library's usage (same place SEO-06 and SEO-04 live).
**N/A when:** content is never meant to be shared/linked externally
(pure internal tool).

---

## SEO-17 — Per-page `<meta name="robots">` directives
**What:** Page-level `robots` meta tag (or equivalent HTTP header) —
`noindex`/`nofollow`/`index,follow` — set correctly per route, distinct
from the site-wide `robots.txt` file (SEO-09).
**Why:** `robots.txt` only controls *crawling*; it doesn't stop a URL from
being *indexed* if it's linked from elsewhere. Pages that should never
appear in search results (internal search-results pages, duplicate/thin
utility pages, auth-gated content that occasionally renders a public shell)
need an explicit `noindex`, and pages that should be indexed must not
carry a stray `noindex` left over from a template default or a debugging
change.
**Generic detection:** Sample a few page templates — especially ones
already `Disallow`'d in `robots.txt` or otherwise low-value (search
results, "coming soon" stubs, thin/duplicate listing pages) — and check
whether they emit their own `noindex`, or wrongly rely on the
crawl-block alone. Separately check that real content pages never emit an
unintended `noindex`.
**Discovery hints:** grep for `name="robots"` or a head-management prop
(e.g. `noindex={...}`) and check its default value when not explicitly
passed — a default of `noindex` applied too broadly is as much a bug as a
default of `index` applied to something that shouldn't be.
**N/A when:** the property is single-purpose and every route should behave
identically (rare).

---

## SEO-18 — Viewport meta tag correctness
**What:** `<meta name="viewport" content="width=device-width, initial-scale=1">`
(or equivalent) present in the document head — not missing, not fixed to a
non-responsive pixel width, and not disabling user zoom
(`user-scalable=no`, or `maximum-scale=1` used to the same effect).
**Why:** Google indexes and ranks primarily off the mobile rendering of a
page (mobile-first indexing). A missing or misconfigured viewport tag makes
the page render as a zoomed-out desktop layout on mobile, which can trip a
direct "not mobile-friendly" flag and hurts mobile usability signals more
broadly. Disabling zoom is also a WCAG 1.4.4 (reflow/zoom) accessibility
failure, not just an SEO one.
**Generic detection:** Fetch the rendered `<head>` for a sample of pages and
confirm a viewport meta tag is present, uses `width=device-width` (not a
fixed pixel value), and doesn't hard-disable zoom.
**Discovery hints:** grep the root HTML template — `public/index.html` for
CRA/Vite, `app/layout.tsx`/`pages/_document.tsx` for Next.js — for
`name="viewport"`. This tag is almost always static/site-wide rather than
per-route, so one check usually covers the whole app unless a per-page
override exists.
**N/A when:** the site is intentionally non-responsive or print-only
(rare).

---

## SEO-19 — Resource hints for LCP-critical origins (preconnect/dns-prefetch/preload)
**What:** Third-party origins the page depends on early in the render path
— image/asset CDN, webfont host, primary API host — are hinted via
`<link rel="preconnect">` or `dns-prefetch` in the document head, and the
actual LCP candidate asset (or a render-blocking critical webfont) is
hinted via `<link rel="preload">` where applicable.
**Why:** Without a preconnect hint, the browser only begins DNS + TCP + TLS
setup for a third-party origin once it discovers a resource from it mid-
parse — adding avoidable round trips squarely on the path to the Largest
Contentful Paint. This overlaps with SEO-13 (Core Web Vitals) but merits
its own explicit check: it's commonly skipped even after image-sizing and
lazy-load work (SEO-03, SEO-13) is otherwise done, since it lives in the
root HTML template rather than the component that actually renders the LCP
image.
**Generic detection:** Identify the LCP candidate's origin (usually the
image/asset CDN) and any third-party font host, then check the root HTML
template for matching `preconnect`/`dns-prefetch` hints; separately check
whether the LCP asset itself or a critical webfont has a
`<link rel="preload">`.
**Discovery hints:** grep the root HTML template for `rel="preconnect"`,
`rel="dns-prefetch"`, `rel="preload"`; cross-reference the hinted origins
against the CDN base URL used by the shared image component (see SEO-03)
and any `@font-face`/font-loading configuration.
**N/A when:** all critical assets are already same-origin (nothing
third-party to hint), or the page has no asset on the LCP path (rare).

---

## Growing this file
Add a new category here **only if it's a universal technical-SEO practice**
applicable across projects/stacks — no project-specific paths, no
stack-specific library names as the only detection method (name a stack's
tool as an example, not as the requirement). Project-specific findings
(paths, current state, dates) belong in that project's `project-checklist.md`
instead.
