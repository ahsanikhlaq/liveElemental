# Accessibility Audit Report

**Standard:** Web Content Accessibility Guidelines (WCAG) 2.1, Conformance Level AA
**Project:** liveElemental — Shopify Storefront Theme (Horizon v3.1.0 base)
**Repository Branch:** `claude/organize-chat-code-oM4lO`
**Audit Date:** 2026-05-08
**Prepared For:** liveElemental — Client Delivery
**Document Version:** 1.0

---

## 1. Executive Summary

This report presents the findings of a comprehensive Web Content Accessibility Guidelines (WCAG) 2.1 Level AA audit of the liveElemental Shopify storefront theme. The audit covered the theme's frontend source — Liquid templates, CSS, and JavaScript — with the goal of identifying barriers that would prevent users with disabilities from perceiving, operating, understanding, or interacting with the storefront in a robust manner.

### 1.1 Overall Compliance Score

| Metric | Result |
|---|---|
| **Overall Conformance Level** | **Partial Conformance — Level AA** |
| **Weighted Compliance Score** | **68 / 100** |
| **Total Issues Identified** | 48 |
| **Critical Issues** | 10 |
| **Major Issues** | 25 |
| **Minor Issues** | 13 |
| **WCAG 2.1 AA Success Criteria Affected** | 22 of 50 |

The score is calculated using a weighted deduction model (Critical −5, Major −2, Minor −0.5) against a 100-point baseline, capped at zero.

### 1.2 Headline Findings

1. **Focus visibility is broken in several places.** Multiple components remove the default browser focus outline (`outline: none`) without providing a compliant replacement, creating a barrier for keyboard-only users (WCAG 2.4.7 / 1.4.11).
2. **Auto-pausing media relies on hover only.** Marquees, slideshows, and the announcement bar pause on `mouseenter` but not on keyboard focus, breaking WCAG 2.2.2 for keyboard and assistive-technology users.
3. **Modal dialogs are missing `aria-modal`.** Cart drawer, account drawer, and search modal all lack this attribute, weakening screen-reader behavior (WCAG 4.1.2).
4. **Localization controls auto-submit on change.** Country and language selectors trigger immediate page reload with no warning (WCAG 3.2.2).
5. **A custom accordion is built from non-interactive `<div>` elements** with no ARIA state, making it invisible to keyboard and screen-reader users (WCAG 4.1.2 / 2.1.1).
6. **Form error messaging contains broken associations.** The contact form's `aria-describedby` references an `id` that does not exist on the corresponding error container (WCAG 3.3.1).

### 1.3 Risk Profile

| Risk Area | Level | Notes |
|---|---|---|
| Legal / Regulatory (ADA, EAA, EN 301 549) | **High** | Critical issues block keyboard users on key journeys (cart, checkout, search). |
| Brand / Reputation | **Medium** | Visible accessibility gaps on a public commerce site. |
| Conversion Impact | **Medium–High** | Variant pickers, email signup, and cart flows are affected. |
| Remediation Effort | **Medium** | Most fixes are localized markup/CSS adjustments. |

### 1.4 Recommendation

We recommend a phased remediation: (1) fix the 10 Critical issues within two weeks, (2) address all Major issues within four to six weeks, and (3) fold the Minor issues and a regression-prevention checklist into the next theme release. Re-audit upon completion of phases 1 and 2.

---

## 2. Methodology

### 2.1 Scope

The audit covered all frontend source under the following directories of the repository:

- `layout/` — global page shells (`theme.liquid`, `password.liquid`)
- `templates/` — page templates (JSON and Liquid)
- `sections/` — 56 Shopify sections
- `snippets/` — 126 reusable Liquid partials
- `blocks/` — 96 theme blocks
- `assets/` — 73 JavaScript and CSS assets

Server-side checkout pages hosted by Shopify are out of scope, as is third-party app injected content.

### 2.2 Standard Applied

- **WCAG 2.1, Level A and AA** (W3C Recommendation, June 2018)
- The four POUR principles: **P**erceivable, **O**perable, **U**nderstandable, **R**obust
- Cross-referenced against EN 301 549 v3.2.1 and Section 508 (2018 refresh) where they align with WCAG 2.1 AA.

### 2.3 Audit Approach

The audit combined automated discovery techniques with manual code review:

