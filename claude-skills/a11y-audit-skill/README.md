# a11y-audit-skill

Repo-wide, read-only sweep for **missing or incomplete WCAG 2.2 AA
practices** — landmarks, skip links, heading hierarchy, alt text, form
labels, keyboard operability, focus management, color contrast, captions,
reduced-motion, and more — rather than a single reported bug.

**Manual code review only** — no automated scanner (axe-core/Pa11y/
Lighthouse) and no real assistive-technology session drives these findings
by default; see "Methodology limitations" in `SKILL.md`.

## Two files, two jobs

- **`categories.md`** — generic, framework-agnostic. What each WCAG
  category is, the Success Criterion it maps to, and how to discover the
  relevant implementation in an unfamiliar codebase. Copy this file — and
  `SKILL.md` itself — verbatim into any project.
- **`project-checklist.md`** — this project's instance: actual file paths,
  current state, and a fast re-verification recipe per category. Starts
  empty and fills in on first run. **Never copy this file between
  projects** — it's specific to the codebase that produced it.

## How to use

```
/a11y-audit                      # every category
/a11y-audit A11Y-09               # one category only
/a11y-audit add: <describe a new universal WCAG practice to track>
```

Every finding is re-verified against current code and reported as
`STILL_OPEN`, `PARTIALLY_RESOLVED`, `RESOLVED`, or `NOT_APPLICABLE`. The
audit changes no code — fixes go through a separate, explicit step.

30 categories (`A11Y-01`–`A11Y-30`) cover WCAG 2.2 Level A and AA — see
`categories.md` for full detail.

## Growing this skill

- New universal practice (applies to any project) → add to `categories.md`.
- New finding for this project on an existing category → update
  `project-checklist.md`.
- Wrong code already causing bugs (not a missing feature) → a bug-fix
  workflow, not this completeness sweep.
