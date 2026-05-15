# P&RE Editorial Design System
*Extracted from `ai-eval-engg.html`, `ai-eval-metrics.html`, and `ai-obs-tools.html` — a portable spec for reuse on other long-form pieces.*

---

## 1 · Foundations

### 1.1 Typography (three fonts, no more)

| Role          | Family                          | Weights    | Use                                       |
| ------------- | ------------------------------- | ---------- | ----------------------------------------- |
| Display/serif | **Instrument Serif** (Google)   | 400, 400i  | H1, H2, ordinals, pull-quotes, italicised emphasis |
| Body          | **Inter**                       | 300–700    | All running text, UI, H3/H4, nav, buttons |
| Mono          | **JetBrains Mono**              | 400, 500   | Metrics, micro-meta, key/value tables     |

Load all three from Google Fonts in one `<link>`:
```html
<link href="https://fonts.googleapis.com/css2?family=Instrument+Serif:ital@0;1&family=Inter:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet">
```

**Type scale** — fluid where it matters:
- `h1.display` → `clamp(48px, 7vw, 92px)`, line-height `0.98`, tracking `-0.02em`
- `h2.section-title` → `clamp(36px, 4.5vw, 56px)`, line-height `1.05`
- `h3` → `22px / 1.3`, weight 600, tracking `-0.01em`
- `h4` → `16px / 1.4`, weight 600
- Body `p` → `16.5px / 1.6`, max-width `760px`
- `p.lead` → `22px / 1.45`, max-width `880px`

**Italic = accent.** Inside serif headlines and lead paragraphs, `<em>` switches to italic Instrument Serif **and** changes colour to `--accent`. This is the signature move of the system.

### 1.2 Body weight depends on mode
- Light mode body: weight **400**
- Dark mode body: weight **500** (medium — for readability on dark)

Implement via `--body-weight` CSS variable, not a hard-coded value.

### 1.3 Colour tokens (Accenture palette)

```css
:root {
  --purple-50:#F5EBFE; --purple-100:#E7D2FD; --purple-200:#C9A5FA;
  --purple-300:#B181F7; --purple-400:#A100FF; --purple-500:#7A00C2;
  --purple-600:#5A1F95; --purple-700:#460073; --purple-800:#2D0049; --purple-900:#15002A;
  --black:#000; --grey-900:#1A1A1A; --grey-800:#2B2B2B; --grey-700:#4A4A4A;
  --grey-500:#767676; --grey-300:#C4C4C4; --grey-200:#E1E1E1;
  --grey-100:#EEEEEE; --grey-50:#F7F7F7; --white:#FFF;
  --pink:#E84C8A; --blue:#2E5BFF; --aqua:#00E5C7;
}
```

**Semantic tokens** — always reference these, never the raw palette:

| Token              | Light                          | Dark                          |
| ------------------ | ------------------------------ | ----------------------------- |
| `--bg`             | `white`                        | `#0A0A0A`                     |
| `--bg-alt`         | `grey-50`                      | `#131113`                     |
| `--bg-elevated`    | `white`                        | `#1A171C`                     |
| `--surface-border` | `grey-200`                     | `#2A252E`                     |
| `--ink`            | `grey-900`                     | `#F2EEF5`                     |
| `--ink-muted`      | `grey-700`                     | `#C9C3D0`                     |
| `--ink-subtle`     | `grey-500`                     | `#8E8794`                     |
| `--accent`         | `purple-700`                   | `purple-200`                  |
| `--accent-soft`    | `purple-400`                   | `purple-300`                  |
| `--accent-bg`      | `purple-50`                    | `#1F1525`                     |
| `--accent-line`    | `purple-200`                   | `#3D2A52`                     |
| `--secondary-1`    | `pink`                         | `pink`                        |
| `--secondary-2`    | `blue` (light only)            | `aqua` (dark only)            |
| `--rule`           | `1px solid grey-200`           | `1px solid #2A252E`           |

**Rules** — pulled from Accenture guidance:
- Light: dark purples for accent, lighter palette for surfaces. Dark: invert.
- Pink + Blue allowed in light; Pink + Aqua allowed in dark. **No aqua in light. No blue in dark.**
- Secondary colours only in infographics, never in text, never as a purple substitute.
- Don't crowd a page with multiple purple shades — one accent purple plus its tints/bg is enough.

**Signal-state tokens** for data tables (badges, scores, status indicators) — introduce only when needed:

