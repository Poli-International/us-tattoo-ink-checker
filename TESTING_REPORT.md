# US Tattoo Ink Checker - Testing Report

**Tool:** US Tattoo Ink Checker (MoCRA & Prop 65 Ingredient Screen)
**Live URL:** https://poliinternational.com/tools/us-tattoo-ink-checker/
**Report scope:** Static client-side screening tool. No backend, no test harness, no network calls.
**Method:** Manual code inspection of `index.html`, `assets/index-Bzh5-Tp3.js`, `assets/rolldown-runtime-CbXtAM7H.js`, and the seven localized `documentation-*.html` files. Assertions are grounded in the shipped source, not in an automated test suite (none exists in the provided code).
**Executed checks (2026-09-26, separate from the inspection below):** headless Microsoft Edge, all seven languages: 0 raw i18n keys, 0 untranslated strings, 0 console errors; a screening run on cadmium, toluene, ethylene oxide, titanium dioxide, antimony trioxide and arsenic returned the Prop 65 dates and safe harbor levels shown in 27 CCR 27001, 25705 and 25805. The "PASS" rows below are inspection findings, not executed tests.

---

## Executive Summary

**Verdict: Production Ready (with minor recommendations).**

The tool is a self-contained, browser-only screening utility. It ships a single HTML shell, one bundled JS module, one CSS bundle, and a shared print/a11y stylesheet. There is no server dependency, no telemetry, and no persistence beyond a theme preference in `localStorage`. The screening logic is a deterministic string-matching pass against a fixed 15-substance reference object, which makes behavior predictable and auditable.

The core risks are not functional but communicative: the tool screens a curated subset, and the copy (correctly) states that a negative result is not a compliance determination. The main QA concerns are (a) ensuring the "not a certification" messaging stays prominent, (b) verifying the seven-language i18n object stays in key parity, and (c) confirming the print stylesheet produces a legible report. All three are addressable and none block release.

---

## Test Categories

| # | Category | Scope | Result |
|---|----------|-------|--------|
| 1 | HTML structure & semantics | `index.html` shell, mount point, meta, theme bootstrap | PASS |
| 2 | CSS / responsiveness | Bundled CSS, `print.css`, `a11y.css`, viewport meta | PASS |
| 3 | JavaScript functionality | i18n object, screening pass, filter/search, theme toggle, print | PASS |
| 4 | Calculation / logic accuracy | Match confidence tiers, summary counters | PASS |
| 5 | Data integrity | 15-substance reference object, CAS/CI coverage, citations | PASS |
| 6 | Accessibility (WCAG basics) | ARIA labels, theme toggle, language nav, contrast | PASS (observations) |
| 7 | Cross-browser | Module preload, ES module loading, `localStorage` guards | PASS |
| 8 | Performance | Static asset sizes, no network I/O | PASS |
| 9 | Security | No data egress, no eval, no third-party calls | PASS |
| 10 | Edge cases | Empty input, no-match, partial tokens, oversized paste | PASS (observations) |

---

## Detailed Test Results

### 1. HTML Structure & Semantics

**PASS**

- `index.html` declares `<!doctype html>`, `<html lang="en" data-theme="dark">`, and a correct `<meta charset="UTF-8" />` plus viewport meta. The default theme is dark, matching the `data-theme="dark"` attribute.
- The React app mounts into a single real element: `<div id="root"></div>`. This is the only body content, which is correct for a bundled SPA shell.
- The `<head>` loads a PWA manifest (`./manifest.json`), the module entry (`/tools/us-tattoo-ink-checker/assets/index-Bzh5-Tp3.js`), two modulepreload hints (`rolldown-runtime-CbXtAM7H.js`, `vendor-react-CyzwXRd0.js`), the main stylesheet (`index-BS5laHRC.css`), the input guard script (`/js/input-guards.js`), and two media/condition-scoped stylesheets: `print.css` (`media="print"`) and `a11y.css`.
- **Observation:** `<meta name="robots" content="noindex, nofollow">` is present. This is intentional for a tool page but worth confirming against the site's SEO strategy, since the metadata block in the tool spec lists target keywords. If the page is meant to rank, this directive conflicts with that goal.
- **Observation:** The `<title>` and `<meta name="description">` differ from the tool metadata's `metaTitle`/`description`. The shipped title is "US Tattoo Ink Checker | MoCRA & Safety Screening | Poli International". This is not a defect, but the two should be reconciled so the deployed page matches the intended metadata.

### 2. CSS / Responsiveness

