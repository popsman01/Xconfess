# Accessibility (a11y) Checklist — Confession & Profile Pages

Scope: track accessibility issues on the most visible confession and profile
pages. Covers keyboard navigation, labeling, contrast, and focus order.

## Conformance target

Release-level target: **WCAG 2.2 Level AA** on all critical routes listed in
the matrix below. A release is blocked until every critical route passes the
automated gates and the manual checks recorded here. Non-critical routes are
tracked but do not block release.

## Route / component matrix

Each critical route is checked against every gate. `A` = automated (CI),
`M` = manual. A route is release-ready only when all gates pass.

| Route / component | Keyboard | Focus | Semantics | Contrast | Motion | AT behavior |
| --- | --- | --- | --- | --- | --- | --- |
| Confession feed / list view | A+M | A+M | A | A | A | M |
| Confession detail / single view | A+M | A+M | A | A | A | M |
| Confession creation / submission form | A+M | A+M | A | A | A | M |
| `ShareButtons` component | A+M | A+M | A | A | A | M |
| Profile view (own) | A+M | A+M | A | A | A | M |
| Profile view (other user, if public) | A+M | A+M | A | A | A | M |
| Profile edit form | A+M | A+M | A | A | A | M |

Gate definitions:

- **Keyboard** — every interactive element reachable and operable via
  `Tab`/`Shift+Tab`/`Enter`/`Space`; no keyboard traps.
- **Focus** — visible focus indicator (≥3:1), logical order, focus moved into
  modals and restored on close, focus not lost after async actions.
- **Semantics** — programmatic labels, logical heading hierarchy, landmark
  regions, `aria-describedby` on validation errors, `aria-live` for async
  updates.
- **Contrast** — body text ≥4.5:1, large text ≥3:1, UI/focus indicators
  ≥3:1, non-color differentiators for color-coded badges.
- **Motion** — respects `prefers-reduced-motion`; no non-essential animation
  that cannot be disabled; no content that flashes more than 3 times/second.
- **AT behavior** — screen reader (VoiceOver/NVDA) announces names, roles,
  states, and dynamic updates correctly on the route.

---

## 1. Keyboard Navigation

- [ ] Every interactive element (buttons, links, form fields, toggles) is
      reachable via `Tab` / `Shift+Tab` alone — no mouse required
- [ ] Tab order follows visual/reading order, not DOM insertion order that
      happens to differ from layout
- [ ] No keyboard traps — user can always Tab out of a modal, dropdown, or
      widget
- [ ] Confession submission form: full flow (compose → submit → confirmation)
      completable via keyboard only
- [ ] `ShareButtons` — each share action (copy link, share to X, etc.)
      triggerable via `Enter` / `Space`, not click-only handlers
- [ ] Custom components (dropdowns, badge tooltips, reputation displays) use
      standard key interactions: `Enter`/`Space` to activate, `Esc` to close,
      arrow keys where a native `<select>` would use them
- [ ] Skip-to-content link present and functional on confession feed and
      profile pages
- [ ] Infinite-scroll / paginated confession feed doesn't strand keyboard
      focus when new content loads

## 2. Labels & Semantics

