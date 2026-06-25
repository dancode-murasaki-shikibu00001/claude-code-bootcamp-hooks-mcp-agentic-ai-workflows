# Technical Specification — Issue #2

## 1. Issue Overview

| Field       | Value                                                                 |
|-------------|-----------------------------------------------------------------------|
| Title       | Inside the footer, when hover onto the "Cookie Policy" nothing being displayed |
| Description | Hovering over "Cookie Policy" in the footer showed no content. It should display text about the site's cookie usage. |
| Labels      | None                                                                  |
| State       | CLOSED (resolved in PR #3)                                            |
| Priority    | Low                                                                   |

---

## 2. Problem Analysis

**Root cause (verified from code):**

`src/components/Footer.jsx` contained three footer legal links — Privacy Policy, Terms of Service, and Cookie Policy — all rendered as bare `<a>` elements with hover highlight styling only. None had any interactive content (tooltip, modal, popover) wired to them. Cookie Policy was specifically called out in the issue.

The `<a>` for Cookie Policy (pre-fix, line ~152) had:
- A `group-hover` background highlight
- No `title` attribute
- No `onMouseEnter`/`onMouseLeave` handlers
- No associated tooltip or popover element

No tooltip infrastructure existed anywhere in the codebase for these footer links.

---

## 3. Proposed Solution

Add a self-contained `CookiePolicyTooltip` component inside `Footer.jsx` that:
- Uses a single `useState(false)` flag to toggle visibility
- Shows a tooltip card above the link on `mouseenter`, hides it on `mouseleave`
- Displays a brief, readable cookie policy description
- Is styled exclusively with Tailwind CSS utility classes
- Does not introduce any new library or abstraction

This is the minimal patch-level change. Privacy Policy and Terms of Service are not called out in the issue and are left unchanged.

**Trade-off noted:** Co-locating `CookiePolicyTooltip` inside `Footer.jsx` triggers the `react-refresh/only-export-components` ESLint warning (non-default export of a component in a file that also has a default export). This is acceptable given the component's small size and single-file scope. If the footer grows, the component could be extracted to `src/components/CookiePolicyTooltip.jsx`.

---

## 4. Step-by-Step Implementation

1. **Add `useState` import** — add `import { useState } from "react"` at the top of `Footer.jsx`.
2. **Create `CookiePolicyTooltip` component** — define a named functional component above `Footer` that manages a `visible` boolean via `useState`. Render an `<a>` trigger with `onMouseEnter`/`onMouseLeave` handlers. Conditionally render a positioned tooltip `<div>` with cookie policy copy when `visible` is `true`.
3. **Replace inline `<a>` for Cookie Policy** — in `Footer`'s JSX, replace the bare `<a>Cookie Policy</a>` element with `<CookiePolicyTooltip />`.
4. **Verify styling** — tooltip uses `absolute bottom-full` positioning, a downward CSS border-triangle arrow, dark `bg-gray-800` background, and `z-50` to clear the footer layout.

---

## 5. Verification Strategy

### Unit Tests
- No test infrastructure exists in this project (`package.json` has no test script). Unit tests are out of scope.

### Integration Tests
- None applicable — no test framework configured.

### Manual Checks
- Navigate to any page → scroll to footer.
- Hover over **Cookie Policy** → tooltip with heading "Cookie Policy" and descriptive text should appear above the link.
- Move mouse away → tooltip should disappear.
- Hover over **Privacy Policy** and **Terms of Service** → no tooltip should appear (unchanged).
- Test on mobile viewport (touch device): tooltip is mouse-driven; on touch screens the link simply has no interaction, which is acceptable per the issue scope.
- Verify dark/light theme toggle does not affect tooltip visibility (tooltip uses hard-coded dark palette, consistent with footer's fixed dark background).

---

## 6. Files to Modify

| File Path                        | Nature of Change                                                  |
|----------------------------------|-------------------------------------------------------------------|
| `src/components/Footer.jsx`      | Add `useState` import; add `CookiePolicyTooltip` component; replace Cookie Policy `<a>` with `<CookiePolicyTooltip />` |

---

## 7. New Files to Create

None required for this fix.

---

## 8. Existing Utilities to Leverage

| Utility                        | Benefit                                                      |
|--------------------------------|--------------------------------------------------------------|
| Tailwind CSS utility classes   | Tooltip positioning (`absolute`, `bottom-full`, `z-50`) and styling without new CSS |
| React `useState`               | Already used throughout the codebase for local UI toggle state |

---

## 9. Acceptance Criteria

- Hovering over "Cookie Policy" in the footer displays a tooltip with readable cookie policy text.
- Tooltip disappears when the mouse leaves.
- "Privacy Policy" and "Terms of Service" links are visually and functionally unchanged.
- No new libraries introduced.
- `npm run build` passes without errors.

---

## 10. Out of Scope

- Adding tooltips or content to Privacy Policy and Terms of Service (not requested in issue).
- Creating dedicated `/cookie-policy`, `/privacy-policy`, or `/terms` routes.
- Persisting user cookie consent (no backend/localStorage consent tracking).
- Accessibility improvements (keyboard focus trigger, ARIA roles) — not called out in issue.
- Mobile/touch interaction for the tooltip.