| Token                 | Light                          | Dark                          | Use                          |
| --------------------- | ------------------------------ | ----------------------------- | ---------------------------- |
| `--state-positive-bg` | `rgba(46,91,255,0.08)` (blue)  | `rgba(0,229,199,0.12)` (aqua) | "Yes", high score, supported |
| `--state-positive`    | `--blue`                       | `--aqua`                      | foreground for above         |
| `--state-warn-bg`     | `rgba(232,76,138,0.08)` (pink) | `rgba(232,76,138,0.12)`       | "Partial", mid-tier          |
| `--state-warn`        | `--pink`                       | `--pink`                      | foreground for above         |
| `--state-neutral-bg`  | `rgba(0,0,0,0.04)`             | `rgba(255,255,255,0.06)`      | "No", absent, n/a            |
| `--state-neutral`     | `--ink-subtle`                 | `--ink-subtle`                | foreground for above         |

These are the *only* place secondary colours appear in the system — purely as infographic state, never on running text or as decoration.

### 1.4 Theme switching
Drive everything off `<html data-theme="light|dark">`. Toggle button persists choice to `localStorage` and respects `prefers-color-scheme` on first load. Add a 0.25s ease transition on `background` and `color` for the body.

---

## 2 · Layout shell

```
nav.top  (sticky, blurred, full-bleed, 1px bottom rule)
main     (max-width: 1280px; padding: 0 48px)
  section  (padding: 96px 0; bottom rule, except last)
footer   (full-bleed; bg-alt; centred; 56px padding)
```

- One column. Editorial. No sidebars at full width except the `grid-2` reading-room pattern (see §3.3).
- Section rhythm: **96px vertical** between sections at desktop, **64px** under 980px.
- Hero gets `120px 0 80px` padding — slightly more breathing room than other sections.
- Smooth scroll on `<html>`; section IDs match nav anchors.

---

## 3 · Components (the working set)

Each component is self-contained, uses semantic tokens, and degrades to a single column under 980px.

### 3.1 Eyebrow + display headline
The opener for every section.
```html
<div class="eyebrow">Section Label · Optional Date</div>
<h2 class="section-title">A serif headline with <em>one italic phrase</em>.</h2>
```
- Eyebrow: `12px`, `letter-spacing: 0.18em`, uppercase, `--accent`, prefixed with a 24px horizontal rule (`::before`).
- Italic span inside the headline = the single point of colour. One `<em>` per headline is plenty.

### 3.2 Hero
- Eyebrow → `h1.display` → `p.lead` → `.hero-meta` row.
- `.hero-meta`: flex, gap 36px, wraps. Each item is `<div><span>LABEL</span><strong>Value</strong></div>` — uppercase label, body-cased value.

### 3.3 `grid-2` — reading room with sticky aside
```
grid-template-columns: 240px 1fr; gap: 64px;
aside { position: sticky; top: 110px; }
```
For deep-read sections that need a pinned context column (case citations, definitions, marginalia). Collapses to single column under 980px.

### 3.4 `tldr-grid` — 4-up summary cards
Four claims, equal weight. Implemented as a CSS grid where the **gap is `1px` of `--surface-border`** showing through — no individual borders needed.
```
display: grid;
grid-template-columns: repeat(4, 1fr);
gap: 1px;
background: var(--surface-border);
border: var(--rule);
```
Each card: large serif italic ordinal (`52px`, `--accent`), then `h4`, then a 14px paragraph.

### 3.5 `pillars` — numbered deep-dive rows
For five-or-fewer surfaces, each with body copy + metric panel on the right.
```
display: grid; grid-template-columns: 80px 1fr 380px; gap: 48px;
border-top: 1px solid grey-200; padding: 48px 0;
.ord  // 48px italic serif numeral in --accent
.body // tag + h3 + paragraph
.metrics // bg-alt panel with k/v list, each row a border-bottom
```

### 3.6 `lifecycle` — 5-up process stages
Equal-width tiles in a 5-column grid, `gap: 12px`. Each stage:
- `.step` uppercase micro-label in `--accent`
- `h4` title, `p` description
- `.metric` mono footer row with KPI/measurement, separated by a 1px top rule

Hover: `border-color: --accent` + `translateY(-2px)`.

### 3.7 `compare` — old vs new table
3-column grid wrapped in a single border. Header row uses `--bg-alt`. Columns: `label | old | new`. The "new" column is set in `--ink` weight 500; "old" in `--ink-muted`. Stacks to single column under 600px.

