# pagespeed-debugger

Diagnoses real PageSpeed Insights / Core Web Vitals problems (slow LCP, high
TBT, poor CLS, a low Lighthouse performance score) on a **live URL**, using
actual Lighthouse data instead of guesses from reading source code.

**Read-only by default. Never modifies application code and never touches
the target site beyond a normal page request.**

Works on any site, in any project, with any framework.

Full workflow: [`SKILL.md`](SKILL.md).

## How to use

```
Use the pagespeed-debugger skill on https://example.com/slow-page
```

```
Use the pagespeed-debugger skill to compare https://example.com/page
before and after today's deploy.
```

```
Use the pagespeed-debugger skill on these 3 URLs, look for a shared cause:
https://example.com/a
https://example.com/b
https://example.com/c
```

## Setup — optional, three tiers

**Tier 1 — no setup.** `collect.cjs` falls back to running a real Lighthouse
audit locally via `npx lighthouse` (needs local Chrome). No account, no key,
no quota. Enough for most one-off investigations.

**Tier 2 — a free PageSpeed Insights API key**, for faster/scriptable
repeated runs or field (real-user CrUX) data:

1. [console.cloud.google.com](https://console.cloud.google.com) → create a
   project → enable **PageSpeed Insights API** → create an API key.
2. Restrict the key to that API. If **Application restrictions** is set to
   "HTTP referrers" it will reject CLI calls — set it to "None" or "IP
   addresses".
3. Use it via `export PSI_API_KEY=<key>` (recommended), `--key <key>`, or a
   gitignored `.psi-api-key` file next to this skill's folder.

**Key rotation:** if a key is ever pasted into chat or Slack, treat it as
exposed and regenerate it in Cloud Console.

**Tier 3 — fully manual.** If neither is available, `collect.cjs` prints
exact steps to run [pagespeed.web.dev](https://pagespeed.web.dev) or Chrome
DevTools' Lighthouse panel by hand.

## Scripts

```bash
# Collect a report (tries PSI-with-key -> local Lighthouse -> PSI-anonymous -> manual instructions)
node scripts/collect.cjs <url> [--strategy mobile|desktop] [--key <key>] [--out <path>]

# Turn a saved report into a ranked bottleneck breakdown
node scripts/diagnose.cjs <path-to-report.json>

# Compare two saved reports (before/after, or two URLs)
node scripts/compare.cjs <reportA.json> <reportB.json>
```

Reports are saved under `reports/pagespeed-debugger/raw/` by default
(gitignore this directory). Requires Node 18.20+ (the pinned local-Lighthouse
version's own minimum; `collect.cjs`'s PSI/anonymous paths need only global
`fetch`, available since Node 18.0), no npm dependencies.

## What it does NOT do

- Does not modify application code automatically.
- Does not deploy, invalidate caches, or touch the target site beyond a
  normal page request per audit run.