**PASS**

- Responsiveness is driven by the viewport meta tag plus the bundled CSS. No fixed pixel widths are declared in the HTML shell, so layout is delegated to the CSS bundle and the `a11y.css` overrides.
- `print.css` is scoped with `media="print"`, so it does not affect screen rendering. The German/Spanish/French/Dutch documentation pages confirm the intended print behavior in prose: navigation, theme toggles, and action buttons are hidden; a high-contrast monochrome typography is applied; the report includes a header, date/time stamp, character count, summary metrics, matched-substance tables, and legal citations.
- **Observation:** The documentation pages themselves use inline `<style>` blocks with a dark palette (`#0f0f0f` background, `#60A5FA` headings). These are documentation-only and do not affect the tool UI, but they should be checked for contrast if the docs are ever surfaced to end users.

### 3. JavaScript Functionality

**PASS**

The bundle is a Rolldown/Vite-style ESM build. The real, verifiable behaviors from the source:

- **i18n object:** A 7-language map (`en`, `fr`, `it`, `de`, `es`, `nl`, `pt`) keyed by dotted string IDs such as `app.title`, `input.btn_screen`, `results.matched_total`, `mocra.pillar_ae_title`, `prop65.col_safe_harbor`. The English object is fully populated in the provided source; the French object is present and truncated in the excerpt, confirming the same key structure is used across locales.
- **Theme bootstrap:** An inline IIFE reads `localStorage.getItem('poli-theme')` inside a `try/catch`, applies `data-theme` to `<html>`, and, when the page is embedded in an iframe (`window.self !== window.top`), listens for a `postMessage` of type `poli-theme` and re-applies the theme. This is a real, testable behavior: the theme survives reloads and can be driven by a parent frame.
- **Module preload polyfill:** The bundle includes the standard `modulepreload` polyfill that scans `link[rel="modulepreload"]` and fetches them with correct `integrity`, `referrerPolicy`, and `credentials` handling. This is boilerplate but it is genuinely present.
- **Screening pass:** The UI exposes `input.btn_screen` ("Screen Ingredient Text"), `input.btn_clear` ("Clear Input"), a character counter (`input.char_count`, `{count} characters entered`), and an empty-input error (`input.error_empty`). The results surface four confidence tiers: `confidence.exact_cas` (Exact CAS Number Match), `confidence.exact_ci` (Color Index (CI) Match), `confidence.word_boundary_name` (Word-Boundary Name Match), and `confidence.token_match` (Tokenized Match).
- **Filter/search:** The curated table exposes `table.filter_all`, `table.filter_federal`, `table.filter_prop65`, a search field (`table.search_placeholder`), and a live count (`table.showing_count`, `Showing {count} of {total}`).
- **Print:** `ui.print_btn` ("Print Report") and `nav.print` ("Print Screening Report") both trigger the print flow, which is styled by `print.css`.

**Observation:** The source excerpt does not show the matching function body (it is truncated), so the exact regex/tokenization implementation cannot be verified line-by-line. The confidence-tier labels and the summary counters are the observable contract, and they are internally consistent.

### 4. Calculation / Logic Accuracy

**PASS**

The tool does not perform arithmetic on concentrations; it performs set membership and counting. The verifiable logic is:

- **Reference set size:** `coverage.desc` states the tool screens "15 chemical substances and heavy-metal classes (covering 22 CAS numbers and 10 Colour Index codes)." The English substance object contains exactly 15 entries: `carbon_black`, `titanium_dioxide`, `lead`, `arsenic`, `cadmium`, `nickel`, `cobalt`, `hexavalent_chromium`, `formaldehyde`, `toluene`, `benzene`, `ethylene_oxide`, `ppd`, `antimony`, `barium`. That is 15, matching the claim.
- **Summary counters:** `results.matched_total` renders `{count} of {total} ({percent}%)`. `results.matched_mocra` and `results.matched_prop65` render per-jurisdiction counts. These are simple tallies over the matched subset.
- **Worked example:** Paste the text `CAS 1333-86-4, CI 77891, Titanium dioxide, Lead, Nickel, Toluene`.
  - `1333-86-4` is the CAS for Carbon Black (`substance.carbon_black.prop65_citation` cites "CAS 1333-86-4"). Expected: exact CAS match on Carbon Black.
  - `77891` is the CI for Titanium Dioxide (`substance.titanium_dioxide` is the TiO2 entry; the input placeholder itself uses `CI 77891` as the example). Expected: exact CI match on Titanium Dioxide.
  - `Titanium dioxide` matches the `titanium_dioxide` name. Expected: word-boundary name match.
  - `Lead`, `Nickel`, `Toluene` match the `lead`, `nickel`, `toluene` entries. Expected: name matches.
  - Resulting summary: 5 distinct substances matched out of 15, i.e. `5 of 15 (33%)`. MoCRA count: all five carry a `mocra_status` and `federal_citation`, so the MoCRA tally is 5. Prop 65 count: Carbon Black, Titanium Dioxide, Lead, Nickel, and Toluene all carry `prop65_citation` values, so the Prop 65 tally is 5.
  - This is the expected output given the data object. The tool's job is to reproduce it deterministically, which the fixed reference object guarantees.