### 3.8 `roadmap` — 3-column horizons
Three side-by-side panels, internal dividers only (`border-right`, none on the last). Each horizon:
- `.h-label` italic serif (Horizon 1 / 2 / 3)
- `.h-range` uppercase micro-text (months)
- `h4` title
- `<ul>` of items, each prefixed with `→` glyph in `--accent` via `::before`

### 3.9 `keyidea` — pull-quote
Full-bleed within the section. Top + bottom rules. Centred italic serif blockquote at `clamp(32px, 4vw, 44px)`. Cite line in uppercase Inter at `13px`.

### 3.10 `incident` / `callout` — accent-bg note
Left-bordered block (`border-left: 3px solid --accent`), tinted background (`--accent-bg`), 36px/40px padding, max-width 880px. Uppercase label on top, then content. Use sparingly — one or two per article.

### 3.11 `diagram-wrap` — figure container
`--bg-alt` background, `1px --surface-border` border, 48px padding. Uppercase 12px caption above the diagram. SVG inside.

### 3.12 `refs-grid` — references
Two columns at desktop, one column under 980px. Each `.ref-section` has:
- Accent uppercase heading with a `border-bottom: 1px solid --accent-line`
- Ordered list, 13.5px items, each with a 1px row divider
- `.src` sub-line in `--ink-subtle`, 11.5px

### 3.13 `pov-points` — numbered argument list
Counter-based numbering via CSS `counter()`. Each row is a 2-column grid (`80px 1fr`), with a `decimal-leading-zero` italic serif numeral generated by `::before`. Rows separated by 1px top rules.

### 3.14 Tables — two variants

The system has **two distinct table patterns**. Pick by purpose, not by preference.

#### 3.14a `table.metrics` — Editorial reference table
*Used in `ai-eval-metrics.html`.* For long, readable, scannable lists where each row is content (definitions, metrics, references). Generous padding, full-width text wrapping, page-level scroll.

```css
.metrics-table-wrap {
  border: 1px solid var(--surface-border);
  overflow-x: auto;
  background: var(--bg);
}
table.metrics {
  width: 100%;
  border-collapse: collapse;
  font-size: 14px;
  min-width: 980px;          /* horizontal scroll below this */
}
table.metrics thead th {
  text-align: left;
  font-weight: 600;
  font-size: 11px;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--ink-subtle);
  padding: 18px 20px;
  border-bottom: 1px solid var(--surface-border);
  background: var(--bg-alt);
  white-space: nowrap;
  position: sticky;          /* sticky under top nav */
  top: 73px;                 /* = nav height */
  z-index: 5;
}
table.metrics tbody tr {
  border-bottom: 1px solid var(--surface-border);
  transition: background 0.12s ease;
}
table.metrics tbody tr:hover { background: var(--accent-bg); }
table.metrics tbody tr:last-child { border-bottom: none; }
table.metrics td {
  padding: 18px 20px;        /* generous — read like prose */
  vertical-align: top;
  color: var(--ink);
}
table.metrics td.metric-name { font-weight: 600; white-space: nowrap; }
table.metrics td.metric-name a {
  color: var(--ink);
  text-decoration: none;
  border-bottom: 1px dotted var(--surface-border);
}
table.metrics td.metric-name a:hover {
  color: var(--accent);
  border-bottom-color: var(--accent);
}
```

**Reference chips** — small numbered links in the name cell, used to attribute sources inline:
```css
.ref-pair a {
  font-family: 'JetBrains Mono', monospace;
  font-weight: 500;
  font-size: 10px;
  padding: 2px 5px;
  border: 1px solid var(--surface-border);
  border-radius: 3px;
  color: var(--ink-muted);
  margin-left: 4px;
}
.ref-pair a:hover {
  border-color: var(--accent);
  color: var(--accent);
  background: var(--accent-bg);
}
```

**Category badges** in a leading column — small uppercase pills using `--accent-bg` + `--accent`:
```css
.cat-badge {
  display: inline-block;
  font-size: 10.5px;
  font-weight: 600;
  letter-spacing: 0.05em;
  text-transform: uppercase;
  padding: 3px 8px;
  border-radius: 3px;
  background: var(--accent-bg);
  color: var(--accent);
}
```

#### 3.14b `table.compare` — Dense comparison matrix
*Used in `ai-obs-tools.html`.* For wide, data-dense N×M tables (platforms × capabilities, tools × features) where horizontal scrolling is expected. Sticky first column, smaller type, zebra rows, expandable detail rows.

