# gsc-indexing-debugger

Diagnoses Google Search Console indexing issues — wrong canonical selected,
"Crawled - currently not indexed", duplicate-content flags, sitemap
warnings, and more — using evidence from your live site and (optionally)
the real Search Console API, instead of guessing from a ticket.

**Read-only by default. Never modifies application code or requests
indexing without your explicit approval.**

Full phase-by-phase workflow: [`SKILL.md`](SKILL.md).

## How to use

```
Use the gsc-indexing-debugger skill to investigate:
https://www.example.com/some-page
Do not modify code.
```

```
Affected URL: https://www.example.com/page-a
Google-selected canonical: https://www.example.com/page-b
```

**Bulk CSV export:** attach/paste a GSC Pages-report export and say "in bulk
mode" — it clusters and shows a sampling plan before deep-diving anything.

**Follow-up verification:**
```
Recheck the previous investigation for /some-page — has Google recrawled it yet?
```

## Optional: GSC API setup

Without this, GSC evidence is manual (paste URL Inspection text/screenshots/
CSV) and the tool works fine either way. Set this up for automated live
Search Console state:

1. In [Google Cloud Console](https://console.cloud.google.com/), create a
   service account (no project roles needed) and enable the **Search
   Console API**.
2. **Keys → Add Key → Create new key → JSON**, download it.
3. In [Search Console](https://search.google.com/search-console) → your
   property → **Settings → Users and permissions → Add user** → paste the
   service account's `client_email` → grant **Restricted** (read-only).
4. Save the JSON key as `.gsc-service-account.json` next to this skill's own
   `SKILL.md`/`scripts/` (wherever you've copied this skill to — not
   necessarily your project's own repo root; `defaultKeyPath()` in
   `scripts/lib/gsc-api-client.cjs` resolves relative to this skill's own
   folder, two levels up from `scripts/lib`), gitignored — never commit it —
   or set `GSC_SERVICE_ACCOUNT_KEY_PATH` to point anywhere else.
5. For a Domain property, set `"domain"` in
   `gsc-indexing-debugger.config.json` (copy from the `.example.json` here)
   or pass `--site-url sc-domain:example.com`.

If no key is found, scripts print `{ "automated": false, ... }` and the
workflow falls back to manual GSC input automatically.

**Key rotation:** if a key is ever pasted into chat/email/Slack, treat it as
exposed and rotate it immediately.

## Scripts

| Script | Purpose |
|---|---|
| `fetch-as-googlebot.cjs` | Fetch a URL with a Googlebot UA; records redirects, status, headers, content hash |
| `check-stability.cjs` | Repeat-fetch a URL to test prerender/response stability |
| `extract-signals.cjs` | Extract title/canonical/meta/OG/JSON-LD/robots/hreflang |
| `extract-visible-text.cjs` | Extract visible text + word count + hash |
| `compare-pages.cjs` | Deterministic comparison facts between two URLs |
| `check-sitemap.cjs` | Confirm sitemap membership via the real sitemap index |
| `parse-gsc-export.cjs` | Cluster a bulk GSC CSV export for representative sampling |
| `gsc-api-inspect.cjs` | Call the real Search Console URL Inspection API (needs a service account) |
| `gsc-api-list-sitemaps.cjs` / `gsc-api-get-sitemap.cjs` | List/fetch sitemaps (prefer `.get()` — more reliable) |
| `gsc-api-average-position.cjs` | Average position/clicks/impressions/CTR by date |
| `gsc-api-position-movers.cjs` | Site-wide position comparison between two date ranges |

No npm install needed — plain Node, built-in modules only.

## What it does NOT do

- Does not modify application code.
- Does not request indexing without explicit approval.
- Does not deep-investigate every URL in a bulk export — clusters first and
  waits for approval before deep-diving representatives.

Output: a Markdown report (`templates/report-template.md`) plus a JSON
evidence file (`templates/evidence-schema.json`), saved under `reports/`
(gitignore this directory).
