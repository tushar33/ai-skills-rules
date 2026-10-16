# Project checklist — <YOUR PROJECT NAME>

This is the **per-project instance** of `categories.md` — the generic
category list has no paths; this file gets your project's actual paths,
current state, and verification recipes as `/a11y-audit` runs.

You don't need to fill this in by hand. Run `/a11y-audit` once with this
file empty (as it is now) and the skill will populate it via the discovery
workflow in `categories.md` — for each category it locates the relevant
implementation in your codebase, determines the current state, and writes
an entry here in the format below. On every later run, it re-verifies each
entry's recorded recipe first (fast path) instead of rediscovering
everything from scratch.

Status values: `STILL_OPEN`, `PARTIALLY_RESOLVED`, `RESOLVED`,
`NOT_YET_AUDITED`, `NOT_APPLICABLE`. Never delete an entry — update its
status/date instead so history survives across runs.

If your project has more than one actively-developed codebase or branch
serving the same product (e.g. a legacy version and an in-progress
redesign), tag each entry's status for both — see `a11y-audit.md`'s rules
section. If not, ignore this and just track one status per category.

---

<!--
Entries get appended here by /a11y-audit, one per category, e.g.:

## A11Y-01 — Semantic landmarks & page structure
**Status:** STILL_OPEN · **Last checked:** YYYY-MM-DD

**Recipe:**
```bash
grep -rn "<main\|<header\|<nav\|<footer" src --include=*.tsx
```
**State:** <what was actually found, with file:line references>
**Fix:** <the recommended fix>
-->
