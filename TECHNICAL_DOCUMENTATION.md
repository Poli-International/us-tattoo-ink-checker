# US Tattoo Ink Checker - Technical Documentation

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Data Schemas](#data-schemas)
3. [Calculation / Logic Algorithms](#calculation--logic-algorithms)
4. [API Reference](#api-reference)
5. [Integration Guide](#integration-guide)
6. [Customization](#customization)
7. [Performance](#performance)
8. [Browser Compatibility](#browser-compatibility)
9. [Security](#security)
10. [Version History](#version-history)
11. [Support / Contact](#support--contact)

---

## Architecture Overview

### Technology Stack

The US Tattoo Ink Checker is a client-side single-page application built with:

- **HTML5** (`index.html`) as the shell document.
- **CSS** delivered as a compiled stylesheet (`assets/index-BS5laHRC.css`) plus two shared print/a11y stylesheets (`/tools/shared/print.css`, `/tools/shared/a11y.css`).
- **JavaScript (ES modules)**, bundled by Rolldown. The application entry is `assets/index-Bzh5-Tp3.js`, which imports a runtime chunk (`assets/rolldown-runtime-CbXtAM7H.js`) and a React vendor chunk (`assets/vendor-react-CyzwXRd0.js`).
- **React** (via the vendor chunk) for the UI layer.
- **localStorage** for theme persistence only (`poli-theme`).
- **No backend.** All screening runs in the browser; nothing is transmitted.

### File Structure

```
/tools/us-tattoo-ink-checker/
├── index.html                              # App shell, meta tags, theme bootstrap
├── manifest.json                           # PWA manifest reference
├── assets/
│   ├── index-Bzh5-Tp3.js                   # Application bundle (i18n strings, substance DB, logic)
│   ├── index-BS5laHRC.css                  # Compiled application styles
│   ├── rolldown-runtime-CbXtAM7H.js        # Module preload / CJS interop runtime
│   └── vendor-react-CyzwXRd0.js            # React vendor bundle
├── documentation.html                      # English user guide
├── documentation-fr.html                   # French user guide
├── documentation-de.html                   # German user guide
├── documentation-es.html                   # Spanish user guide
├── documentation-it.html                   # Italian user guide
├── documentation-nl.html                   # Dutch user guide
└── documentation-pt.html                   # Portuguese user guide
```

Shared assets loaded from the site root:

- `/js/input-guards.js` (loaded synchronously before the module bundle)
- `/tools/shared/print.css` (media="print")
- `/tools/shared/a11y.css`

### Component / Logic Breakdown

The application bundle contains three logical layers:

1. **Module preload polyfill** (top of `index-Bzh5-Tp3.js`): a standard Vite/Rolldown `modulepreload` shim that fetches `<link rel="modulepreload">` targets when the browser lacks native support.
2. **Internationalization layer**: a `languages` array (`o`) of seven locale descriptors and a translation map (`s`) keyed by locale code. Each locale object holds flat dotted keys such as `app.title`, `input.label`, `mocra.pillar_reg_title`, and per-substance keys such as `substance.lead.name`.
3. **Substance reference database**: a curated set of 15 substances/classes, each with MoCRA status, federal citation, notes, Prop 65 listing date, endpoints, NSRL, MADL, and OEHHA citation. The UI screens pasted text against this set and renders summary counts, a MoCRA detail table, a Prop 65 detail table, and a filterable curated list.

The `index.html` shell also contains an inline theme bootstrap script that reads `poli-theme` from localStorage and, when embedded in an iframe, listens for `postMessage` events of type `poli-theme` to sync the theme with the parent.

---

## Data Schemas

### Locale Descriptor Array

Defined in the bundle as `o`:

```js
[
  { code: "en", label: "English" },
  { code: "fr", label: "Français" },
  { code: "it", label: "Italiano" },
  { code: "de", label: "Deutsch" },
  { code: "es", label: "Español" },
  { code: "nl", label: "Nederlands" },
  { code: "pt", label: "Português" }
]
```

| Field | Type | Description |
|-------|------|-------------|
| `code` | string | ISO 639-1 locale code used as the translation map key |
| `label` | string | Display name shown in the interface language selector |

### Translation Map

Defined as `s`, keyed by locale code (`en`, `fr`, `it`, `de`, `es`, `nl`, `pt`). Each value is a flat object of dotted string keys. Representative keys:

| Key | Example value (en) |
|-----|--------------------|
| `app.title` | `US Tattoo Ink Regulation Checker` |
| `app.subtitle` | `MoCRA Federal Requirements & California Proposition 65 Screening` |
| `nav.screener` | `Ingredient Screening` |
| `nav.mocra_section` | `MoCRA (Federal Law)` |
| `nav.prop65_section` | `Proposition 65 (California)` |
| `nav.curated_list` | `Curated Substance List` |
| `nav.print` | `Print Screening Report` |
| `input.label` | `Paste Safety Data Sheet (SDS) Section 3, Bottle Labeling, or Ingredient Text` |
| `input.placeholder` | `Paste ingredients, chemical names, or CAS numbers (e.g. CAS 1333-86-4, CI 77891, Titanium dioxide, Lead, Nickel, Toluene)...` |
| `input.btn_screen` | `Screen Ingredient Text` |
| `input.btn_clear` | `Clear Input` |
| `input.error_empty` | `Please enter or paste ingredient text, chemical names, or CAS numbers to screen.` |
| `input.char_count` | `{count} characters entered` |
| `results.matched_total` | `Matched in Submitted Text: {count} of {total} ({percent}%)` |
| `results.matched_mocra` | `Subject to Federal MoCRA Framework: {count}` |
| `results.matched_prop65` | `Listed under California Proposition 65: {count}` |
| `confidence.exact_cas` | `Exact CAS Number Match` |
| `confidence.exact_ci` | `Color Index (CI) Match` |
| `confidence.word_boundary_name` | `Word-Boundary Name Match` |
| `confidence.token_match` | `Tokenized Match` |

Placeholders use single-brace syntax: `{count}`, `{total}`, `{percent}`, `{date}`, `{year}`.

### Substance Record

Each of the 15 curated substances is stored as a set of translation keys under `substance.<id>.*`. The identifiers observed in the bundle are:

`carbon_black`, `titanium_dioxide`, `lead`, `arsenic`, `cadmium`, `nickel`, `cobalt`, `hexavalent_chromium`, `formaldehyde`, `toluene`, `benzene`, `ethylene_oxide`, `ppd`, `antimony`, `barium`.

Per-substance fields:

| Field key | Type | Example (`substance.lead`) |
|-----------|------|----------------------------|
| `name` | string | `Lead and Lead Compounds` |
| `mocra_status` | string | `Heavy metal impurity. FDA draft guidance for cosmetics recommends maximum 10 ppm as an unavoidable impurity. Under FD&C Act § 601, inks containing harmful levels are adulterated.` |
| `federal_citation` | string | `Public Law 117-328; FDA Draft Guidance for Industry: Lead in Cosmetic Lip Products and Externally Applied Cosmetics` |
| `notes` | string | `Trace elemental contaminant in mineral and synthetic pigments. Requires raw material heavy metal testing documentation.` |
| `listing_date` | string | `October 1, 1992 (cancer); February 27, 1987 (reproductive)` |
| `endpoints` | string | `Carcinogenicity, Developmental Toxicity, Male & Female Reproductive Toxicity` |
| `nsrl` | string | `15 µg/day (oral)` |
| `madl` | string | `0.5 µg/day` |
| `prop65_citation` | string | `OEHHA Title 27 CCR § 25705 (NSRL) & § 25805 (MADL)` |

Substances without an OEHHA safe harbor use the sentinel strings `No Safe Harbor Level established by OEHHA` and `Not established`.

### Curated Scope Constants

The coverage strings document the reference set size:

- 15 chemical substances and heavy-metal classes
- 22 CAS numbers
- 10 Colour Index (CI) codes

These values appear in `coverage.desc`, `results.total_curated`, `print.curated_scope`, and `footer.disclaimer_compiled`.

---

## Calculation / Logic Algorithms

The bundle is minified; the following describes the observable behavior of the screening pipeline as reflected in the translation keys, summary strings, and confidence labels. No function names beyond the minified identifiers are recoverable from the provided source, so the algorithm is described by its stages.

### Stage 1: Input Capture

1. The user pastes text into the textarea bound to `input.label` / `input.placeholder`.
2. A live character counter renders `input.char_count` with `{count}` substituted by the current string length.
3. On submit (`input.btn_screen`), if the trimmed input is empty, the error string `input.error_empty` is displayed and screening is aborted.
4. `input.btn_clear` resets the textarea and clears results.

### Stage 2: Normalization and Matching

The submitted text is compared against the curated substance set. Four match confidence tiers are exposed in the UI via the `confidence.*` keys, which implies the following matching order:

1. **Exact CAS Number Match** (`confidence.exact_cas`): the input is scanned for CAS registry numbers (e.g. `1333-86-4`, `7440-02-0`) and compared against the CAS numbers associated with each curated substance.
2. **Color Index (CI) Match** (`confidence.exact_ci`): the input is scanned for CI codes (e.g. `77891`, `77288`) and matched against the substance's CI identifiers.
3. **Word-Boundary Name Match** (`confidence.word_boundary_name`): chemical names (e.g. `Titanium dioxide`, `Lead`, `Nickel`, `Toluene`) are matched with word-boundary semantics so partial tokens do not produce false positives.
4. **Tokenized Match** (`confidence.token_match`): a fallback tokenization pass for names that appear inside longer strings or with punctuation variation.

A substance is counted as matched if any tier produces a hit.

### Stage 3: Aggregation

After matching, the tool computes:

- `results.total_curated`: total substances in the reference database (15).
- `results.matched_total`: `{count} of {total} ({percent}%)` where `count` is the number of matched substances, `total` is 15, and `percent` is `count / total * 100`.
- `results.matched_mocra`: count of matched substances that carry a MoCRA status.
- `results.matched_prop65`: count of matched substances that appear on the Prop 65 list.

If no substance matches, `results.no_matches` and `results.no_matches_sub` are shown instead of the detail tables.

### Stage 4: Rendering

- **MoCRA detail table** (`mocra.matched_header`): one row per matched substance with columns `mocra.col_substance` (Substance & CAS), `mocra.col_fda_status` (FDA MoCRA Status & Provisions), and `mocra.col_citation` (Statutory Citation).
- **Prop 65 detail table** (`prop65.matched_header`): one row per matched substance with columns `prop65.col_substance`, `prop65.col_endpoints`, `prop65.col_safe_harbor`, `prop65.col_citation`, and `prop65.col_actions`. Safe harbor cells render `prop65.nsrl_label` / `prop65.madl_label` values, or `prop65.no_safe_harbor` when none exists.
- **Curated list table** (`table.title`): the full 15-substance reference set, filterable by jurisdiction (`table.filter_all`, `table.filter_federal`, `table.filter_prop65`) and by free-text search (`table.search_placeholder`). A `table.showing_count` string reports `{count}` of `{total}` visible rows.

### Stage 5: Print

`nav.print` / `ui.print_btn` triggers the browser print dialog. The print stylesheet (`/tools/shared/print.css`) hides navigation and controls and renders a report with `print.report_title`, `print.report_subtitle`, `print.date_label` (with `{date}`), `print.curated_scope`, and `results.print_attribution`.

---

## API Reference

The tool exposes no public JavaScript API. It is a self-contained browser application. The only externally observable interfaces are:

### `window.postMessage` (theme sync)

When the tool is embedded in an iframe (`window.self !== window.top`), it listens for messages of the following shape:

```js
{
  type: "poli-theme",
  light: true | false
}
```

Behavior: sets `document.documentElement[data-theme]` to `"light"` or `"dark"` and persists the value to `localStorage["poli-theme"]`.

### `localStorage` key

| Key | Values | Purpose |
|-----|--------|---------|
| `poli-theme` | `"light"` \| `"dark"` | Persists the user's theme choice across sessions |

### UI Handlers (internal)

| Handler | Trigger | Behavior |
|---------|---------|----------|
| Screen | `input.btn_screen` click | Validates input, runs the matching pipeline, renders summary and detail tables |
| Clear | `input.btn_clear` click | Empties the textarea and clears results |
| Print | `nav.print` / `ui.print_btn` | Invokes `window.print()` |
| Language change | `ui.language` selector | Switches the active locale in the translation map |
| Theme toggle | `ui.theme_dark` / `ui.theme_light` | Toggles `data-theme` on `<html>` and writes `poli-theme` |
| Jurisdiction filter | `table.filter_all` / `table.filter_federal` / `table.filter_prop65` | Filters the curated list table |
| Search | `table.search_placeholder` input | Filters curated substances by name, CAS, or CI |

No parameters or return values are exposed outside the bundle.

---

## Integration Guide

### Standalone Embedding

The tool is a static, dependency-free bundle (React vendor chunk is bundled locally; no CDN calls). To embed:

```html
<iframe
  src="https://poliinternational.com/tools/us-tattoo-ink-checker/"
  title="US Tattoo Ink Checker"
  width="100%"
  height="900"
  loading="lazy"
  referrerpolicy="no-referrer">
</iframe>
```

### Theme Synchronization

When embedded, the host page can push a theme into the iframe:

```js
const frame = document.querySelector('iframe');
frame.contentWindow.postMessage({ type: 'poli-theme', light: true }, '*');
```

The iframe applies `data-theme="light"` (or `"dark"`) and stores the choice in `localStorage`.

### Direct Link

Open the tool directly at:

```
https://poliinternational.com/tools/us-tattoo-ink-checker/
```

### Localization

Seven locales are bundled: `en`, `fr`, `it`, `de`, `es`, `nl`, `pt`. Language is selected in-app via the `ui.language` control; there is no URL parameter for locale in the provided source.

### Print Integration

The tool ships with a dedicated print stylesheet (`/tools/shared/print.css`) and an accessibility stylesheet (`/tools/shared/a11y.css`). Hosts that embed the tool in an iframe should allow the print dialog to open from the iframe context, or provide a "print report" affordance that calls `window.print()` on the iframe's content window.

---

## Customization

Customization is limited to what the bundle exposes:

- **Theme**: light/dark via the in-app toggle or the `poli-theme` postMessage channel. The default is `dark` (set in `<html data-theme="dark">`).
- **Language**: seven bundled locales selectable at runtime.
- **Curated list filtering**: jurisdiction filter and free-text search on the 15-substance table.

The substance reference set, MoCRA pillar text, and Prop 65 safe harbor values are compiled into the bundle and are not user-editable at runtime. Changing them requires rebuilding the bundle from source.

---

## Performance

- **Bundle size**: the application ships as four assets (one JS bundle, one CSS file, two preload chunks). The React vendor chunk is split out and preloaded via `<link rel="modulepreload">`.
- **No network calls at runtime**: all screening is local, so latency is bounded by the size of the pasted text and the 15-substance reference set.
- **Module preload polyfill**: the bundle includes a shim that fetches `modulepreload` links on browsers without native support, avoiding a waterfall on older engines.
- **Lazy iframe loading**: the integration example uses `loading="lazy"` to defer the iframe until it is near the viewport.

---

## Browser Compatibility

- **ES modules**: the bundle is loaded with `<script type="module">`, requiring a browser with ES module support (all evergreen browsers).
- **`modulepreload`**: natively supported in modern Chromium, Firefox, and Safari; the bundled polyfill covers older engines.
- **`localStorage`**: used for theme persistence; wrapped in `try/catch` in the inline bootstrap so private-mode or storage-disabled browsers degrade gracefully.
- **`postMessage`**: used only when the tool is embedded in an iframe; the listener is registered conditionally on `window.self !== window.top`.
- **Print**: relies on the standard `window.print()` API and a print media stylesheet.

---

## Security

- **No server transmission**: all input is processed in the browser. The footer disclaimer states records are "processed entirely in the browser and are never transmitted to any external server."
- **Input handling**: pasted text is matched against a fixed reference set; it is not evaluated as code. The bundle does not use `eval` or `Function` constructors on user input.
- **XSS surface**: React's default escaping applies to rendered strings. The inline theme bootstrap script only reads `localStorage` and `postMessage` payloads and writes a fixed `data-theme` attribute value (`"light"` or `"dark"`), not arbitrary HTML.
- **`postMessage` origin**: the inline listener does not validate `event.origin` in the provided source. Hosts embedding the tool should treat the theme channel as cosmetic-only and avoid sending sensitive data over it.
- **`meta robots`**: the page is marked `noindex, nofollow`, so it is not indexed by search engines.
- **`referrerpolicy`**: the integration example uses `no-referrer` to avoid leaking the embedding page URL.

---

## Version History

### 1.0.0

- Initial release of the US Tattoo Ink Checker.
- Curated reference set of 15 substances / classes covering 22 CAS numbers and 10 Colour Index codes.
- Screening against Federal MoCRA (Public Law 117-328) and California Proposition 65 (Health & Safety Code § 25249.5 et seq.).
- Four match confidence tiers: exact CAS, exact CI, word-boundary name, tokenized match.
- MoCRA detail table with statutory citations and Prop 65 detail table with NSRL/MADL safe harbor values.
- Filterable curated substance list with jurisdiction filter and free-text search.
- Print report with dedicated print stylesheet.
- Seven interface languages: English, French, Italian, German, Spanish, Dutch, Portuguese.
- Light/dark theme with `localStorage` persistence and iframe `postMessage` sync.
- Fully client-side; no data leaves the browser.

---

## Support / Contact

For technical support, bug reports, or integration questions:

**Email:** support@poliinternational.com

Published by Poli International, manufacturer of BioFlex® body jewelry and developer of free technical and regulatory tools for professional body modification studios worldwide.