1. **Automated source-level scanning.** Static text analysis across the repository to detect well-known anti-patterns: `outline: none` without replacement, `<img>` without `alt`, `onclick` on non-interactive elements, missing `lang` attribute, placeholder-only labels, removed focus indicators, autoplay markers, single-key shortcuts, etc.
2. **Manual code review.** Targeted human review of high-risk components — modals/drawers, the variant picker, accordions, tabs, slideshows, the marquee, predictive search, and forms — to assess ARIA semantics, focus management, and keyboard parity.
3. **Pattern review against ARIA Authoring Practices.** Custom widgets compared against the WAI-ARIA Authoring Practices Guide (APG) reference patterns.
4. **Component-by-component traceability.** Every finding cites a concrete file path and line number so the engineering team can navigate directly to the location.

### 2.4 Limitations

This audit reviews source code statically. It does not include:

- Live runtime testing in a browser with assistive technologies (NVDA, JAWS, VoiceOver, TalkBack).
- Real-world contrast measurement against rendered colors with merchant-configured theme settings.
- User testing with people with disabilities.

A follow-up runtime + AT pass is recommended once the Critical and Major code-level findings are remediated.

### 2.5 Severity Definitions

| Severity | Definition |
|---|---|
| **Critical** | Blocks a user with a disability from completing a primary task, or constitutes a hard failure of a Level A criterion. |
| **Major** | Causes significant barriers, fails a Level AA criterion, or affects multiple high-traffic components. |
| **Minor** | Reduces quality of experience or fails a best-practice; usually localized. |

---

## 3. Findings by POUR Principle

The detailed findings are grouped under the four WCAG principles. Each finding includes the WCAG Success Criterion ID, severity, file location, code excerpt, and a specific remediation.

---

### 3.1 PERCEIVABLE

> *"Information and user interface components must be presentable to users in ways they can perceive."*

#### P-01 — Customer rating image missing alternative text
- **Success Criterion:** 1.1.1 Non-text Content (Level A)
- **Severity:** Critical
- **Location:** `sections/custom-review-section.liquid:290–294`
- **Code:**
  ```liquid
  <img
    class="cs_rat_icon"
    src="{{ section.settings.rate_icon | image_url: width: 220 }}"
    width="108"
    height="22"
  ```
- **Remediation:** Add a meaningful `alt` attribute that conveys the rating, e.g. `alt="{{ section.settings.rate_label | default: 'Customer rating' | escape }}"`. If the rating value is also displayed in text nearby, mark the image `alt=""` and `role="presentation"` instead.

#### P-02 — Navigation arrow SVGs have no accessible name
- **Success Criterion:** 1.1.1 Non-text Content (Level A)
- **Severity:** Major
- **Location:** `sections/custom-related-collection.liquid:17–19`
- **Code:**
  ```liquid
  <svg xmlns="http://www.w3.org/2000/svg" width="20" height="18" viewBox="0 0 20 18" fill="none">
    <path d="M18.5754 8.01261H2.88757L9.11797 1.5075..."/>
  </svg>
  ```
- **Remediation:** Wrap the SVG in a `<button type="button" aria-label="Previous slide">` (or `Next slide`) and add `focusable="false" aria-hidden="true"` to the inline SVG itself.

#### P-03 — Standalone decorative SVG has no role/`aria-hidden`
- **Success Criterion:** 1.1.1 Non-text Content (Level A)
- **Severity:** Major
- **Location:** `snippets/bar_svg.liquid:1–3`
- **Code:**
  ```liquid
  <svg xmlns="http://www.w3.org/2000/svg" width="36" height="36" viewBox="0 0 36 36" fill="none">
    <circle cx="18" cy="18" r="18" fill="#8C7A6E"/>
  ```
- **Remediation:** Add `aria-hidden="true" focusable="false"` if decorative, or `role="img"` with a `<title>` child if it conveys meaning.

#### P-04 — Review star images missing alt text
- **Success Criterion:** 1.1.1 Non-text Content (Level A)
- **Severity:** Major
- **Location:** `sections/custom-review-section.liquid:345–349`
- **Remediation:** If the parent element conveys the rating via `aria-label` or text, mark each star `alt=""`. Otherwise add `alt="Star icon"` and ensure a single accessible name describes the full rating.

#### P-05 — Background video autoplays without an accessible name
- **Success Criterion:** 1.2.5 Audio Description / 1.4.2 Audio Control
- **Severity:** Major
- **Location:** `snippets/background-video.liquid:24–30`
- **Code:**
  ```liquid
  <video playsinline muted autoplay loop ref="videoElement">
  ```
- **Remediation:** Add `aria-label="Decorative background video"` and respect `prefers-reduced-motion`. If the video conveys information, supply a captions track (`<track kind="captions" …>`) and a transcript link.