```css
.table-wrap {
  overflow: auto;
  margin: 0 24px 24px;
  border-radius: 8px;
  border: 1px solid var(--surface-border);
}
table.compare {
  border-collapse: separate;
  border-spacing: 0;
  width: max-content;
  min-width: 100%;
  font-size: 12px;            /* dense — every column visible */
}
table.compare thead th {
  padding: 10px 12px;
  text-align: left;
  font-weight: 600;
  font-size: 10.5px;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  color: var(--ink-subtle);
  background: var(--bg);
  border-bottom: 2px solid var(--accent); /* signature accent rule */
  white-space: nowrap;
}
table.compare thead th:not(:last-child),
table.compare tbody td:not(:last-child) {
  border-right: 1px solid var(--surface-border);
}

/* Sticky first column — stays visible while scrolling horizontally */
table.compare thead th.sticky,
table.compare tbody td.sticky {
  position: sticky;
  left: 0;
}
table.compare thead th.sticky { z-index: 3; background: var(--bg); }
table.compare tbody td.sticky { z-index: 2; background: var(--bg); }

table.compare tbody td {
  padding: 10px 12px;
  border-bottom: 1px solid var(--surface-border);
  vertical-align: top;
  line-height: 1.5;
  color: var(--ink);
  max-width: 280px;          /* cap cell width so wide cells wrap */
}

/* Zebra striping on data rows */
table.compare tbody tr:nth-child(even of .data-row) td {
  background: var(--bg-alt);
}
/* Hover + expanded states */
table.compare tbody tr.data-row:hover td { background: var(--accent-bg) !important; }
table.compare tbody tr.expanded td { background: var(--accent-bg) !important; }
```

**Status badges** — three-state visual: positive / partial / negative. Use the signal-state tokens from §1.3:
```css
.badge {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 2px 8px;
  border-radius: 4px;
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 0.02em;
}
.badge-dot { width: 6px; height: 6px; border-radius: 50%; }

.badge-yes      { background: var(--state-positive-bg); color: var(--state-positive); }
.badge-yes  .badge-dot { background: var(--state-positive); }
.badge-partial  { background: var(--state-warn-bg);     color: var(--state-warn); }
.badge-partial .badge-dot { background: var(--state-warn); }
.badge-no       { background: var(--state-neutral-bg);  color: var(--state-neutral); }
.badge-no  .badge-dot { background: var(--state-neutral); }
```

**Tool name cell** — first column anchors the row, serif + accent:
```css
.tool-name {
  font-weight: 700;
  color: var(--accent);
  font-size: 12.5px;
  font-family: 'Instrument Serif', Georgia, serif;
}
```

**Expandable detail row** — a hidden row revealed below the parent on click, used for long-form metadata that won't fit in cells:
```css
.detail-row td {
  padding: 0 !important;
  border-bottom: 2px solid var(--accent-line) !important;
  background: var(--accent-bg) !important;
}
.detail-grid {
  padding: 16px 20px;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  gap: 14px;
}
.detail-label {
  font-size: 10px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  color: var(--accent);
  font-family: 'Instrument Serif', Georgia, serif;
  margin-bottom: 3px;
}
.detail-val {
  font-size: 12px;
  color: var(--ink-muted);
  line-height: 1.55;
}
```

**Choosing between the two:**

| Use `table.metrics` when…                       | Use `table.compare` when…                       |
| ----------------------------------------------- | ----------------------------------------------- |
| ≤6 columns, content is mostly prose             | ≥8 columns, content is mostly cells/badges      |
| Each row reads like a definition or entry        | Each row is a record in a matrix                |
| Page-level vertical scroll                       | Container-level horizontal scroll               |
| 14px type, 18px padding                          | 12px type, 10–12px padding, dense              |
| No sticky column needed                          | First column sticky                             |
| Hover = tint the row                             | Hover = tint + click reveals detail row         |

### 3.15 Filter chips (above tables)

A horizontal row of toggleable category chips placed directly above the table. Used in both metrics and obs-tools to filter rows.

```css
.chip-row {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin: 0 0 24px;
}
.chip {
  font-family: inherit;
  font-size: 13px;
  font-weight: 500;
  padding: 6px 14px;
  border-radius: 999px;
  border: 1px solid var(--surface-border);
  background: var(--bg);
  color: var(--ink-muted);
  cursor: pointer;
  transition: all 0.15s ease;
}
.chip:hover { border-color: var(--accent); color: var(--accent); }
.chip.active {
  background: var(--ink);
  color: var(--bg);
  border-color: var(--ink);
}
.chip .count {
  font-family: 'JetBrains Mono', monospace;
  font-size: 11px;
  opacity: 0.6;
  margin-left: 4px;
}
```