- [ ] All form inputs (confession text, profile fields) have a programmatic
      `<label>` (not just placeholder text — placeholders disappear on input
      and aren't reliably read by all screen readers)
- [ ] Icon-only buttons (share, edit profile, badge icons) have `aria-label`
      or visually-hidden text describing the action
- [ ] Images (profile avatars, badge icons) have meaningful `alt` text, or
      `alt=""` if purely decorative
- [ ] Confession anchoring/verification status (e.g. "anchored on-chain") is
      exposed as text, not conveyed by color or icon alone
- [ ] Reputation badges expose their name/meaning to assistive tech, not just
      a decorative icon (ties to `ReputationBadges` contract — badge type
      should be announced, e.g. "ConfessionStarter badge")
- [ ] Form validation errors are associated with their field via
      `aria-describedby`, and errors are announced (e.g. `aria-live` region)
      rather than only shown visually
- [ ] Headings use a logical hierarchy (`h1` → `h2` → `h3`) on both
      confession and profile pages — no skipped levels for styling reasons
- [ ] Landmark regions (`<nav>`, `<main>`, `<header>`) present so screen
      reader users can jump between sections

## 3. Color Contrast

- [ ] Body text meets WCAG AA: 4.5:1 contrast ratio against background
- [ ] Large text / headings meet AA: 3:1 minimum
- [ ] UI component contrast (button borders, form field borders, focus
      indicators) meets 3:1 against adjacent colors
- [ ] Badge/reputation color-coding (e.g. distinguishing badge types by
      color) has a non-color differentiator too (icon, label, shape)
- [ ] Link text is distinguishable from surrounding body text by more than
      color alone (underline or weight difference), especially inside
      confession bodies
- [ ] Dark mode / offline-mode banner (`WebSocketReconnectBanner`,
      `offline/page.tsx`) checked separately — contrast can regress
      independently in alternate themes
- [ ] Contrast checked in both the confession feed's card/list state and
      the expanded detail state, since background treatments often differ

## 4. Focus Order & Visibility

- [ ] Every focusable element has a visible focus indicator (not just
      relying on browser default, which some CSS resets strip) with ≥3:1
      contrast against its background
- [ ] Focus indicator is never removed via `outline: none` without a
      replacement style
- [ ] Opening a modal (e.g. confession detail overlay, share dialog) moves
      focus into the modal; closing it returns focus to the triggering
      element
- [ ] Focus order in the confession submission form matches the visual
      layout (text field → options → submit), not reordered by CSS
      (e.g. flexbox/grid `order`) in a way that breaks tab sequence
- [ ] After async actions (submit confession, award badge, adjust
      reputation) focus lands somewhere sensible — not lost to `<body>`
- [ ] Toast/notification content (e.g. reconnect banner, submission
      confirmation) doesn't steal focus unexpectedly, but is announced via
      `aria-live="polite"`
- [ ] Profile edit form: focus moves to the first invalid field on failed
      validation submit

## 5. Motion & Animation

- [ ] Non-essential animation (transitions, spinners, feed entry effects)
      is disabled or reduced under `prefers-reduced-motion: reduce`
- [ ] No content flashes more than three times per second (WCAG 2.2 §2.3.1)
- [ ] Auto-updating content (live feed, reconnect banner) can be paused,
      stopped, or hidden (WCAG 2.2 §2.2.2)
- [ ] Motion is never the only means of conveying state or feedback

## 6. Automated & Manual Checks

Automated (deterministic, run in CI):

- [ ] `npm run frontend:lint` — includes `eslint-plugin-jsx-a11y` rules
- [ ] `npm run frontend:test` — component tests assert roles, labels, and
      `aria-*` contracts for the matrix routes
- [ ] `npm run frontend:test:e2e` — axe-core scan per critical route; any
      critical/serious violation fails the run

Manual (recorded per release):

- [ ] Keyboard-only pass completed on all matrix routes
- [ ] Screen reader pass (VoiceOver or NVDA) on confession submission flow
      and profile edit flow at minimum
- [ ] Contrast spot-check in light and dark themes on feed and detail states
- [ ] Reduced-motion pass with OS setting enabled

## 7. CI Regression Gate

- [ ] Automated axe scan is wired into `frontend:test:e2e` and fails the
      build on any new critical/serious violation on a critical route
- [ ] Component a11y assertions in `frontend:test` fail on missing labels,
      roles, or `aria-*` contracts
- [ ] A failing gate blocks merge; waivers require an owner and expiry noted
      in the deferred-items list below

## 8. Testing & Sign-off

- [ ] Automated scan run (axe, Lighthouse, or equivalent) against confession
      feed, confession detail, and profile pages — zero critical/serious
      issues outstanding
- [ ] Manual keyboard-only pass completed on all target surfaces above
- [ ] Manual screen reader pass (VoiceOver or NVDA) completed on confession
      submission flow and profile edit flow at minimum
- [ ] Known issues logged with owner and severity if not fixed before this
      review cycle
- [ ] Re-test scheduled for next review cycle if any items are deferred

**Decision:** ☐ Meets bar ☐ Meets bar with deferred items (list below) ☐ Does not meet bar

_Deferred items / owners:_