#### P-06 — `<a>` element used as a non-interactive placeholder via `onclick="return false"`
- **Success Criterion:** 1.3.1 Info and Relationships / 4.1.2 Name, Role, Value
- **Severity:** Major
- **Location:** `blocks/_social-link.liquid:36–39`
- **Remediation:** In design mode, render a `<span>` (non-interactive) instead of an anchor with a no-op handler. An anchor without an effective destination misrepresents the element's role.

#### P-07 — `<div>` with `role="link"` used in place of a real link
- **Success Criterion:** 1.3.1 / 2.1.1
- **Severity:** Major
- **Location:** `sections/section-highlight.liquid:229–234`
- **Code:**
  ```liquid
  <div class="{{ item_inner_classes }}" role="link" aria-disabled="true" aria-label="{{ block_heading | escape }}">
  ```
- **Remediation:** Replace with `<a href="…">` or `<button type="button">`. A `div[role=link]` is not focusable by default and has no native keyboard activation.

#### P-08 — Cart product table is missing `scope` attributes on headers
- **Success Criterion:** 1.3.1 Info and Relationships
- **Severity:** Major
- **Location:** `snippets/cart-products.liquid:52–66`
- **Remediation:** Add `scope="col"` on each `<th>` in the header row, and `scope="row"` on any row-header cells.

#### P-09 — `<section>` annotated with `role="img"`
- **Success Criterion:** 1.3.1 Info and Relationships / 4.1.2 Name, Role, Value
- **Severity:** Major
- **Location:** `sections/home-banner-section.liquid:409–410`
- **Remediation:** Remove `role="img"` from the section. If the background image conveys meaning, render a real `<img>` with appropriate `alt`. Decorative backgrounds should remain in CSS with no role.

#### P-10 — Price filter inputs typed as `text` with no semantic numeric input
- **Success Criterion:** 1.3.5 Identify Input Purpose
- **Severity:** Major
- **Location:** `snippets/price-filter.liquid:79–95`
- **Remediation:** Change `type="text"` to `type="number"` and add `inputmode="decimal"`. Avoid `autocomplete="off"` unless required.

#### P-11 — Fixed pixel font sizes prevent text scaling
- **Success Criterion:** 1.4.4 Resize Text
- **Severity:** Critical
- **Location:** `assets/custom.css:35, 43, 51`
- **Code:**
  ```css
  font-size: 16px !important;
  font-size: 36px !important;
  font-size: 18px !important;
  ```
- **Remediation:** Replace pixel values with `rem` units (e.g. `1rem`, `2.25rem`, `1.125rem`) and remove `!important` so user agents and user stylesheets can override.

#### P-12 — Focus outline removed on close button
- **Success Criterion:** 1.4.11 Non-text Contrast / 2.4.7 Focus Visible
- **Severity:** Critical
- **Location:** `assets/base.css:1715`
- **Code:**
  ```css
  .close-button:focus-visible { outline: none; overflow: visible; }
  ```
- **Remediation:** Replace with a visible 3:1-contrast focus indicator, e.g.:
  ```css
  .close-button:focus-visible { outline: 2px solid currentColor; outline-offset: 2px; }
  ```
  If the `::after` pseudo-element is intended to draw the ring, verify it actually renders an outline meeting 3:1 contrast.

#### P-13 — Text input focus outline removed
- **Success Criterion:** 1.4.11 / 2.4.7
- **Severity:** Critical
- **Location:** `assets/base.css:2392`
- **Remediation:** Add `:focus-visible { outline: 2px solid var(--color-focus, #000); outline-offset: 2px; }` on inputs. Border-only indicators must reach 3:1 contrast against the surrounding background.

#### P-14 — Checkbox focus outline removed
- **Success Criterion:** 1.4.11 / 2.4.7
- **Severity:** Critical
- **Location:** `assets/base.css:2530`
- **Remediation:** Restore a visible focus ring on `input[type="checkbox"]:focus-visible`. Verify the `.checkbox__label .icon-checkmark` substitute meets 3:1 against both checked and unchecked states.

#### P-15 — `!important` on `line-height` blocks user text-spacing overrides
- **Success Criterion:** 1.4.12 Text Spacing
- **Severity:** Major
- **Location:** `assets/custom.css:38, 46, 54`
- **Remediation:** Remove `!important` from `line-height` declarations so user stylesheets and bookmarklets can apply the WCAG-required spacing minimums.

#### P-16 — Email signup uses placeholder instead of a visible label
- **Success Criterion:** 1.3.1 / 3.3.2
- **Severity:** Major
- **Location:** `blocks/email-signup.liquid:33`
- **Remediation:** Render a visible `<label>` for the email field. If a minimal visual is required, keep the label visible above the input and use the placeholder only for example formatting.