The active chip inverts (`--ink` background, `--bg` text) — high contrast, intentional. Counts in mono, dimmed.

### 3.16 Scoring methodology block

A structured explanation of how a composite score is constructed, placed in the page footer or beneath the comparison table. The pattern *(adapted from `ai-obs-tools.html`)*:

```
┌────────────────────────────────────────────────────────────┐
│  H4 — "Scoring Methodology"                                │
│                                                            │
│  Formula sentence: <strong>Composite</strong> is the avg   │
│  of two dimensions Y and Z, scaled to a 10-point scale:    │
│  <strong>Overall = (Y + Z) / 2 × 2</strong>                │
│                                                            │
│  ── Dimension 1 (range) — Subtitle                         │
│     Description paragraph. Scoring bands:                  │
│     ≤A → 1, A–B → 2, … , E+ → 5.                           │
│                                                            │
│  ── Dimension 2 (range) — Subtitle                         │
│     Description paragraph. Evaluated across N criteria,    │
│     each scored 0–1 and summed:                            │
│       • Criterion 1 (0–1): …                               │
│       • Criterion 2 (0–1): …                               │
│       • …                                                  │
│                                                            │
│  ── Score Tiers                                            │
│     [positive] 8.0–10.0 — Established …                    │
│     [accent]   6.5–7.9  — Solid …                          │
│     [neutral]  <6.5     — Emerging …                       │
└────────────────────────────────────────────────────────────┘
```

**Layout pattern:**
```css
.scoring {
  padding: 32px 0;
  border-top: 1px solid var(--surface-border);
  font-size: 13px;
  color: var(--ink-muted);
  line-height: 1.7;
  max-width: 880px;
}
.scoring h4 {
  font-family: 'Instrument Serif', Georgia, serif;
  font-weight: 400;
  font-size: 24px;
  color: var(--ink);
  margin-bottom: 16px;
}
.scoring .formula {
  margin-bottom: 16px;
}
.scoring .formula strong { color: var(--ink); font-weight: 600; }
.scoring .formula .accent { color: var(--accent); font-weight: 600; }

.scoring .dimension {
  margin: 18px 0;
}
.scoring .dimension-title {
  font-size: 12px;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--accent);
  font-weight: 600;
  margin-bottom: 8px;
}
.scoring .dimension-title .range {
  font-family: 'JetBrains Mono', monospace;
  font-size: 11px;
  color: var(--ink-subtle);
  margin-left: 8px;
  letter-spacing: 0;
}

.scoring .criteria {
  padding-left: 16px;
  margin: 10px 0;
}
.scoring .criteria li {
  list-style: none;
  padding: 6px 0;
  border-top: 1px solid var(--surface-border);
}
.scoring .criteria li:first-child { border-top: none; }
.scoring .criteria .crit-name {
  color: var(--accent);
  font-weight: 600;
}

.scoring .tiers {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
  margin-top: 24px;
  padding-top: 20px;
  border-top: 1px solid var(--surface-border);
}
.scoring .tier {
  font-size: 12px;
  line-height: 1.5;
}
.scoring .tier .range {
  font-family: 'JetBrains Mono', monospace;
  font-weight: 600;
  font-size: 13px;
  display: block;
  margin-bottom: 4px;
}
.scoring .tier.high   .range { color: var(--state-positive); }
.scoring .tier.mid    .range { color: var(--accent); }
.scoring .tier.low    .range { color: var(--ink-subtle); }
```

**Rules for scoring blocks:**
1. **Always show the formula as a sentence first.** Then break it down. Readers want the punchline, then the proof.
2. **Sub-criteria belong in a bordered list, not bullets.** Each gets a 1px top rule and a mono range marker.
3. **Score tiers are three columns, three colours.** High = positive (blue/aqua), Mid = `--accent` (purple), Low = `--ink-subtle`. Never use red — this is methodology, not alarm.
4. **All bands and ranges in JetBrains Mono.** The numbers carry the credibility; mono signals "this is data."
5. **Anchor to a max-width of 880px.** Long methodology paragraphs need a comfortable reading column.

### 3.17 Maturity score visualization (in-cell)

A compact two-tier composite score display for use inside a table cell — used in `ai-obs-tools.html` to summarise the methodology output per row.

