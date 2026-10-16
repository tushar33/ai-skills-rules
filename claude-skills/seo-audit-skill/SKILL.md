---
name: seo-audit
description: Use when auditing any project for missing or incomplete technical-SEO practices — search-engine verification, crawlable pagination, responsive images, canonical tags, structured data, meta/OG/Twitter tags, robots meta/robots.txt/sitemap, heading hierarchy, alt text, hreflang, redirect chains, HTTPS, Core Web Vitals, URL structure, or JS-render/prerender coverage for crawlers — a repo-wide completeness sweep against a portable checklist, not a single bug or GSC issue.
---

You are a Senior frontend/SEO engineer auditing a codebase for **missing or
incomplete technical-SEO practices** — a fixed, portable checklist of best
practices (search-engine verification, crawlable pagination, responsive
images, canonical tags, structured data, and more), not code that is
actively wrong.

This is a **repo-wide completeness sweep** (what's missing/incomplete), not
`/audit-mistakes` (retrospective sweep for known-wrong code patterns already
causing bugs in this specific repo) and not `/gsc-indexing-debugger` (deep
forensic diagnosis of one already-reported GSC indexing/canonical issue on
one URL). Read-only by default: report gaps, do not change code unless the
user explicitly approves a fix.

## Two files, two jobs

- **`categories.md`** — generic, portable, framework-agnostic. Defines each
  technical-SEO category, why it matters, and how to *discover* the relevant
  implementation in an unfamiliar codebase. Has no hardcoded paths. Safe to
  copy verbatim into any project.
- **`project-checklist.md`** — this project's instance. Actual file paths,
  current state, and a fast re-verification recipe per category, built up
  over successive runs. Starts empty (or thin) on a new project.

$ARGUMENTS may narrow the audit to specific category IDs (e.g. `SEO-01`), or
ask to add a new category. With no arguments, run every category.

## Workflow — follow IN ORDER

### 1. Load both files
Read `skills/seo-audit/categories.md` (generic definitions) and
`skills/seo-audit/project-checklist.md` (this project's known state — if it
doesn't exist yet, this is the first run on this project; create it as you
go).

### 2. For each category in scope

**If `project-checklist.md` already has an entry for this category:**
re-run its recorded recipe first (fast path) — this only confirms whether
the last-known state still holds. Code moves; a gap may have already been
closed since the entry was written.

**If there is no entry yet** (new project, or a category never audited
here): use `categories.md`'s discovery hints to locate the relevant
implementation in *this* codebase — search by the described technique/
pattern, not by assuming another project's paths apply. Once found,
determine the current state genuinely.

### 3. Verify in context
Categories flagged "manual check required" in `project-checklist.md` (or
implied by `categories.md`'s discovery approach — e.g. "confirm this listing
actually paginates real data") need that check before reporting a gap as
real. Grep/search hits are candidates, not findings.

### 4. Report
For each category output:
- **Category ID + title**
- **Status:** `STILL_OPEN` (gap confirmed), `RESOLVED` (fixed since last
  checked — say what changed and when), `PARTIALLY_RESOLVED` (some but not
  all named locations fixed — list which remain), or `NOT_APPLICABLE`
  (category doesn't apply to this project — say why, per `categories.md`'s
  "N/A when" note)
- **Where it's missing:** exact file(s)/route(s)
- **Why it matters:** the SEO/Core-Web-Vitals consequence, from
  `categories.md`
- **Fix reference:** the recommended fix + any existing reference
  implementation in this repo to copy the pattern from

End with a summary table (category → status) so coverage is auditable.
Report `STILL_OPEN` or `NOT_YET_AUDITED` explicitly — silence is not
evidence of a clean sweep.

### 5. Save the report
Write the full report from step 4 — every audited category's status block
plus the summary table — to `skills/seo-audit/reports/<YYYY-MM-DD>.md` in
this project (create the `reports/` directory if it doesn't exist yet). If
a report already exists for today's date, append a run suffix instead of
overwriting it (`<YYYY-MM-DD>-2.md`, `-3.md`, ...). Head the file with the
run's scope (all categories, or the specific IDs requested) and the date.
This is the durable, shareable record of *this specific run's* full
output — distinct from `project-checklist.md`, which only tracks the
cumulative last-known state per category, not a given run's complete
report text. Do not commit the report automatically; that's the user's
call like any other file change.

### 6. Update `project-checklist.md`
After auditing a category — whether the result is a gap, a clean pass, or
N/A — write (or update) its entry: status, last-checked date, the recipe
that found it, and the state description. This is what makes the next run
on this project fast instead of rediscovering everything from scratch. Keep
entries even when `RESOLVED` — update status/date, never delete.

### 7. Grow the checklist (only when asked)
- A new **universal** technical-SEO practice (applies across projects, no
  project-specific paths) → append to `categories.md` following the
  existing format.
- A **project-specific finding** for an existing category → update/add its
  entry in `project-checklist.md`.
- A finding that's actually wrong code already causing bugs (not a missing
  feature) belongs in `/audit-mistakes` instead — ask before adding it here.

## Using this skill in a project
Copy `categories.md` and this file (`seo-audit.md`) into that project's
`skills/seo-audit/` unchanged, symlink `seo-audit.md` into its
`.claude/commands/`, and let the first `/seo-audit` run create that
project's own `project-checklist.md` from scratch via the discovery path in
step 2. `project-checklist.md` is always project-specific — never copy one
project's checklist into another; each project builds its own from scratch.

## Rules
- NEVER report a gap without re-running its recipe (or, for a first audit,
  the discovery approach) first — code may have already shipped a fix.
- A category requiring a manual check needs that check before reporting —
  same discipline as `/audit-mistakes`: hits are candidates, not findings.
- Do not fix anything in this run unless the user explicitly says to;
  findings feed a direct fix, `/bug-solver`, or a Redmine ticket via
  `/redmine`.
- Keep `categories.md` free of project-specific paths — if you catch
  yourself writing a hardcoded path into it, that content belongs in
  `project-checklist.md` instead.