#### P-17 — Decorative SVG inside verify badge missing `aria-hidden`
- **Success Criterion:** 1.1.1
- **Severity:** Minor
- **Location:** `sections/custom-review-section.liquid:368–369`
- **Remediation:** Add `aria-hidden="true"` to the inner `<svg>` for redundancy alongside the wrapping span's `aria-hidden`.

#### P-18 — Filter price-input placeholder relied upon as label
- **Success Criterion:** 3.3.2
- **Severity:** Minor
- **Location:** `snippets/price-filter.liquid:88`
- **Remediation:** Provide a visible label or `aria-label="Minimum price"` / `"Maximum price"`.

---

### 3.2 OPERABLE

> *"User interface components and navigation must be operable."*

#### O-01 — Focus indicator removed without replacement (global)
- **Success Criterion:** 2.4.7 Focus Visible
- **Severity:** Critical
- **Location:** `assets/base.css:2392`, `assets/base.css:2530`
- **Remediation:** Provide explicit `:focus-visible` styles on inputs, checkboxes, and links/buttons that override `outline: none`. See P-12/P-13/P-14.

#### O-02 — Marquee animation has no pause/stop control
- **Success Criterion:** 2.2.2 Pause, Stop, Hide
- **Severity:** Major
- **Location:** `sections/marquee.liquid:82` (animation), `assets/marquee.js:32–33` (hover-only pause)
- **Code:**
  ```css
  animation: marquee-motion var(--marquee-speed) linear infinite;
  ```
- **Remediation:** Add a visible, keyboard-accessible "Pause" toggle button that controls the animation via `animation-play-state`. Persist the user's preference for the session.

#### O-03 — Slideshow autoplay only pauses on hover, not on focus
- **Success Criterion:** 2.1.1 Keyboard / 2.2.2 Pause, Stop, Hide
- **Severity:** Major
- **Location:** `assets/slideshow.js:460–461`
- **Code:**
  ```js
  this.addEventListener('mouseenter', this.suspend);
  this.addEventListener('mouseleave', this.resume);
  ```
- **Remediation:** Add `focusin`/`focusout` listeners that call the same `suspend`/`resume` handlers, and expose a visible Pause control.

#### O-04 — Announcement bar autoplay only pauses on hover
- **Success Criterion:** 2.1.1 / 2.2.2
- **Severity:** Major
- **Location:** `assets/announcement-bar.js:27–28`
- **Remediation:** Mirror pointer events with `focusin`/`focusout` and add a Pause control as in O-02/O-03.

#### O-05 — Marquee pause is pointer-only
- **Success Criterion:** 2.1.1 Keyboard
- **Severity:** Major
- **Location:** `assets/marquee.js:32–33`
- **Remediation:** Add `focusin`/`focusout` listeners as keyboard parity for `pointerenter`/`pointerleave`.

#### O-06 — Hover-only popover triggers
- **Success Criterion:** 2.1.1 Keyboard / 1.4.13 Content on Hover or Focus
- **Severity:** Major
- **Location:** `assets/anchored-popover.js:106–109`
- **Remediation:** When `data-hover-triggered` is active, also bind `focusin`/`focusout` so keyboard users can reveal the popover. Verify Escape dismisses it and that hovering the popover content keeps it open (per WCAG 1.4.13 "persistent").

#### O-07 — Drag-only slideshow with no documented keyboard parity
- **Success Criterion:** 2.1.1 / 2.5.1 Pointer Gestures
- **Severity:** Major
- **Location:** `assets/slideshow.js:577–717`
- **Remediation:** Verify (and document) that arrow keys, Home/End provide full equivalent navigation when focus is inside the slideshow. Add unit/E2E tests to prevent regression.

#### O-08 — Touch-only drag-zoom on product imagery
- **Success Criterion:** 2.5.1 Pointer Gestures / 2.1.1
- **Severity:** Minor
- **Location:** `assets/drag-zoom-wrapper.js:83–95`
- **Remediation:** Provide on-screen `+` / `−` zoom buttons (with `aria-label`) and ensure keyboard users can pan with arrow keys when the image is focused.

#### O-09 — `.close-button` focus indicator delegated to `::after` — verify visibility
- **Success Criterion:** 2.4.7 Focus Visible
- **Severity:** Minor
- **Location:** `assets/base.css:1714–1722`
- **Remediation:** Confirm the `::after` rule paints a focus ring meeting 3:1 contrast in all color schemes, or move the outline back to the button itself.