```css
.maturity-score {
  font-weight: 700;
  font-size: 15px;
  font-family: 'Instrument Serif', Georgia, serif;
}
.maturity-score.high { color: var(--state-positive); }
.maturity-score.mid  { color: var(--accent); }
.maturity-score.low  { color: var(--ink-subtle); }
.maturity-unit { font-size: 10px; color: var(--ink-subtle); }

.maturity-bars { display: flex; gap: 1.5px; margin: 4px 0; }
.maturity-bar {
  width: 8px;
  height: 4px;
  border-radius: 1.5px;
  background: rgba(0,0,0,0.06);
}
.maturity-bar.filled-high { background: var(--state-positive); }
.maturity-bar.filled-mid  { background: var(--accent); }
.maturity-bar.filled-low  { background: var(--ink-subtle); }

.maturity-sub {
  font-size: 9.5px;
  color: var(--ink-subtle);
  line-height: 1.35;
}
```

Pattern: serif numeral (`X.Y`) + tiny "/ 10" suffix + 10 micro-bars (filled count = score) + 9.5px caption underneath naming the constituent dimensions ("Age 4 / Feat 4").

---

## 4 · Recurring patterns (the rules that make it cohere)

1. **Rules over boxes.** Sections, tables, and lists are separated by `1px solid --surface-border`, not card shadows. The whole page is one continuous editorial sheet.
2. **One italic per headline.** Every serif heading should contain exactly one `<em>` that carries the accent colour. Two `<em>`s reads as decoration; one reads as voice.
3. **Mono is for measurements.** Numbers, metric names, KPIs, timestamps — JetBrains Mono, never body Inter.
4. **Uppercase + 0.1em+ tracking = micro-label.** Used for eyebrows, tags, step counters, table labels, ref-section headings, hero-meta keys. Always 11–13px, weight 500–600.
5. **Generous side padding.** `48px` on main at desktop, `24px` on mobile. Never let body copy touch the viewport edge.
6. **Max line length 760–900px.** Body `p` capped at `760px`; lead `p` at `880px`. Long lines kill readability.
7. **Hover is restrained.** Only interactive elements get hover state, and only colour or 2px translate — never scale, glow, or shadow swells.
8. **Animations: 0.2–0.25s ease, two properties max.** Theme transition on body, hover on tiles. That's the budget.
9. **Signal colour belongs to data.** Blue/pink/aqua only ever appear inside badges, status dots, score tiers, and infographic fills. Never on running prose, never as accent text decoration.

---

## 5 · Responsive breakpoints

```
@media (max-width: 980px)  → main padding 24px, multi-col grids collapse to 1 col,
                              tldr-grid → 2 cols, lifecycle → 2 cols, section padding → 64px,
                              sticky aside becomes static
@media (max-width: 760px)  → hamburger nav engages (see §6), table.metrics begins horizontal scroll,
                              chip-row wraps, scoring tiers → 1 col
@media (max-width: 600px)  → tldr-grid → 1 col, lifecycle → 1 col, compare → 1 col stack
```

---

## 6 · Navigation

### 6.1 Desktop nav (sticky top bar)
```
position: sticky; top: 0; z-index: 100;
background: var(--bg);  /* with backdrop-filter: blur(10px) */
border-bottom: var(--rule);
inner: max-width 1280px, padding 18px 48px, flex space-between
```
- Brand: 10px accent-soft dot with 4px accent-bg halo + uppercase 13px label
- Links: 13.5px `--ink-muted`, hover → `--accent`
- Theme toggle: pill button, `1px --surface-border`, hover → border + text become accent

### 6.2 Mobile hamburger menu

> ⚠️ **Note on source:** The existing `ai-eval-metrics.html` doesn't implement a real hamburger — it only hides `.nav-link` at `max-width: 760px`, leaving phone users with no navigation. The spec below fixes that gap. Use this as the canonical mobile pattern going forward.

**Behaviour:**
- Appears below 760px in place of the inline nav links
- Tap opens a full-height **drawer from the right** (not a dropdown — the drawer pattern handles many links + theme toggle gracefully)
- Drawer dismissed by: tapping the X, tapping the scrim, pressing Esc, or selecting a link
- Page body scroll-locks while open (`overflow: hidden` on `<body>`)
- Animated 240ms ease-out slide + scrim fade
- Fully accessible: `aria-expanded`, `aria-controls`, focus trap, focus returns to trigger on close

