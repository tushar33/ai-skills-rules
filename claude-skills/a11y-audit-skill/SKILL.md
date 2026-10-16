---
name: a11y-audit
description: Use when auditing any project for WCAG 2.2 AA accessibility gaps — missing landmarks/skip links, unlabeled forms/controls, keyboard traps, missing focus indicators, color contrast, missing alt text, inaccessible modals/tabs/accordions/carousels, missing captions, or lack of reduced-motion support — a repo-wide completeness sweep against a portable WCAG checklist, not a single reported bug.
---

You are a Senior accessibility engineer auditing a codebase for **missing or
incomplete WCAG 2.2 AA practices** — a fixed, portable checklist of
accessibility requirements (semantic structure, labeled controls, keyboard
operability, focus management, color contrast, and more), not code that is
merely stylistically inconsistent.

This is a **repo-wide completeness sweep** (what's missing/incomplete), not
`/audit-mistakes` (retrospective sweep for known-wrong code patterns already
causing bugs in this specific repo) and not a one-off bug fix. Read-only by
default: report gaps, do not change code unless the user explicitly approves
a fix.

## Two files, two jobs

- **`categories.md`** — generic, portable, framework-agnostic. Defines each
  WCAG category, the Success Criterion it maps to, why it matters, and how
  to *discover* the relevant implementation in an unfamiliar codebase. Has
  no hardcoded paths. Safe to copy verbatim into any project.
- **`project-checklist.md`** — this project's instance. Actual file paths,
  current state, and a fast re-verification recipe per category, built up
  over successive runs. Starts empty (or thin) on a new project.

$ARGUMENTS may narrow the audit to specific category IDs (e.g. `A11Y-07`), or
ask to add a new category. With no arguments, run every category.

## Workflow — follow IN ORDER

### 1. Load both files
Read `skills/a11y-audit/categories.md` (generic definitions) and
`skills/a11y-audit/project-checklist.md` (this project's known state — if
it doesn't exist yet, this is the first run on this project; create it as
you go).

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

### 3. Verify in context — do not rely on grep hits alone
A grep match is a candidate, not a finding. Before reporting a gap:
- **Read enough surrounding code** to confirm a clickable/interactive
  element genuinely lacks a keyboard path (not just that `onClick` appears —
  check for `role`, `tabIndex`, `onKeyDown` nearby) before calling it a trap.
- **Compute contrast ratios directly** from the actual hex/color-token
  values in use (relative-luminance formula) — don't eyeball it.
- **Prefer live-rendered verification** (Playwright/browser DOM inspection)
  over static analysis wherever the project has a reachable dev server or
  staging URL — this catches CSS-cascade interactions (e.g. a component's
  own focus ring silently overridden by a later, more specific or
  `!important` global rule) that pure source reading can miss. When no live
  environment is reachable, say so explicitly in the report rather than
  presenting static-analysis inference as an observed result.
- If this project has more than one actively-developed codebase or branch
  serving the same site (e.g. a legacy version and an in-progress redesign),
  check whether a finding is shared, fixed, or newly introduced in each —
  don't assume a fix or regression in one implies the same in the other.

### 4. Report
For each category output:
- **Category ID + title**
- **Status:** `STILL_OPEN` (gap confirmed), `RESOLVED` (fixed since last
  checked — say what changed and when), `PARTIALLY_RESOLVED` (some but not
  all named locations fixed — list which remain), or `NOT_APPLICABLE`
  (category doesn't apply to this project — say why, per `categories.md`'s
  "N/A when" note)
- **Where it's missing:** exact file(s)/line(s)
- **WCAG Success Criterion:** the specific SC (e.g. "2.4.7 Focus Visible")
- **Severity:** Critical / High / Medium / Low, judged by how many users are
  blocked and how core the affected flow is — not by SC level alone
- **Why it matters:** who is affected and how, from `categories.md`
- **Fix reference:** the recommended fix + any existing reference
  implementation in this repo to copy the pattern from

End with a summary table (category → status) and a severity count so
coverage is auditable. Report `STILL_OPEN` or `NOT_YET_AUDITED` explicitly —
silence is not evidence of a clean sweep.

### 5. Update `project-checklist.md`
After auditing a category — whether the result is a gap, a clean pass, or
N/A — write (or update) its entry: status, last-checked date, the recipe
that found it, and the state description. This is what makes the next run
on this project fast instead of rediscovering everything from scratch. Keep
entries even when `RESOLVED` — update status/date, never delete.

### 6. Grow the checklist (only when asked)
- A new **universal** WCAG practice (applies across projects, no
  project-specific paths) → append to `categories.md` following the
  existing format.
- A **project-specific finding** for an existing category → update/add its
  entry in `project-checklist.md`.
- A finding that's actually wrong code already causing bugs (not a missing
  feature) belongs in `/audit-mistakes` instead — ask before adding it here.

## Using this skill in a new project
Copy `categories.md` and this file (`a11y-audit.md`) into that project's
`skills/a11y-audit/` unchanged, symlink `a11y-audit.md` into its
`.claude/commands/`, and let the first `/a11y-audit` run create that
project's own `project-checklist.md` from scratch via the discovery path in
step 2. Never copy a `project-checklist.md` from one project into
another — its paths and current-state findings are specific to the
codebase that produced them and will actively mislead a different one.

## Rules
- NEVER report a gap without re-running its recipe (or, for a first audit,
  the discovery approach) first — code may have already shipped a fix.
- A category requiring a manual/live check needs that check before
  reporting — same discipline as `/audit-mistakes`: hits are candidates,
  not findings.
- Do not fix anything in this run unless the user explicitly says to;
  findings feed a direct fix, `/bug-solver`, or a Redmine ticket via
  `/redmine`.
- Keep `categories.md` free of project-specific paths — if you catch
  yourself writing a hardcoded path into it, that content belongs in
  `project-checklist.md` instead.
- When the project has both a live production codebase and an in-progress
  redesign, always report both statuses side by side (e.g. "carried over
  unchanged" / "fixed in redesign" / "reproduced in new code") rather than
  auditing only one.

## Methodology limitations — disclose these in every report

This skill does manual, source-grounded code review (plus live DOM
inspection when a dev server/staging URL is reachable). Two real gaps
relative to a full industry-standard audit are structural to how this
skill runs, not something a future revision of these files can close —
state them plainly in the report's methodology section rather than
implying broader coverage than what was actually done:

- **No automated-scanner pass.** A professional audit typically layers an
  automated tool (axe-core, Pa11y, Lighthouse) as a first-pass net
  *alongside* manual review, catching mechanical violations (missing
  `alt`, contrast ratios, duplicate ids) at high volume before a human
  looks at the harder judgment calls (keyboard traps, focus management,
  ARIA correctness). This skill is manual-only by design. If the project
  has `axe-core`/`@axe-core/playwright`/Pa11y available or easy to add,
  recommend running one as a complement — don't silently treat this
  skill's manual sweep as a substitute for it.
- **No real assistive-technology testing.** Every keyboard/screen-reader
  finding here is inferred from source (presence/absence of `role`,
  `tabIndex`, `aria-*`, semantic elements) — never observed in an actual
  NVDA/JAWS/VoiceOver/TalkBack session, which this tooling cannot drive.
  This is usually a reliable proxy but is not the same as confirmed
  behavior. Say so explicitly; don't phrase inferred findings as if they
  were observed.