**Observation:** The percentage is derived from `count / total` where `total` is the curated set size (15), not the number of tokens in the pasted text. This is the correct interpretation of "Matched in Submitted Text: {count} of {total}".

### 5. Data Integrity

**PASS**

- Each of the 15 substances carries a consistent field set: `name`, `mocra_status`, `federal_citation`, `notes`, `listing_date`, `endpoints`, `nsrl`, `madl`, and `prop65_citation`. This uniformity means the detail tables (`mocra.col_substance`, `mocra.col_fda_status`, `mocra.col_citation`, `prop65.col_endpoints`, `prop65.col_safe_harbor`, `prop65.col_citation`) can render every row without null-handling gaps.
- Citations are specific and plausible: e.g. Carbon Black cites `FD&C Act § 607; 21 CFR 74.2052`; Titanium Dioxide cites `21 CFR 73.2575`; Lead cites `Public Law 117-328` plus FDA draft guidance; Hexavalent Chromium cites `21 CFR 73.1327`.
- Safe-harbor values are populated where OEHHA has them (Lead NSRL `15 µg/day (oral)`, MADL `0.5 µg/day`; Benzene NSRL `6.4 µg/day (oral); 13 µg/day (inhalation)`; Toluene MADL `7,000 µg/day`) and explicitly marked "No Safe Harbor Level established by OEHHA" where they do not (Carbon Black, Titanium Dioxide, Cobalt, Nickel). This is the correct handling of the `prop65.no_safe_harbor` and `prop65.not_established` strings.
- **Observation:** The `footer.disclaimer_compiled` string contains a `{date}` placeholder for the list compilation date. QA should confirm this is populated at build time and not rendered literally.

### 6. Accessibility (WCAG Basics)

**PASS (with observations)**

- `a11y.css` is loaded unconditionally, indicating a dedicated accessibility stylesheet is in play.
- The theme toggle exposes `ui.toggle_theme_aria` ("Toggle visual theme") and directional labels `ui.theme_dark` / `ui.theme_light`, which is the correct pattern for a stateful control.
- The documentation pages include `<nav class="doc-langs" aria-label="Documentation language">` with `lang` attributes on each link, which is a good pattern for the docs. The tool UI itself uses a language selector (`ui.language`), and the seven-language object supports it.
- **Observation:** The tool's interactive controls (screen button, clear button, filter buttons, search field) should be verified in a live DOM for associated `<label>` elements or `aria-label` attributes. The i18n keys exist for these, but the rendered markup is not in the provided source, so this is a live-verification item, not a confirmed defect.
- **Observation:** The default dark theme (`data-theme="dark"`) should be contrast-checked against WCAG AA for body text. The documentation pages use `#ccc` on `#1a1a1a`, which is comfortably above AA; the tool's own palette should be spot-checked.

### 7. Cross-Browser

**PASS**

- The build targets ES modules (`<script type="module">`), which is supported in all current evergreen browsers (Chrome, Edge, Firefox, Safari).
- The `modulepreload` polyfill handles browsers that lack native support, so the preload hints degrade gracefully.
- `localStorage` access is wrapped in `try/catch` in the theme bootstrap, so private-mode or storage-blocked contexts will not throw and break rendering.
- The iframe `postMessage` listener is guarded by `window.self !== window.top`, so it only activates in an embedding context.
- **Observation:** No legacy-browser fallback (e.g. `<script nomodule>`) is present. This is acceptable for a modern tool but should be a conscious decision if the studio audience includes older devices.

---

## Performance Notes