**Markup:**
```html
<nav class="top">
  <div class="inner">
    <div class="brand"><span class="dot"></span>P&amp;RE</div>

    <!-- desktop links -->
    <div class="nav-links">
      <a href="#metrics">Metrics</a>
      <a href="#implementation">Implementation</a>
      <a href="#tools">Tools</a>
      <button class="theme-toggle">…</button>
    </div>

    <!-- mobile trigger -->
    <button class="hamburger" id="hamburger"
            aria-label="Open menu" aria-controls="drawer" aria-expanded="false">
      <span></span><span></span><span></span>
    </button>
  </div>
</nav>

<!-- drawer + scrim, outside nav so z-index stays sane -->
<div class="scrim" id="scrim" hidden></div>
<aside class="drawer" id="drawer" hidden aria-label="Menu">
  <div class="drawer-head">
    <span class="eyebrow">Contents</span>
    <button class="drawer-close" aria-label="Close menu">✕</button>
  </div>
  <nav class="drawer-nav">
    <a href="#metrics"><span class="ix">01</span>Metrics</a>
    <a href="#implementation"><span class="ix">02</span>Implementation</a>
    <a href="#tools"><span class="ix">03</span>Tools</a>
    <a href="#matrix"><span class="ix">04</span>Matrix</a>
  </nav>
  <div class="drawer-foot">
    <button class="theme-toggle">…</button>
  </div>
</aside>
```

**CSS:**
```css
/* Trigger — hidden on desktop, shown on mobile */
.hamburger {
  display: none;
  background: transparent;
  border: 1px solid var(--surface-border);
  border-radius: 999px;
  width: 40px; height: 40px;
  padding: 0;
  cursor: pointer;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 4px;
  transition: border-color 0.2s;
}
.hamburger:hover { border-color: var(--accent); }
.hamburger span {
  display: block;
  width: 16px;
  height: 1.5px;          /* fine rules — match the editorial 1px ethos */
  background: var(--ink);
  transition: transform 0.24s ease, opacity 0.24s ease;
}
/* Animated X when open */
.hamburger[aria-expanded="true"] span:nth-child(1) { transform: translateY(5.5px) rotate(45deg); }
.hamburger[aria-expanded="true"] span:nth-child(2) { opacity: 0; }
.hamburger[aria-expanded="true"] span:nth-child(3) { transform: translateY(-5.5px) rotate(-45deg); }

@media (max-width: 760px) {
  .nav-links { display: none; }
  .hamburger { display: inline-flex; }
}

/* Scrim — dims the page behind the drawer */
.scrim {
  position: fixed; inset: 0;
  background: rgba(0,0,0,0.4);
  z-index: 200;
  opacity: 0;
  transition: opacity 0.24s ease;
}
.scrim[data-open="true"] { opacity: 1; }
html[data-theme="dark"] .scrim { background: rgba(0,0,0,0.6); }

/* Drawer — slides from the right */
.drawer {
  position: fixed;
  top: 0; right: 0; bottom: 0;
  width: min(360px, 86vw);
  background: var(--bg);
  border-left: 1px solid var(--surface-border);
  z-index: 201;
  transform: translateX(100%);
  transition: transform 0.24s ease;
  display: flex;
  flex-direction: column;
  padding: 24px;
}
.drawer[data-open="true"] { transform: translateX(0); }

.drawer-head {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 32px;
  padding-bottom: 18px;
  border-bottom: 1px solid var(--surface-border);
}
.drawer-close {
  background: transparent;
  border: 1px solid var(--surface-border);
  border-radius: 999px;
  width: 32px; height: 32px;
  color: var(--ink);
  cursor: pointer;
  font-size: 14px;
}
.drawer-close:hover { border-color: var(--accent); color: var(--accent); }

/* Nav list — editorial: italic serif, numbered, generous spacing */
.drawer-nav {
  display: flex;
  flex-direction: column;
  flex: 1;
}
.drawer-nav a {
  display: flex;
  align-items: baseline;
  gap: 16px;
  padding: 18px 0;
  border-top: 1px solid var(--surface-border);
  font-family: 'Instrument Serif', Georgia, serif;
  font-size: 28px;
  line-height: 1.1;
  color: var(--ink);
  text-decoration: none;
  transition: color 0.15s, padding-left 0.2s;
}
.drawer-nav a:last-child { border-bottom: 1px solid var(--surface-border); }
.drawer-nav a:hover,
.drawer-nav a:focus-visible {
  color: var(--accent);
  padding-left: 6px;
}
.drawer-nav .ix {
  font-family: 'JetBrains Mono', monospace;
  font-size: 11px;
  font-style: normal;
  color: var(--ink-subtle);
  letter-spacing: 0.08em;
  min-width: 24px;
}

.drawer-foot {
  padding-top: 24px;
  border-top: 1px solid var(--surface-border);
  display: flex;
  justify-content: flex-start;
}
```