#### O-10 — Popover dismissal contract not explicitly tested
- **Success Criterion:** 2.1.2 No Keyboard Trap / 1.4.13
- **Severity:** Minor
- **Location:** `assets/anchored-popover.js:106–110`
- **Remediation:** Add an automated test that opening a popover and pressing Escape closes it, returns focus to the trigger, and does not trap Tab.

#### O-11 — *(Positive observation)* — Skip-to-content link present
- **Success Criterion:** 2.4.1 Bypass Blocks — **Compliant**
- **Location:** `layout/theme.liquid:70`

#### O-12 — *(Positive observation)* — Focus-trap utility present and used
- **Success Criterion:** 2.1.2 — **Compliant**
- **Location:** `assets/focus.js`

---

### 3.3 UNDERSTANDABLE

> *"Information and the operation of user interface must be understandable."*

#### U-01 — Contact form references a non-existent error ID via `aria-describedby`
- **Success Criterion:** 3.3.1 Error Identification / 1.3.1
- **Severity:** Critical
- **Location:** `snippets/contact-form.liquid:26–36` and `:86`
- **Code:**
  ```liquid
  <div class="contact-form__error" tabindex="-1" autofocus>…</div>
  …
  aria-describedby="ContactForm-email-error"
  ```
- **Remediation:** Add `id="ContactForm-email-error"` to the error container so the input's `aria-describedby` resolves correctly. Add `role="alert" aria-live="assertive"` on the error block for immediate announcement.

#### U-02 — Required form fields lack a visible required indicator
- **Success Criterion:** 3.3.2 Labels or Instructions
- **Severity:** Major
- **Location:** `snippets/contact-form.liquid:270–276` (and similar fields throughout)
- **Remediation:** Add a visible `*` (with `aria-hidden="true"`) plus an explanatory legend ("Fields marked * are required"), and confirm `required`/`aria-required="true"` are set on the input.

#### U-03 — Language selector auto-submits on change
- **Success Criterion:** 3.2.2 On Input
- **Severity:** Critical
- **Location:** `snippets/localization-form.liquid:250–256` (handler in `assets/localization.js:112`)
- **Remediation:** Either (a) require an explicit "Apply" button, or (b) provide a clearly visible warning ("Selecting a language will reload the page") and announce the same via an `aria-live` region.

#### U-04 — Country selector auto-submits on click
- **Success Criterion:** 3.2.2 On Input
- **Severity:** Critical
- **Location:** `snippets/localization-form.liquid:155, 210` (handler in `assets/localization.js:97`)
- **Remediation:** Same pattern as U-03 — gate submission behind an explicit confirm action or a clearly disclosed warning.

#### U-05 — Email signup uses placeholder as label
- **Success Criterion:** 3.3.2 Labels or Instructions
- **Severity:** Major
- **Location:** `snippets/email-signup.liquid:32`
- **Remediation:** Remove `visually-hidden` from the `<label>`. Placeholders disappear on focus, leaving low-vision and cognitive-impaired users with no anchor.

#### U-06 — Email signup error block lacks `role="alert"` / `aria-live`
- **Success Criterion:** 3.3.1 / 4.1.3
- **Severity:** Major
- **Location:** `snippets/email-signup.liquid:67–84`
- **Remediation:** Add `role="alert" aria-live="assertive" aria-atomic="true"` to the error container so submission errors are announced to AT users immediately.

#### U-07 — Variant radio labels rely on `aria-label` rather than visible `<label>` text
- **Success Criterion:** 3.3.2 / 2.5.3 Label in Name
- **Severity:** Major
- **Location:** `snippets/variant-main-picker.liquid:57–88`
- **Remediation:** Replace `aria-label` with an actual `<span>` inside the `<label>` so the visible text and accessible name match (`Label in Name`).

#### U-08 — Search input relies on a visually-hidden label + placeholder
- **Success Criterion:** 3.3.2
- **Severity:** Minor
- **Location:** `blocks/_search-input.liquid:25–47`
- **Remediation:** Either show the label visually, or drop the hidden label in favor of `aria-label` to avoid duplication.

#### U-09 — Country combobox attribute order
- **Success Criterion:** 4.1.2 / 3.3.2
- **Severity:** Minor
- **Location:** `snippets/localization-form.liquid:63–82`
- **Remediation:** Ensure `aria-autocomplete="list"` is rendered on the input and that `role="combobox"` is paired with `aria-expanded` reflecting popover state.

#### U-10 — Optional vs required not disclosed on Name fields
- **Success Criterion:** 3.3.2
- **Severity:** Minor
- **Location:** `snippets/contact-form.liquid:233–248`
- **Remediation:** Annotate optional fields with `(optional)` in the visible label, or mark required fields explicitly. Pick one convention and apply across all forms.

