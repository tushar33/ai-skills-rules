# a11y-audit

**Claude command:** `/a11y-audit`
**Skill definition:** `.claude/commands/a11y-audit.md` → `a11y-audit.md`
**Generic categories (portable):** `skills/a11y-audit/categories.md`
**Your project's instance:** `skills/a11y-audit/project-checklist.md`

Repo-wide, read-only sweep for **missing or incomplete WCAG 2.2 AA
practices** — semantic landmarks, skip links, heading hierarchy, page
language, image/icon alt text, complex-visual text alternatives, form
labels/validation, keyboard operability of custom controls, focus
visibility/management, color contrast, tables, multimedia captions/audio
control/audio-description, motion/reduced-motion, target size, timing/
session limits, error prevention on consequential actions, reflow/text-
spacing at high zoom, consistent navigation/terminology, multiple ways to
locate content, link quality, viewport zoom/orientation, status messages/
live regions, and the newer WCAG 2.2 criteria (redundant entry, accessible
authentication, dragging movements, focus not obscured) — rather than a
single reported bug.

**Manual code review only** — no automated scanner (axe-core/Pa11y/
Lighthouse) and no real assistive-technology session drives these
findings by default; see "Methodology limitations" in `a11y-audit.md` for
what that means for confidence level, and consider layering an automated
scanner alongside it if your project already has one available.

## Two files, two jobs

This skill is split so it's actually reusable across projects:

- **`categories.md`** — generic, framework-agnostic. What each WCAG
  category is, the Success Criterion it maps to, why it matters, and *how
  to discover* the relevant implementation in an unfamiliar codebase (no
  hardcoded paths). Copy this file — and `a11y-audit.md` itself — verbatim
  into any project's `skills/a11y-audit/`.
- **`project-checklist.md`** — your project's instance. Actual file
  paths, current state, and a fast re-verification recipe per category,
  built up as `/a11y-audit` runs. Starts empty (see the stub shipped in
  this repo) and fills itself in on first run via the discovery workflow
  in `categories.md`. **Never copy this file between projects** — its
  content is specific to the codebase that produced it.

## How it differs from a linter or a one-off bug fix

| Approach | Direction | Scope | Subject |
|---|---|---|---|
| ESLint a11y plugin | continuous, mechanical | per-file, per-commit | syntactic rule violations |
| One-off bug report | reactive | one reported issue | any bug |
| **`/a11y-audit`** | **completeness sweep** | **whole repo, portable across projects** | **missing/incomplete WCAG 2.2 AA practices** |

## How to use

```
/a11y-audit                     # every category, using project-checklist.md's fast path + discovery for gaps
/a11y-audit A11Y-09              # one category only (e.g. custom interactive controls)
/a11y-audit add: <describe a new universal WCAG practice to track>
```

Every finding is re-verified against current code (a checklist entry
records a last-known state, but code moves) and reported as `STILL_OPEN`,
`PARTIALLY_RESOLVED`, `RESOLVED`, or `NOT_APPLICABLE`. The audit changes no
code — fixes go through explicit approval or direct implementation
afterward, in a separate step.

## Categories (see `categories.md` for full detail)

30 categories, `A11Y-01` through `A11Y-30`, covering WCAG 2.2 Level A and
AA — from foundational structure (landmarks, headings, labels) through the
newer WCAG 2.2 additions (redundant entry, accessible authentication,
dragging movements, focus not obscured, target size). `A11Y-30` (browser/
assistive-technology compatibility) is always reported `NOT_APPLICABLE` to
this skill's own methodology, by design — see `categories.md` for why.

Run `/a11y-audit` with no arguments on a fresh project to have it discover
and populate every category from scratch.

## Growing this skill

- New **universal** practice (applies to any project) → add to
  `categories.md`.
- New **finding for your project** on an existing category → update
  `project-checklist.md`.
- Wrong code already causing bugs (not a missing feature) → belongs in a
  bug-fix workflow instead, not this completeness sweep.