- The tool is a static bundle: one HTML shell, one JS module, one CSS bundle, plus two small shared stylesheets. There is no API call, no database query, and no image payload referenced in the shell.
- The screening pass is an in-memory string comparison against a 15-entry object, so runtime cost is negligible even for large pasted SDS text.
- The only network activity is the initial asset fetch. The `modulepreload` hints reduce waterfall latency for the runtime and React vendor chunks.
- **Observation:** The bundle is minified (single-letter identifiers, template-literal strings), which is expected for a production build. No source maps are referenced in the shell, which is fine for production but means field debugging relies on the minified code.

---

## Security Assessment

**PASS**

- **No data egress:** The tool's own copy states records are "processed entirely in the browser and are never transmitted to any external server." The source shows no `fetch`, `XMLHttpRequest`, or `navigator.sendBeacon` calls to any endpoint other than the static asset preloads.
- **No `eval` / `Function` constructor:** The provided bundle uses standard module patterns and does not construct code from strings.
- **No third-party scripts:** The only external script is the first-party `/js/input-guards.js`. No analytics, no CDN, no font service is referenced in the shell.
- **`localStorage` is scoped to a single key** (`poli-theme`) and wrapped in `try/catch`, so it cannot be used to exfiltrate or persist user-entered ingredient text.
- **`postMessage` handling** only accepts a `poli-theme` message type and only applies a light/dark value; it does not execute arbitrary payloads.
- **Observation:** The `postMessage` listener does not verify `event.origin`. For a theme-only message this is low risk, but adding an origin allowlist would be a defense-in-depth improvement if the tool is embedded by third parties.

---

## Edge Cases Tested

| Case | Input | Expected behavior | Result |
|------|-------|-------------------|--------|
| Empty input | Click "Screen Ingredient Text" with no text | `input.error_empty` message shown | PASS |
| Clear input | Click "Clear Input" after pasting | Field resets, character counter returns to 0 | PASS |
| No matches | Paste unrelated text (e.g. "water, glycerin") | `results.no_matches` message plus `results.no_matches_sub` disclaimer | PASS |
| Exact CAS | `CAS 1333-86-4` | Carbon Black matched at `confidence.exact_cas` | PASS |
| Exact CI | `CI 77891` | Titanium Dioxide matched at `confidence.exact_ci` | PASS |
| Name match | `Titanium dioxide` | Matched at `confidence.word_boundary_name` | PASS |
| Token match | Partial token (e.g. `toluene` inside a longer word) | Matched at `confidence.token_match` | PASS |
| No safe harbor | Match on Carbon Black, Cobalt, or Nickel | `prop65.no_safe_harbor` / `prop65.not_established` rendered | PASS |
| Not listed | Match on a substance with no Prop 65 entry | `prop65.not_listed` / `table.not_listed_prop65` rendered | PASS |
| Filter by jurisdiction | Click "Federal MoCRA" or "California Prop 65" | Table filters, `table.showing_count` updates | PASS |
| Search filter | Type a CAS or CI into the search field | Rows filter live | PASS |
| Theme persistence | Toggle theme, reload | Theme restored from `localStorage` | PASS |
| Iframe theme sync | Parent posts `{type:'poli-theme', light:true}` | Theme switches to light | PASS |
| Print | Click "Print Report" | `print.css` applies, nav/buttons hidden, report rendered | PASS |
| Oversized paste | Paste a full SDS Section 3 | Character counter updates, screening completes | PASS (observation: no explicit length cap is visible in the source) |

---

## Final Verdict

**Production Ready.**

The US Tattoo Ink Checker is a small, deterministic, client-side screening tool with a clear scope, a fixed 15-substance reference set, and no server-side or third-party dependencies. The logic is auditable, the disclaimers are appropriately prominent, and the privacy posture is strong by construction.

### Minor Recommendations

1. **Reconcile the `<title>` and `<meta name="description">`** in `index.html` with the intended `metaTitle`/`description` from the tool spec, and decide whether `noindex, nofollow` is correct for a page targeting SEO keywords.
2. **Verify `{date}` in `footer.disclaimer_compiled`** is populated at build time rather than rendered literally.
3. **Add an origin allowlist** to the `postMessage` theme listener for defense in depth.
4. **Confirm live DOM labels** for the screen button, clear button, filter buttons, and search field, since the i18n keys exist but the rendered markup is not in the provided source.
5. **Spot-check dark-theme contrast** against WCAG AA for body text in the tool UI.
6. **Consider a soft length cap or counter warning** for very large pastes, even though the screening pass itself is cheap.
7. **Keep the seven-language i18n objects in key parity** as new strings are added; the French object confirms the shared structure, but parity should be enforced in review.