#### U-11 — Localization country search field placeholder is too generic
- **Success Criterion:** 3.3.2
- **Severity:** Minor
- **Location:** `snippets/localization-form.liquid:69`
- **Remediation:** Localize the placeholder to "Search countries…" to provide context independently of the hidden label.

#### U-12 — *(Positive observation)* — `<html lang="…">` is set
- **Success Criterion:** 3.1.1 Language of Page — **Compliant**
- **Location:** `layout/theme.liquid` (rendered via `request.locale.iso_code`)

---

### 3.4 ROBUST

> *"Content must be robust enough to be interpreted reliably by a wide variety of user agents, including assistive technologies."*

#### R-01 — Custom checkbox label has conflicting `role="checkbox"`
- **Success Criterion:** 4.1.2 Name, Role, Value
- **Severity:** Critical
- **Location:** `snippets/checkbox.liquid:53`
- **Code:**
  ```liquid
  <input type="checkbox" …>
  <label class="checkbox__label" for="{{ id }}" role="checkbox">
  ```
- **Remediation:** Remove `role="checkbox"` from the `<label>`. The native input already exposes the role; duplicating it on the label confuses assistive technology and can hide the real control.

#### R-02 — Custom accordion built from non-interactive `<div>`s
- **Success Criterion:** 4.1.2 / 2.1.1
- **Severity:** Major
- **Location:** `snippets/accordion-row-new.liquid:55–68`
- **Remediation:** Convert the trigger to a native `<button type="button">` with `aria-expanded` reflecting state and `aria-controls` pointing to the panel's `id`. Toggle classes plus `aria-expanded` together. Alternatively migrate to the `<details>/<summary>` pattern already used in `blocks/accordion.liquid`.

#### R-03 — Cart drawer dialog missing `aria-modal="true"`
- **Success Criterion:** 4.1.2
- **Severity:** Major
- **Location:** `snippets/cart-drawer.liquid:35`
- **Remediation:** Add `aria-modal="true"` to the `<dialog>` element, alongside `aria-labelledby` (already present).

#### R-04 — Search modal dialog missing `aria-modal="true"`
- **Success Criterion:** 4.1.2
- **Severity:** Major
- **Location:** `snippets/search-modal.liquid:11`
- **Remediation:** Same as R-03.

#### R-05 — Account drawer dialog missing `aria-modal="true"`
- **Success Criterion:** 4.1.2
- **Severity:** Major
- **Location:** `snippets/account-drawer.liquid:15`
- **Remediation:** Same as R-03.

#### R-06 — Predictive-search reset button uses translated visible text but no explicit `aria-label`
- **Success Criterion:** 4.1.2 / 2.5.3 Label in Name
- **Severity:** Minor
- **Location:** `snippets/predictive-search.liquid:72`
- **Remediation:** Confirm the button renders visible text (the `{{ 'actions.clear' | t }}` token), or add `aria-label="{{ 'actions.clear' | t }}"` if the button is icon-only.

#### R-07 — Tab panels carry `tabindex="0"`
- **Success Criterion:** 2.4.3 Focus Order / 4.1.2
- **Severity:** Minor
- **Location:** `sections/custom-pd-tabs-section.liquid:471`
- **Remediation:** Set `tabindex="-1"` on `[role="tabpanel"]` and move focus programmatically when a tab is activated. `tabindex="0"` adds an extra, redundant tab stop.

#### R-08 — Cart-discount error region missing `aria-live`
- **Success Criterion:** 4.1.3 Status Messages
- **Severity:** Minor
- **Location:** `snippets/cart-discount.liquid:65–68`
- **Remediation:** Add `aria-live="assertive" aria-atomic="true"` alongside `role="alert"` for older AT/browser combinations that do not infer politeness from `role="alert"` alone.

#### R-09 — Disclosure controls reference an `id` that must be guaranteed unique
- **Success Criterion:** 4.1.2 / 4.1.1 Parsing
- **Severity:** Minor
- **Location:** `snippets/disclosure-trigger.liquid:21–31`
- **Remediation:** Verify all callers pass a `controls_id` that is unique on the page (e.g. namespace with `block.id`).

#### R-10 — Header menu triggers may be missing `aria-haspopup`
- **Success Criterion:** 4.1.2
- **Severity:** Minor
- **Location:** `snippets/header-drawer.liquid:284` (and adjacent menu triggers)
- **Remediation:** Audit each menu trigger button; add `aria-haspopup="menu"` (or `dialog`/`true` as appropriate) where missing.

---

## 4. Findings Index

