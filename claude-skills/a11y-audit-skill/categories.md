# WCAG 2.2 AA categories — generic, portable

Framework-agnostic. This file has **no hardcoded paths** — it defines what
to check and why, plus discovery hints for finding the relevant
implementation in an unfamiliar codebase. Copy this file (and
`a11y-audit.md`) into any project's `skills/a11y-audit/` unchanged. The
project-specific instance — actual file paths, current state, verification
recipes for *this* codebase — lives in a separate `project-checklist.md`
that starts empty (or nearly empty) in a new project and gets built up as
the skill runs.

Each category: what it is → WCAG SC → why it matters → generic detection
approach → discovery hints (where this typically lives, across a few common
stacks) → when it doesn't apply.

---

## A11Y-01 — Semantic landmarks & page structure
**What:** `<main>`, `<header>`/`role="banner"`, `<nav>`/`role="navigation"`,
`<footer>`/`role="contentinfo"` — the structural regions a screen reader
user navigates by, instead of a generic `<div>` soup.
**WCAG SC:** 1.3.1 Info and Relationships (A) · 2.4.1 Bypass Blocks (A, see
A11Y-02).
**Why:** Screen readers expose a landmarks list as a primary orientation
tool. Without real landmark elements, a user has no way to jump directly to
main content, navigation, or footer.
**Generic detection:** Grep for `<main`, `<header`, `<nav`, `<footer`,
`role="main"|"navigation"|"banner"|"contentinfo"`, and any framework-specific
"render as a different element" escape hatch (e.g. Chakra's `as="nav"`,
styled-components' `as`, plain `<div>` vs semantic tag choice). Zero or near-
zero hits across a component-heavy app is a strong signal of a gap.
**Discovery hints:** Find the root layout/shell component every page renders
through (often named `Layout`, `AppShell`, `_app`, `RootLayout`) and the
header/nav/footer components it composes.
**N/A when:** never — every page-rendering app needs this.

---

## A11Y-02 — Skip navigation / bypass blocks
**What:** A visually-hidden-until-focused "Skip to main content" link as the
first focusable element on the page, or an equivalent bypass mechanism.
**WCAG SC:** 2.4.1 Bypass Blocks (A).
**Why:** Without it, a keyboard user re-tabs through the entire header/nav
on every single page load before reaching that page's actual content.
**Generic detection:** Case-insensitive search for "skip to content",
"skip-link", "skip navigation" across source and the root HTML template.
**Discovery hints:** Root app component, or the root HTML template
(`index.html`, `_document`, base layout).
**N/A when:** the page has no repeated header/nav chrome above the content
(rare for a real app).

---

## A11Y-03 — Heading hierarchy
**What:** Exactly one real `<h1>` per page (visible or legitimately hidden —
off-screen positioning, not `display:none`), heading levels that descend
without skipping (h1→h2→h3, never h1→h3), and no non-heading content
(nav items, buttons, labels) styled via a heading component purely for its
visual weight.
**WCAG SC:** 1.3.1 Info and Relationships (A) · 2.4.6 Headings and Labels
(AA).
**Why:** Screen reader users jump between headings as a primary content
outline. A missing/duplicated h1, a level skip, or a heading-styled button
breaks that outline or creates false landmarks.
**Generic detection:** Find the shared heading/typography components (often
`H1`..`H5` wrappers or a `Heading` component with an `as`/`level` prop) and
grep for their usage on non-title content — buttons, nav items, form field
labels, list item captions dressed up in a heading purely for font-weight.
**Discovery hints:** A design-system `Typography/` or `Theme/` folder
usually holds the heading-level wrapper components; grep their import list
sitewide for repeated non-title usage.
**N/A when:** never.

---

## A11Y-04 — Page language & title
**What:** `<html lang="...">` set correctly and kept in sync with any
in-app language switcher; a dynamic, per-page, non-empty `<title>`.
**WCAG SC:** 3.1.1 Language of Page (A) · 2.4.2 Page Titled (A).
**Why:** Screen readers select pronunciation/voice rules from the declared
page language — a stale `lang` attribute after a language switch makes
content read with the wrong phonetic rules. A missing/generic title leaves
screen reader and browser-tab users with no way to identify the page.
**Generic detection:** Check whether `document.documentElement.lang` is
ever set dynamically (grep `documentElement.lang`, `htmlAttributes`,
framework head-management API's `lang` prop) versus hardcoded once in the
root HTML template. Separately confirm the shared head/meta component sets
a real per-page `<title>`.
**Discovery hints:** The same head-management component/hook responsible
for meta tags (react-helmet, `next/head`, framework-native `<Head>`) is
usually also the right place for `lang` sync; check if it's already wired
there or missing entirely. Cross-check against any i18n library's
`changeLanguage`/locale-switch call site.
**N/A when:** the app is genuinely single-language with no switcher (then
only the static `lang` attribute correctness matters, not the sync check).

---

## A11Y-05 — Non-text content: images, icons, SVG
**What:** Every `<img>`/`<Image>` has a meaningful `alt` for content images
or an explicit `alt=""` for decorative ones (never omitted). Decorative
inline SVGs carry `aria-hidden="true"`; meaningful ones carry a `<title>` or
`aria-label`.
**WCAG SC:** 1.1.1 Non-text Content (A).
**Why:** A screen reader either announces nothing (if `alt` is omitted and
falls back to the filename/src) or announces a meaningless decorative
image, when the correct behavior is either a real description or silence.
**Generic detection:** Find the shared image wrapper component and check
whether it enforces (or at least dev-time-warns on) a missing `alt` prop.
Sample real call sites across the app for `alt=""` misapplied to real
content photos, or `alt` simply omitted on decorative dividers/icons.
**Discovery hints:** Shared `Image`/`Img` wrapper component, usually in a
common/shared components folder.
**N/A when:** never — any app rendering images needs this.

---

## A11Y-06 — Complex visuals: charts, infographics, data visualizations
**What:** A text-equivalent (summary sentence, `aria-label`, or a visually-
hidden data table) for any canvas/SVG-rendered chart or infographic.
**WCAG SC:** 1.1.1 Non-text Content (A) · 1.3.1 Info and Relationships (A).
**Why:** Chart libraries (ECharts, D3, Chart.js, Recharts) render to
`<canvas>` or a `<svg>` with no inherent text content — a screen reader user
gets nothing at all from the visualization by default.
**Generic detection:** Find the chart-library wrapper component(s) and check
whether any accessible-name/table-alternative mechanism exists, given that
the underlying structured data is usually already available in the
component's props/state.
**Discovery hints:** Grep for the chart library's import name across the
codebase to find every chart component; check each for `aria-label`,
`role="img"`, or a sibling visually-hidden table.
**N/A when:** the app has no data visualizations.

---

## A11Y-07 — Forms: labels, required fields, input purpose
**What:** Every form input has a programmatically-associated label (real
`<label for>`/`htmlFor`, not placeholder-only or a visually-adjacent-but-
unlinked text node); required fields are marked with `required`/
`aria-required` (not a bare visual asterisk alone); common input types
declare `autocomplete` per WHATWG's input-purpose token list.
**WCAG SC:** 1.3.1 Info and Relationships (A) · 1.3.5 Identify Input
Purpose (AA) · 3.3.2 Labels or Instructions (A).
**Why:** Placeholder-only labeling disappears once the user types and isn't
reliably exposed as an accessible name across AT. Missing `autocomplete`
breaks password-manager/browser-autofill support, disproportionately
affecting users with motor or cognitive impairments.
**Discovery hints:** Any form-heavy page (auth, registration, checkout,
contact, search) — check the form-library/UI-kit's label component for
whether it auto-wires `htmlFor`/`id`, and whether real call sites actually
use it versus relying on `placeholder`.
**Generic detection:** Grep form files for `placeholder=` co-occurring with
zero `label`/`FormLabel`/`htmlFor` in the same field block; grep for
`autoComplete`/`autocomplete` presence rate across auth/registration forms.
**N/A when:** the app has no forms.

---

## A11Y-08 — Forms: error identification & validation announcement
**What:** Validation errors are associated with their field via
`aria-describedby`, and/or announced via `role="alert"`/a live region — not
rendered as a plain, unassociated text node next to an `isInvalid`-flagged
field. Ideally also a top-of-form error summary on submit failure.
**WCAG SC:** 3.3.1 Error Identification (A) · 4.1.3 Status Messages (AA).
**Why:** A screen reader user tabbing into an invalid field must be told
*why* it's invalid, at the moment of focus — not left to discover the error
text visually.
**Generic detection:** For every form found in A11Y-07, check whether its
validation-error rendering path uses the UI kit's accessible error
component (e.g. Chakra's `FormErrorMessage`, MUI's `helperText` +
`error`) versus a bare styled text element with no ARIA wiring.
**Discovery hints:** Same files as A11Y-07; look specifically at what
renders when `errors[fieldName]` is truthy.
**N/A when:** the app has no forms with client-side validation.

---

## A11Y-09 — Custom interactive controls: name, role, keyboard operability
**What:** Any element that isn't a native `<button>`/`<a>`/`<input>` but
behaves like one (a `<div>`/`<span>`/`<Image>` with `onClick`) must carry
`role="button"` (or equivalent), `tabIndex={0}`, an `onKeyDown` handling
Enter/Space, and an accessible name (`aria-label` if no visible text).
Icon-only buttons need `aria-label`. `aria-expanded`/`aria-controls`/
`aria-selected` must be present where a widget's state changes, and any
`aria-*` id-reference (`aria-labelledby`, `aria-describedby`,
`aria-controls`) must resolve to a real id in the DOM.
**WCAG SC:** 2.1.1 Keyboard (A) · 4.1.2 Name, Role, Value (A).
**Why:** This is the single most common accessibility defect class in
component-based UIs — a visual "button" built from a styling primitive
instead of a real interactive element, invisible to keyboard and screen-
reader users alike.
**Generic detection:** Grep for `onClick` on non-native-interactive
elements (styled `div`/`span`/`Box`/`Image`/icon components) and check
within ~10 lines for `role`, `tabIndex`, `onKeyDown`. A hit with none of
those is a strong candidate — confirm by reading the actual render output.
Separately, grep icon-only button usages (`IconButton` or equivalent) for
missing `aria-label`.
**Discovery hints:** This pattern concentrates in: modal/drawer close
buttons, carousel/slider prev-next controls, zoom/lightbox triggers,
password-visibility toggles, filter/menu toggles, tab-like navigation built
from styled divs instead of a real tabs primitive.

For **tabs and menus specifically**, check against the ARIA Authoring
Practices pattern rather than just "has a role": a tablist should use
roving `tabindex` (only the active tab is `tabIndex={0}`, the rest
`tabIndex={-1}`, with Left/Right arrow keys moving both focus and
selection), `aria-selected` on the active tab, and each `tabpanel`
`aria-labelledby` its controlling tab. A menu (as opposed to plain
navigation links) should support Escape-to-close and Up/Down arrow
navigation between items. If the UI framework's own Tabs/Menu primitive
is used correctly (e.g. Chakra's `Tabs`/`Menu`), this is usually free —
the finding to look for is a *reimplementation* with plain styled
`div`s/`onClick` that drops these behaviors silently. Conversely, watch for
the inverse mistake: `role="tab"`/`aria-selected` applied to what is
actually plain page navigation (a nav link styled to look like a tab) —
activating a real tab should only change the associated panel's
visibility, never navigate to a new URL/page.
**N/A when:** never — any interactive UI needs this checked.

---

## A11Y-10 — Focus visibility & focus management (modals, dialogs, drawers)
**What:** A visible focus indicator on every focusable element sitewide (no
blanket `outline: none`/`box-shadow: none` without a replacement); modals/
dialogs trap focus while open, close on Escape (unless there's a specific,
justified reason not to, with an alternative dismiss path always present),
and restore focus to the triggering element on close.
**WCAG SC:** 2.4.7 Focus Visible (AA) · 2.4.11 Focus Appearance (AA, WCAG
2.2) · 2.4.3 Focus Order (A) · 2.1.2 No Keyboard Trap (A).
**Why:** Sighted keyboard users rely entirely on the focus indicator to
know where they are. A modal that disables Escape with no alternative
close path, or that doesn't restore focus, disorients or traps keyboard
users.
**Generic detection:** Grep the global stylesheet(s) and the design-system
theme file for `outline: none`/`outline:none !important`/`:focus{...}`
rules with no replacement indicator — a universal selector (`*:focus`,
`*:focus-visible`) doing this is the most severe variant since it overrides
every component-level fix. Separately, grep the modal/dialog primitive's
usages for `closeOnEsc={false}`/equivalent and `returnFocusOnClose={false}`/
equivalent, and check whether each has a compensating reason or a visible
close control.
**Discovery hints:** Design-system theme file (often `Theme.js`/
`theme.ts`), global CSS entry point(s), and the shared Modal/Dialog/Drawer
wrapper component.
**N/A when:** never.

---

## A11Y-11 — Color contrast
**What:** Normal text ≥ 4.5:1 against its background; large text (≥24px, or
≥19px bold) and UI-component boundaries (button borders, form-input
borders, focus indicators, icons conveying meaning) ≥ 3:1.
**WCAG SC:** 1.4.3 Contrast (Minimum) (AA) · 1.4.11 Non-text Contrast (AA).
**Why:** Insufficient contrast excludes low-vision users and anyone in
poor lighting/glare conditions — one of the most common and most
mechanically-checkable accessibility defects.
**Generic detection:** Extract the color palette from the design-system
theme file/tokens, compute contrast ratios directly via the WCAG relative-
luminance formula for every foreground/background pair that plausibly
pairs together (especially any "muted"/"subtle"/"disabled" gray tokens
against a white/light background), then grep real usage to confirm which
low-contrast tokens are applied to actual readable text/UI vs. purely
decorative elements (e.g. a divider line, which is exempt).
**Discovery hints:** Theme/tokens file; grep component files for the
specific low-contrast token names found there.
**N/A when:** never.

---

## A11Y-12 — Use of color alone
**What:** Any status/state indicator (live/upcoming/past, success/error,
selected/unselected) must be distinguishable by more than color alone —
add text, an icon, a pattern, or an underline.
**WCAG SC:** 1.4.1 Use of Color (A).
**Why:** Color-blind users and grayscale/high-contrast-mode users can't
distinguish state by hue alone.
**Generic detection:** Find status-badge/indicator components and confirm
each state pairs its color with a text label or distinct icon, not color
alone.
**Discovery hints:** Status badge, tag, or pill components; form
valid/invalid states (border color alone vs. border + icon + text).
**N/A when:** the app has no color-coded status indicators.

---

## A11Y-13 — Tables
**What:** Real `<table>` markup (not a styled grid of divs faking tabular
data) has `<th scope="col">`/`<th scope="row">` on every header cell and a
`<caption>` or `aria-labelledby` identifying the table's purpose.
**WCAG SC:** 1.3.1 Info and Relationships (A).
**Why:** Screen readers use `scope` to announce which header applies to a
given cell when navigating table data cell-by-cell.
**Generic detection:** Grep for `<table`/the UI kit's `Table` component;
for each real result, check for `scope=` presence rate and a caption/label.
Also check for the *inverse* anti-pattern: a div-based grid using
`role="table"`/`role="row"`/`role="cell"` without proper `scope` equivalents
(`aria-colindex` etc.), which is usually worse than just using a real
`<table>`.
**N/A when:** the app renders no genuinely tabular data (card grids,
timelines, and lists that merely look table-like in a naming convention
are not tables).

---

## A11Y-14 — Multimedia: captions, transcripts, audio control
**What:** Video has caption/subtitle tracks (`<track kind="captions">` or
the embed platform's caption support); audio/video that autoplays for more
than 3 seconds is muted by default or has an immediately-available pause/
stop control; long-form audio has a transcript.
**WCAG SC:** 1.2.1/1.2.2 Captions (A) · 1.4.2 Audio Control (A).
**Why:** Deaf/hard-of-hearing users need captions for any spoken content;
unmuted autoplay audio with no easy stop is disorienting for screen-reader
users (whose AT voice competes with it) and users with cognitive
impairments.
**Generic detection:** Grep for every video/audio embed pattern in use
(native `<video>`, YouTube/Vimeo iframe embeds, a player library) and check
each for caption-track support and autoplay+mute/control pairing.
**Discovery hints:** Video-embed wrapper component(s); search for
`autoplay`/`autoPlay` parameters passed to any player.
**N/A when:** the app has no audio/video content.

---

## A11Y-15 — Motion & animation
**What:** Respect for `prefers-reduced-motion` (CSS media query or JS
`matchMedia` check) gating non-essential animation/parallax/autoplay; any
auto-advancing carousel/slideshow provides a pause/stop/hide control
(WCAG 2.2.2); no content flashes more than 3 times per second.
**WCAG SC:** 2.3.1 Three Flashes or Below (A) · 2.2.2 Pause, Stop, Hide
(A) · 2.3.3 Animation from Interactions (AAA, but good practice for AA
projects too).
**Why:** Users with vestibular disorders can experience real physical
symptoms (dizziness, nausea) from motion they've explicitly opted out of at
the OS level; an unstoppable auto-advancing carousel is a comprehension and
control barrier for cognitive-impairment and low-vision users alike.
**Generic detection:** Grep for `prefers-reduced-motion` sitewide (its
absence entirely is itself the finding). Separately, find every carousel/
slider component and check for an autoplay config plus whether pause/stop
controls are actually present and reachable (not just configured but
disabled elsewhere).
**Discovery hints:** Carousel/slider library usage (Swiper, Slick,
embla, etc.) — check its `autoplay` config and accessibility-module usage
if the library ships one.
**N/A when:** the app has no autoplaying or decorative animated content.

---

## A11Y-16 — Target size (WCAG 2.2)
**What:** Interactive targets (buttons, links, carousel pagination dots,
close icons) are at least 24×24 CSS pixels, or have an invisible padded hit-
area of that size even if the visible glyph is smaller.
**WCAG SC:** 2.5.8 Target Size Minimum (AA, WCAG 2.2).
**Why:** Small targets are hard to hit precisely for users with motor
impairments or tremor, especially on touch devices.
**Generic detection:** Check carousel pagination dots, small icon-only
close/toggle buttons, and inline action icons for their actual rendered
box size (including padding) versus the 24×24px floor.
**Discovery hints:** Same carousel/pagination components as A11Y-15; any
small icon-button component in the shared UI kit.
**N/A when:** never, though violations here are usually Low/Medium
severity rather than blocking.

---

## A11Y-17 — Timing adjustable & session timeouts
**What:** Any time limit the app imposes (a session timeout, an OTP/
verification-code expiry, an auto-advancing step) can be turned off,
adjusted, or extended by the user — typically via a warning dialog offering
"Extend session" before logout, given well before expiry.
**WCAG SC:** 2.2.1 Timing Adjustable (A).
**Why:** Users with cognitive or motor impairments, or anyone reading
carefully/using AT (which takes longer per screen), may be logged out or
lose in-progress work before they can finish, with no way to buy more time.
**Generic detection:** Find the auth/session layer's token-refresh or
idle-timeout logic and check whether a warning + extend affordance exists
before forced logout, versus a silent timeout.
**Discovery hints:** Redux/session middleware, an axios/fetch response
interceptor handling 401s, or a `setTimeout`/`setInterval` idle-tracker tied
to logout.
**N/A when:** the app has no session expiry or other user-facing time
limit at all (e.g. a fully static, unauthenticated site).

---

## A11Y-18 — Error prevention for legal, financial, or data-loss actions
**What:** Before an irreversible or consequential action (payment,
account deletion, publishing, submitting a legal/financial form), the user
gets a chance to review, confirm, correct, or undo — not an instant,
one-click, no-confirmation commit.
**WCAG SC:** 3.3.4 Error Prevention (Legal, Financial, Data) (AA).
**Why:** A single mis-click shouldn't be able to charge a card, delete an
account, or submit unrecoverable data — this protects everyone, but
especially users with motor or cognitive impairments prone to accidental
activation.
**Generic detection:** Find every "point of no return" action (delete,
pay, publish, submit-final-registration) and check for a confirmation
step, a review screen, or a post-submit undo window.
**Discovery hints:** Checkout/payment flow, account-settings destructive
actions, any final "Submit" on a multi-step form.
**N/A when:** the app has no irreversible or consequential user actions.

---

## A11Y-19 — Reflow & text spacing at high zoom
**What:** Content reflows to a single column with no loss of information
or functionality at 400% zoom / 320px-equivalent width (no two-dimensional
scrolling required except for things that inherently need it — maps,
data tables, images); the layout survives a user's browser-level override
of line-height, paragraph spacing, letter-spacing, and word-spacing.
**WCAG SC:** 1.4.10 Reflow (AA) · 1.4.12 Text Spacing (AA).
**Why:** Low-vision users commonly zoom to 200-400% or override text
spacing via browser/OS settings; fixed-width layouts or hardcoded
line-height that fights an override make content unreadable or clip it.
**Generic detection:** Check for fixed-pixel-width containers (not
responsive/percentage/max-width-based) on core content columns; check
whether any CSS uses `!important` on `line-height`/`letter-spacing` in a
way that would resist a user override.
**Discovery hints:** Global layout/container components; any component
with a hardcoded `width` (not `maxWidth`) in px on a text-bearing element.
**N/A when:** never for a responsive web app, though severity is often
lower than keyboard/label defects.

---

## A11Y-20 — Consistent navigation, identification & help
**What:** The same repeated component (nav bar, icon, button) appears in
the same relative order and looks/behaves the same across every page it's
present on; the same function is always labeled with the same term (never
"Delete" on one screen and "Remove" for the identical action on another);
where a help/contact mechanism exists, it's reachable the same way from
every page it's provided on.
**WCAG SC:** 3.2.3 Consistent Navigation (AA) · 3.2.4 Consistent
Identification (AA) · 3.2.6 Consistent Help (A, WCAG 2.2).
**Why:** Inconsistent navigation or terminology forces users — especially
those with cognitive impairments or using AT, who build a mental model
more slowly — to re-learn the interface on every page.
**Generic detection:** Compare the header/nav/footer component tree across
a sample of different page types for structural drift; grep for repeated
actions/icons (e.g. "delete", "remove", "trash") to check for inconsistent
terminology; check whether a help/support/contact entry point (if one
exists) appears at a consistent location across pages that offer it.
**Discovery hints:** Shared layout/shell component (should make
structural drift unlikely by construction — flag if pages bypass it);
i18n/constants files, where inconsistent synonym labels are easiest to spot.
**N/A when:** the app is a single page with no repeated chrome.

---

## A11Y-21 — Redundant entry & accessible authentication (WCAG 2.2)
**What:** Information the user already provided earlier in the same
process (or that the system already has) is auto-populated or offered as
a selectable option rather than requested again from scratch; the login/
auth flow doesn't rely solely on a cognitive function test (remembering a
password, solving a puzzle) with no alternative — a password-manager-
compatible field, "paste" not blocked, or a non-cognitive alternative
(e.g. email magic link, WebAuthn) is available.
**WCAG SC:** 3.3.7 Redundant Entry (A, WCAG 2.2) · 3.3.8 Accessible
Authentication (Minimum) (AA, WCAG 2.2).
**Why:** Re-entering the same information across steps of one process is
a needless memory/motor burden; a login that requires solving a puzzle or
transcribing/remembering a password with no alternative and no autofill
support excludes users with cognitive and memory impairments.
**Generic detection:** For any multi-step form/wizard, check whether a
value collected in step N is required again verbatim in step N+1+ (versus
pre-filled/confirmed). For login, check for CAPTCHA type (image-transcription
puzzles fail this SC; checkbox/invisible reCAPTCHA generally doesn't),
whether pasting into password fields is blocked (`onPaste` prevented — a
real, common anti-pattern that breaks password managers), and whether
`autocomplete` is present (ties back to A11Y-07).
**Discovery hints:** Multi-step registration/checkout wizards; the login
form's paste/`onPaste` handling and CAPTCHA implementation, if any.
**N/A when:** the app has no multi-step forms and no authentication.

---

## A11Y-22 — Dragging movements & pointer gesture alternatives (WCAG 2.2)
**What:** Anything operable via a drag gesture (reordering a list, a
slider/range control, swipe-to-dismiss) also has a single-pointer
alternative (tap/click buttons, keyboard arrows) that achieves the same
result; multi-point or path-based gestures (pinch, multi-finger swipe)
have a single-pointer equivalent.
**WCAG SC:** 2.5.1 Pointer Gestures (A) · 2.5.2 Pointer Cancellation (A) ·
2.5.7 Dragging Movements (AA, WCAG 2.2).
**Why:** Users with motor impairments or tremor, or anyone using a switch/
head-pointer device, often cannot perform a precise drag or multi-touch
gesture at all.
**Generic detection:** Find any drag-and-drop library usage (`react-dnd`,
`dnd-kit`, native HTML5 drag events, a custom pointer-move-based slider)
and check for an accompanying non-drag control (up/down reorder buttons,
a numeric input alongside a range slider, arrow-key support on a slider).
**Discovery hints:** Reordering UIs (playlist, kanban, image-gallery
reorder), custom range/slider components, swipe-to-dismiss card UIs.
**N/A when:** the app has no drag-based or multi-touch-gesture interactions
(click-based carousels with a swipe *convenience* but full click/keyboard
parity, like most of this repo's carousels, are compliant, not exempt —
recheck against A11Y-09/15 rather than marking N/A).

---

## A11Y-23 — Focus not obscured (WCAG 2.2)
**What:** When an element receives keyboard focus, at least part of it
(ideally all of it) remains visible — not fully hidden behind a sticky
header, cookie banner, or other fixed-position overlay.
**WCAG SC:** 2.4.11 Focus Not Obscured (Minimum) (AA, WCAG 2.2). (2.4.12
Focus Not Obscured (Enhanced) and 2.4.13 Focus Appearance are AAA — out of
scope for an AA-only audit; note them only if the project targets AAA.)
**Why:** A sticky header/footer/cookie-banner that overlaps the top or
bottom of the viewport can completely cover a focused element as the user
tabs through the page, making it impossible to see what currently has
focus even though a focus indicator exists.
**Generic detection:** Identify every fixed/sticky-positioned element
(headers, footers, cookie-consent banners, "back to top" buttons) and
check whether tabbing to content near the top/bottom of the viewport would
land underneath one of them — this usually needs a live/rendered check
(scroll-margin or scroll-padding-top compensation is the common fix), not
just source reading.
**Discovery hints:** Any `position: sticky`/`position: fixed` CSS on a
header, footer, or banner component; check for `scroll-margin-top`/
`scroll-padding-top` compensation already in place.
**N/A when:** the app has no sticky/fixed-position overlays.

---

## A11Y-24 — Downloadable document accessibility
**What:** Linked PDFs, Word/Office documents, or other downloadable files
are themselves accessible (tagged PDF with a real structure/reading order,
not a flat scanned image) — or, at minimum, the link text indicates the
file type and size before download, and an HTML equivalent is offered
where feasible.
**WCAG SC:** 1.1.1 Non-text Content (A) · 1.3.1 Info and Relationships (A)
· 3.2.4 Consistent Identification (AA) — as applied to link text.
**Why:** An untagged/scanned PDF is often completely opaque to a screen
reader (just an image, or reading order scrambled); unlabeled file
type/size surprises users on slow connections or metered data.
**Generic detection:** Grep for `.pdf`/`.docx`/download-link patterns; for
each, check whether the link text mentions the file type/size, and (if the
files are produced by this app, not just linked) whether the generation
pipeline produces tagged/accessible output.
**Discovery hints:** Any "download brochure/report/certificate" feature;
document-generation libraries (`pdf-lib`, server-side PDF rendering) if the
app generates PDFs itself rather than only linking to pre-made ones.
**N/A when:** the app has no downloadable documents.

---

## A11Y-25 — Links: descriptive text, distinguishable, no duplicates
**What:** Link text describes its destination/action out of context (never
a bare "click here"/"read more"/"link" with no surrounding context that
would clarify it); links embedded in body text are visually distinguishable
from surrounding text by more than color alone (underline, weight — not
hue alone); two adjacent links pointing to the same destination (e.g. an
image link immediately followed by a text link to the same target) are
merged into one link rather than left as two separate stops for a
keyboard/screen-reader user; links that open in a new tab/window or leave
the site are indicated before activation.
**WCAG SC:** 2.4.4 Link Purpose (In Context) (A) · 1.4.1 Use of Color (A)
· 3.2.5 Change on Request (AAA, note only if opening new windows without
warning — most AA audits treat this as a strong recommendation, not a hard
AA failure).
**Why:** "Click here" repeated across a page is meaningless out of context
for a screen reader user who navigates a page's links as a flat list (many
AT tools offer a "links list" view specifically). Color-only link styling
is invisible to colorblind users and anyone in grayscale/high-contrast
mode. Duplicate adjacent links double every card/list-item's tab stops
sitewide, a small but compounding tax on keyboard navigation.
**Generic detection:** Grep rendered/JSX link text for generic phrases
("click here", "read more", "here", "link", "learn more" with no adjacent
context naming the subject). Check the design system's link/anchor
component for whether its default style relies on color alone versus
underline/weight. Find any card/list-item pattern that wraps both an image
and its caption/title in separate links to the same URL.
**Discovery hints:** Shared `Link`/`CLink`/anchor-wrapper component; any
card component rendering an image thumbnail + title as two elements.
**N/A when:** the app renders no inline/body-text links (rare).

---

## A11Y-26 — Multiple ways to locate content
**What:** At least two independent paths exist to reach the same page —
e.g. navigation menu + site search, or a sitemap + breadcrumb trail — not
a single, sole path with no alternative.
**WCAG SC:** 2.4.5 Multiple Ways (AA).
**Why:** A user who can't operate one navigation method (e.g. a mega-menu
that's mouse-only, or search if it's not their preferred discovery method)
needs a second way to find the same content.
**Generic detection:** Inventory the distinct navigation/discovery
mechanisms available site-wide (primary nav, search, sitemap, breadcrumbs,
related-content links, category/tag browsing) and confirm at least two
independently reach any given content page — not just that multiple
mechanisms exist somewhere, but that they overlap in coverage.
**Discovery hints:** Header/nav components, a search feature, any
breadcrumb component, a sitemap page/route if one exists.
**N/A when:** the site is a single page or a short, linear process (e.g. a
single checkout flow) where "multiple ways" doesn't meaningfully apply —
per WCAG's own exception for pages that are steps in a process.

---

## A11Y-27 — Viewport zoom & orientation
**What:** The viewport meta tag never disables or caps user zoom
(`user-scalable=no`, or `maximum-scale` below `5`); the layout doesn't
lock to a single device orientation unless that orientation is essential
to the content (e.g. a piano-keyboard app — genuinely rare).
**WCAG SC:** 1.4.4 Resize Text (AA, via viewport zoom) · 1.3.4 Orientation
(AA).
**Why:** Disabling pinch-zoom is one of the most common, entirely
avoidable mobile anti-patterns, and directly blocks low-vision users'
primary zoom mechanism on touch devices. Orientation-locking blocks users
who have their device mounted in a fixed orientation (wheelchair mount,
stand) or who physically cannot rotate it.
**Generic detection:** Grep the root HTML template's `<meta name="viewport">`
for `user-scalable=no` or a restrictive `maximum-scale`; grep global CSS/JS
for any `orientation: portrait`/`landscape` media-query-driven lockout or
the Screen Orientation API's `lock()` call.
**Discovery hints:** Root HTML template (`index.html`/`_document`); global
CSS entry point.
**N/A when:** never for a general-audience web app.

---

## A11Y-28 — Status messages, live regions & loading announcements
**What:** Transient UI feedback — toasts, success/error banners, loading
spinners/skeleton screens, "N more items loaded" after infinite scroll,
streamed/real-time content (chat, live scores) — is exposed via
`aria-live` (or `role="status"`/`role="alert"` as appropriate) so a screen
reader user is notified without needing to be focused on that exact
element when it changes.
**WCAG SC:** 4.1.3 Status Messages (AA).
**Why:** Sighted users see a toast appear in their peripheral vision or a
spinner replaced by content; a screen reader user gets nothing unless the
change happens inside a live region — they may act on stale information
or never learn an action succeeded/failed.
**Generic detection:** Find the shared toast/notification hook or
component and confirm it's built on the UI kit's accessible primitive
(e.g. Chakra's `useToast`, which wires `aria-live` by default) rather than
a custom banner with no ARIA wiring. Separately, check infinite-scroll/
lazy-load/real-time-stream components (chat, live feed, "load more") for
any live-region announcement of new content arriving.
**Discovery hints:** Toast/notification hook; infinite-scroll hook;
WebSocket/SSE-driven components (chat, live scoreboard, live-updating
feed).
**N/A when:** the app has no transient/asynchronous UI feedback at all.

---

## A11Y-29 — Audio description for video
**What:** Where a video conveys meaningful visual-only information not
covered by its existing dialogue/narration (e.g. an on-screen action, a
displayed graphic, a scene change relevant to the story), an audio-
description track or an extended-audio-description version is available.
**WCAG SC:** 1.2.5 Audio Description (Prerecorded) (AA).
**Why:** Blind/low-vision users following video by audio alone miss any
information conveyed purely visually with no corresponding narration.
**Generic detection:** This is largely a content-authoring question, not a
front-end code pattern — the discovery step is identifying whether the
video content type (documentary narration vs. dialogue-only drama vs.
silent-footage-with-music) would plausibly need audio description, then
checking whether the player supports a secondary audio track/description
version at all.
**Discovery hints:** Same video-embed components as A11Y-14; check
whether the player library/embed supports multiple audio tracks.
**N/A when:** all video content already fully narrates its visual content
in dialogue (common for talking-head interviews, lectures), or the app has
no video.

---

## A11Y-30 — Browser & assistive-technology compatibility
**What:** The experience holds up across major browsers (Chrome, Firefox,
Safari, Edge) and mobile browsers, and — ideally — with real assistive
technology (NVDA, JAWS, VoiceOver, TalkBack).
**WCAG SC:** N/A directly — this is a testing-methodology concern that
sits underneath every other SC, not an SC of its own.
**Why:** A pattern that works in Chrome/NVDA can render differently in
Safari/VoiceOver (a long-standing, well-documented source of divergence,
especially for ARIA live regions and custom form controls).
**Generic detection/limitation:** A code-review skill running in this
environment **cannot** drive real browsers or real assistive technology —
there is no automated substitute for this category from static analysis
alone. Report this category as its status genuinely is (`NOT_APPLICABLE`
to this skill's methodology, not to the project) rather than silently
omitting it or guessing a pass. If the project has cross-browser AT test
results from another source (QA team, a paid AT testing service), record
a pointer to where those live instead of attempting to reproduce them here.
**Discovery hints:** N/A.
**N/A when:** always, for this skill specifically — include the category
in every report anyway, explicitly marked `NOT_APPLICABLE` with this
reasoning, so nobody mistakes its absence from the findings for a clean
pass.