**JS — minimal:**
```js
const hamb   = document.getElementById('hamburger');
const drawer = document.getElementById('drawer');
const scrim  = document.getElementById('scrim');

function setMenu(open) {
  hamb.setAttribute('aria-expanded', open);
  if (open) {
    drawer.hidden = false;  scrim.hidden = false;
    requestAnimationFrame(() => {
      drawer.dataset.open = 'true';
      scrim.dataset.open  = 'true';
    });
    document.body.style.overflow = 'hidden';
    drawer.querySelector('a, button').focus();
  } else {
    drawer.dataset.open = 'false';
    scrim.dataset.open  = 'false';
    document.body.style.overflow = '';
    hamb.focus();
    setTimeout(() => { drawer.hidden = true; scrim.hidden = true; }, 250);
  }
}

hamb.addEventListener('click', () => setMenu(hamb.getAttribute('aria-expanded') !== 'true'));
scrim.addEventListener('click', () => setMenu(false));
document.addEventListener('keydown', e => { if (e.key === 'Escape') setMenu(false); });
drawer.querySelectorAll('a').forEach(a => a.addEventListener('click', () => setMenu(false)));
```

**Why this design fits the system:**
- **Slides from the right** — feels like turning a page in a magazine. Drawers from the left fight with reading direction.
- **Italic serif nav items at 28px** — the same display voice used in headlines, scaled down. The drawer becomes a chapter index.
- **Mono numerals (01, 02…)** — same `decimal-leading-zero` convention as `pov-points`. Continues the editorial counting motif.
- **Fine 1.5px rules** for the bars — `1px` is too thin to feel like a target; `3px` is mobile-app territory. 1.5px keeps the editorial weight.
- **Single accent purple, no secondary colours** — the drawer is chrome, not infographic.

---

## 7 · The site footer

```
padding: 56px 48px;
border-top: var(--rule);
background: var(--bg-alt);
text-align: center;
font-size: 12px;
color: var(--ink-subtle);
```
- Disclaim paragraph, max-width 760px, centred
- Meta row (flex, gap 36px) for source/date/version
- Place the **scoring methodology block (§3.16)** here on reference-style pages, between the disclaim and the meta row.

---

## 8 · Anti-patterns (do not do)

- ❌ Sans-serif headlines — the serif italic is the brand
- ❌ Drop shadows or rounded card stacks — flat editorial only
- ❌ Multiple purple shades on one page — pick one accent purple + its bg tint
- ❌ Secondary colours (pink/blue/aqua) on text or as decoration — they belong only to badges, score tiers, infographic fills
- ❌ Red as a "bad" / negative state — use `--ink-subtle` neutral or `--state-warn` pink
- ❌ Hidden mobile nav with no replacement (the current eval-metrics gap) — always pair `display: none` with a hamburger
- ❌ Emoji in headings; replace with the `→` glyph if directional hint is needed
- ❌ More than three font families
- ❌ Body text in mono
- ❌ Underlining link text by default — use `border-bottom: 1px solid --accent-line` instead
- ❌ Dense `table.compare` patterns for prose content; reach for `table.metrics` instead

---

## 9 · Minimal starter HTML

```html
<!DOCTYPE html>
<html lang="en" data-theme="light">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>…</title>
  <link href="https://fonts.googleapis.com/css2?family=Instrument+Serif:ital@0;1&family=Inter:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet">
  <style>/* tokens from §1.3, body/typography from §1.1, components as needed */</style>
</head>
<body>
  <nav class="top">
    <div class="inner">
      <div class="brand">…</div>
      <div class="nav-links">…</div>
      <button class="hamburger" aria-controls="drawer" aria-expanded="false">…</button>
    </div>
  </nav>
  <div class="scrim" id="scrim" hidden></div>
  <aside class="drawer" id="drawer" hidden>…</aside>

  <main>
    <section class="hero">
      <div class="eyebrow">Section · Date</div>
      <h1 class="display">Headline with <em>one italic</em>.</h1>
      <p class="lead">Lead paragraph.</p>
    </section>

    <section id="…">
      <div class="eyebrow">Label</div>
      <h2 class="section-title">Section heading.</h2>
      <!-- pick one or more components from §3 -->
    </section>

    <section id="reference">
      <!-- table.metrics or table.compare from §3.14 -->
      <!-- + filter chips from §3.15 -->
    </section>
  </main>

  <footer>
    <!-- scoring methodology §3.16 if applicable -->
  </footer>
  <script>/* theme toggle §1.4 + hamburger §6.2 */</script>
</body>
</html>
```