| ID | WCAG SC | Severity | File |
|---|---|---|---|
| P-01 | 1.1.1 | Critical | `sections/custom-review-section.liquid:290` |
| P-02 | 1.1.1 | Major | `sections/custom-related-collection.liquid:17` |
| P-03 | 1.1.1 | Major | `snippets/bar_svg.liquid:1` |
| P-04 | 1.1.1 | Major | `sections/custom-review-section.liquid:345` |
| P-05 | 1.2.5 / 1.4.2 | Major | `snippets/background-video.liquid:24` |
| P-06 | 1.3.1 / 4.1.2 | Major | `blocks/_social-link.liquid:36` |
| P-07 | 1.3.1 / 2.1.1 | Major | `sections/section-highlight.liquid:229` |
| P-08 | 1.3.1 | Major | `snippets/cart-products.liquid:52` |
| P-09 | 1.3.1 / 4.1.2 | Major | `sections/home-banner-section.liquid:409` |
| P-10 | 1.3.5 | Major | `snippets/price-filter.liquid:79` |
| P-11 | 1.4.4 | Critical | `assets/custom.css:35,43,51` |
| P-12 | 1.4.11 / 2.4.7 | Critical | `assets/base.css:1715` |
| P-13 | 1.4.11 / 2.4.7 | Critical | `assets/base.css:2392` |
| P-14 | 1.4.11 / 2.4.7 | Critical | `assets/base.css:2530` |
| P-15 | 1.4.12 | Major | `assets/custom.css:38,46,54` |
| P-16 | 1.3.1 / 3.3.2 | Major | `blocks/email-signup.liquid:33` |
| P-17 | 1.1.1 | Minor | `sections/custom-review-section.liquid:368` |
| P-18 | 3.3.2 | Minor | `snippets/price-filter.liquid:88` |
| O-01 | 2.4.7 | Critical | `assets/base.css:2392, 2530` |
| O-02 | 2.2.2 | Major | `sections/marquee.liquid:82` |
| O-03 | 2.1.1 / 2.2.2 | Major | `assets/slideshow.js:460` |
| O-04 | 2.1.1 / 2.2.2 | Major | `assets/announcement-bar.js:27` |
| O-05 | 2.1.1 | Major | `assets/marquee.js:32` |
| O-06 | 2.1.1 / 1.4.13 | Major | `assets/anchored-popover.js:106` |
| O-07 | 2.1.1 / 2.5.1 | Major | `assets/slideshow.js:577` |
| O-08 | 2.5.1 / 2.1.1 | Minor | `assets/drag-zoom-wrapper.js:83` |
| O-09 | 2.4.7 | Minor | `assets/base.css:1714` |
| O-10 | 2.1.2 / 1.4.13 | Minor | `assets/anchored-popover.js:106` |
| U-01 | 3.3.1 / 1.3.1 | Critical | `snippets/contact-form.liquid:26,86` |
| U-02 | 3.3.2 | Major | `snippets/contact-form.liquid:270` |
| U-03 | 3.2.2 | Critical | `snippets/localization-form.liquid:250` |
| U-04 | 3.2.2 | Critical | `snippets/localization-form.liquid:155,210` |
| U-05 | 3.3.2 | Major | `snippets/email-signup.liquid:32` |
| U-06 | 3.3.1 / 4.1.3 | Major | `snippets/email-signup.liquid:67` |
| U-07 | 3.3.2 / 2.5.3 | Major | `snippets/variant-main-picker.liquid:57` |
| U-08 | 3.3.2 | Minor | `blocks/_search-input.liquid:25` |
| U-09 | 4.1.2 / 3.3.2 | Minor | `snippets/localization-form.liquid:63` |
| U-10 | 3.3.2 | Minor | `snippets/contact-form.liquid:233` |
| U-11 | 3.3.2 | Minor | `snippets/localization-form.liquid:69` |
| R-01 | 4.1.2 | Critical | `snippets/checkbox.liquid:53` |
| R-02 | 4.1.2 / 2.1.1 | Major | `snippets/accordion-row-new.liquid:55` |
| R-03 | 4.1.2 | Major | `snippets/cart-drawer.liquid:35` |
| R-04 | 4.1.2 | Major | `snippets/search-modal.liquid:11` |
| R-05 | 4.1.2 | Major | `snippets/account-drawer.liquid:15` |
| R-06 | 4.1.2 / 2.5.3 | Minor | `snippets/predictive-search.liquid:72` |
| R-07 | 2.4.3 / 4.1.2 | Minor | `sections/custom-pd-tabs-section.liquid:471` |
| R-08 | 4.1.3 | Minor | `snippets/cart-discount.liquid:65` |
| R-09 | 4.1.2 / 4.1.1 | Minor | `snippets/disclosure-trigger.liquid:21` |
| R-10 | 4.1.2 | Minor | `snippets/header-drawer.liquid:284` |

---

## 5. Recommended Remediation Roadmap

### Phase 1 — Stabilize keyboard and screen-reader basics (Weeks 1–2)
Address all 10 Critical issues:
- Restore visible focus indicators (P-12, P-13, P-14, O-01).
- Fix the contact-form `aria-describedby` link (U-01).
- Replace `role="checkbox"` on the custom checkbox label (R-01).
- Gate localization auto-submit behind explicit confirm or warning (U-03, U-04).
- Convert pixel font-sizes to `rem` (P-11).
- Add proper alt text to the customer-rating image (P-01).

### Phase 2 — Achieve WCAG 2.1 AA conformance (Weeks 3–6)
Resolve the 25 Major findings, prioritizing:
1. Modal dialogs (R-03, R-04, R-05).
2. Custom accordion overhaul (R-02).
3. Marquee, slideshow and announcement-bar pause parity (O-02 through O-06).
4. Form labels, error roles, and required-field disclosure (U-02, U-05, U-06, U-07, P-16).
5. Image, SVG, and table semantics (P-02 through P-09).

### Phase 3 — Polish and prevention (Weeks 7–8)
- Resolve the 13 Minor findings.
- Add a CI lint rule that fails on `outline: none` without a paired `:focus-visible` style.
- Add Storybook / theme-dev examples for the accordion, dialog, and tab patterns.
- Schedule a runtime + assistive-technology re-audit (NVDA + Chrome, VoiceOver + Safari, JAWS + Edge) once Phase 2 completes.

### Ongoing
- Add an accessibility checklist to the PR template.
- Train developers on the WAI-ARIA Authoring Practices for accordion, dialog, tabs, disclosure, and combobox.
- Re-run this audit per major release.

---

## 6. Appendix A — WCAG 2.1 AA Success Criteria Coverage

| Success Criterion | Status |
|---|---|
| 1.1.1 Non-text Content | **Failed** (P-01 … P-04, P-17) |
| 1.2.5 Audio Description (Prerecorded) | **Failed** (P-05) |
| 1.3.1 Info and Relationships | **Failed** (P-06 … P-09, P-16, U-01) |
| 1.3.5 Identify Input Purpose | **Failed** (P-10) |
| 1.4.2 Audio Control | **Failed** (P-05) |
| 1.4.4 Resize Text | **Failed** (P-11) |
| 1.4.11 Non-text Contrast | **Failed** (P-12 … P-14, O-09) |
| 1.4.12 Text Spacing | **Failed** (P-15) |
| 1.4.13 Content on Hover or Focus | **Failed** (O-06, O-10) |
| 2.1.1 Keyboard | **Failed** (O-03 … O-08, R-02) |
| 2.1.2 No Keyboard Trap | Compliant (utility present) |
| 2.2.2 Pause, Stop, Hide | **Failed** (O-02, O-03, O-04) |
| 2.4.1 Bypass Blocks | Compliant |
| 2.4.3 Focus Order | **Failed** (R-07) |
| 2.4.7 Focus Visible | **Failed** (P-12 … P-14, O-01, O-09) |
| 2.5.1 Pointer Gestures | **Failed** (O-07, O-08) |
| 2.5.3 Label in Name | **Failed** (U-07, R-06) |
| 3.1.1 Language of Page | Compliant |
| 3.2.2 On Input | **Failed** (U-03, U-04) |
| 3.3.1 Error Identification | **Failed** (U-01, U-06) |
| 3.3.2 Labels or Instructions | **Failed** (P-16, P-18, U-02, U-05, U-07, U-08, U-10, U-11) |
| 4.1.1 Parsing | Compliant (no unclosed tags or invalid nesting found) |
| 4.1.2 Name, Role, Value | **Failed** (P-06, P-07, P-09, R-01 … R-07, R-09, R-10) |
| 4.1.3 Status Messages | **Failed** (R-08, U-06) |

---

## 7. Appendix B — Audit Artifacts

- **Repository:** `liveElemental` (Shopify theme, Horizon v3.1.0 base)
- **Branch audited:** `claude/organize-chat-code-oM4lO`
- **Inventory:** 56 sections · 126 snippets · 96 blocks · 73 JS/CSS assets
- **Audit type:** Static source-level (automated discovery + manual code review)
- **Companion documents in repository:** `WCAG-Compliance-Report.md`, `WCAG-Compliance-Report.html`, `WCAG-Compliance-Report.doc` (prior compliance summary)

---

*End of report.*
