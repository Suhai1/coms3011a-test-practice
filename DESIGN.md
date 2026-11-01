# DESIGN.md — Product UI Design System

> **Purpose:** An implementation-ready design contract for an AI coding agent building a polished productivity/data application under time pressure.
>
> **Theme for the exam: Forest Modern only.** Warm limestone surfaces, architectural/editorial hierarchy, deep forest-green actions, precise borders, restrained status colours, instrument-panel data presentation. The alternate palettes in §2.2–2.4 are reference only: **do not switch themes during the exam.**
>
> **Priority:** Functional correctness and clear information hierarchy first; visual consistency second; optional "wow" details third. Never sacrifice usability or required features for decoration.
>
> **If time is short:** implement **§A (layout signature)**, **§2.5 (tokens)**, **§3 (type)** and **§7.1, 7.2, 7.5, 7.7, 7.12** first. Everything else is refinement.

---

## ⏱ PROMPTS: exactly what to paste into Qoder, and when

Copy these word for word. Each one goes in a **new chat** unless it says otherwise. Mode and model are given for each.

| # | Prompt | When | Mode / model |
|---|---|---|---|
| D1 | Visual foundation | Task 1 of PLAN.md (~08:45), before any feature | Agent / Auto |
| D2 | UI kit | Straight after D1 (~09:05), 15–20 min | Agent / Auto |
| D3 | Design line for feature tasks | Paste **under** every Task Prompt, all day | (same chat as the task) |
| D4 | Generic check | After the minimum passing version is done (~14:00) | Ask / Auto |
| D5 | Design pass | Straight after D4 (fixes its findings) | Agent / Auto |
| D6 | One wow detail | Only if core is done and it's before 15:30 | Agent / Auto |
| D7 | "This looks generic" rescue | Any time a screen looks like a template | Agent / Ultimate |
| D8 | Final visual check | 16:00, together with Prompt 10 in PROMPTS.md | Agent / Auto |

### D1 — Visual foundation (Task 1, ~08:45)
```
@DESIGN.md @AGENTS.md @PLAN.md
Set up the visual foundation only. No product features yet.
1. Replace src/app/globals.css with the CSS in DESIGN.md §2.5 exactly (Forest Modern tokens, @theme inline mapping, base styles, focus ring, reduced motion). Keep the @import "tailwindcss" line first.
2. Remove any next/font/google imports (Geist) from src/app/layout.tsx; fonts come from the system stacks in §3.
3. Build the app shell following §A (layout signature) and §6: numbered text sidebar (01, 02, ...) with the sections from PLAN.md, compact top bar, page header component with the instrument index eyebrow and registration rule.
4. Make the home page show the page header and an empty state (§7.12).
Run npm run build and tell me the result. Do not add features, libraries or new colours.
```
**Check it:** `npm run dev`. The background should be warm off-white (not white or grey), the sidebar should have numbered text links, and the title should have a small "01 / OVERVIEW"-style label above it and a dark rule below. Commit: `style(ui): add forest modern foundation and shell`.

### D2 — UI kit (~09:05)
```
@DESIGN.md @AGENTS.md
Create a small reusable UI kit in src/components/ui/ following DESIGN.md §A and §7 exactly. Use only the tokens from §2.5 (Tailwind classes like bg-surface, text-ink-muted, border-line, bg-accent). No new colours, no shadows except where §5.3 allows.
Components (typed props, keyboard accessible, all states from §7):
- Button (primary, secondary, ghost, danger; sizes sm/md; loading state that keeps width)
- Field (label + input/select/textarea + helper + error linked with aria-describedby)
- Badge (success, warning, danger, info, neutral; always shows text)
- Section (index label like "02 / ACTIVITY", title, 1px rule, children; NO card box)
- Panel (bordered surface, only for tables, forms and charts)
- DataTable (ledger style from §A.5: header rule, mono IDs, right-aligned numbers, sortable headers, empty and loading states)
- Readout and ReadoutStrip (§A.3)
- EmptyState, ErrorBanner, Skeleton
Then show them all on a temporary page at /kit so I can check them. Run npm run build.
```
**Check it:** open http://localhost:3000/kit and look at every component. Commit: `feat(ui): add ui kit`. Delete `/kit` at 16:00, or leave it out of the nav.

### D3 — Design line for every feature task (paste under the Task Prompt)
```
Design: use only components from src/components/ui and tokens from DESIGN.md. Follow §A (no card grids, readout strip for numbers, ledger tables, sections with index labels). Every list needs empty, loading and error states. No new colours, gradients, shadows or rounded cards.
```

### D4 — Generic check (~14:00, Ask mode, new chat)
```
@DESIGN.md
You are a strict design reviewer. Do NOT change code. Go through every page in src/app and check it against DESIGN.md §A ("the generic test") and §14 (forbidden defaults).
List every problem, worst first, with the file and a one-line fix:
- rows of 3+ equal boxes/cards, cards around every section, rounded big containers
- icon-only navigation, centred narrow layouts, missing index labels or registration rule
- colours not from §2.5 tokens, gradients, shadows, glow, purple/blue SaaS look
- missing empty/loading/error states, colour-only status, missing focus styles
- numbers not in mono/tabular, numbers left-aligned in tables
```

### D5 — Design pass (Agent, new chat, after D4)
```
@DESIGN.md @AGENTS.md
Fix these design problems, one page at a time, without changing any behaviour or data logic:
[paste D4's list here]
After each page, run npm run build. List what you changed.
```
**Check it:** click through every page. Then `npm test` (to make sure nothing broke) and commit: `style(ui): design pass`.

### D6 — One wow detail (only if core is done and it's before 15:30)
```
@DESIGN.md
Add ONE optional detail from DESIGN.md §15: [Live Result Tally | Focused Record Panel | Delta Strip | Change Ledger].
It must use real app data (no fake numbers), use only existing UI kit components and tokens, and must not change existing behaviour. Keep it under 20 minutes. Run npm run build and npm test.
```
Pick **Live Result Tally** if the app has filters, **Focused Record Panel** if it's table-heavy, **Delta Strip** if it has numbers over time, **Change Ledger** if it has edits/history. Commit separately so you can `git revert` it if it breaks something.

### D7 — "This looks generic" rescue (Ultimate)
```
@DESIGN.md
This page (@[page file]) still looks like a default AI SaaS dashboard. Rework its LAYOUT (not its colours or behaviour) to follow DESIGN.md §A strictly:
- replace any row of equal cards with a ReadoutStrip (one oversized primary number, secondary readouts separated by vertical rules)
- remove card boxes around sections; use Section (index label + title + rule) on the canvas
- put tables in ledger style, full width
- use the 2fr/1fr main + inspection rail split on wide screens
Keep all existing functionality working. Show me before/after of what changed.
```

### D8 — Final visual check (16:00, with Prompt 10)
```
@DESIGN.md
Final visual check, no new features:
1. Every page reachable from the sidebar renders without errors and has the page header with index label.
2. No dead buttons, placeholder links, "Lorem ipsum", or template leftovers (Next.js logo, default favicon, /kit in nav).
3. Every list has an empty state; every form shows errors.
4. Check at 1440px and at 390px wide: no horizontal page overflow.
Fix only what's broken. Run npm run build.
```

---

## §A. Layout signature: "an instrument, not a template"

Colours alone don't stop an app looking AI-made. **Layout does.** These rules override §6 and §7.6 wherever they conflict.

**A.1 Numbered text navigation.** Sidebar items are text-first with a monospace two-digit index: `01  Overview`, `02  Tasks`, `03  Reports`. Index in `--ink-subtle`, label in `--ink`. Active item: `--accent-soft` background + 3px `--accent` left rule. An icon is optional; the label is always visible. **Never** an icon-only rail on desktop.

**A.2 Instrument index + registration rule.** Every page header has:
- an eyebrow in mono, 11px, uppercase, `--ink-subtle`: `01 / OVERVIEW` (number matches the sidebar)
- the title: 32px, weight 700, letter-spacing -0.02em
- one optional short sentence in `--ink-muted`
- a **2px `--ink` rule** spanning the full content width under the header (the "registration rule")
- the primary action aligned right of the title

**A.3 Readout strip, not a card grid.** Summary numbers go in ONE horizontal strip with no boxes:
- the **primary readout**: 56px, mono, weight 700, tabular numbers, with its unit in 14px `--ink-muted` (`24 tasks`)
- 2–3 **secondary readouts**: 24px mono, each separated by a 1px `--line` vertical rule
- every readout has an 11px uppercase label above it and, if useful, a delta below ("+3 vs last week")
- a 1px `--line` rule under the strip
**Never** render a row of 3–4 equal bordered cards with an icon in the corner.

**A.4 Sections, not cards.** A section = mono index label (`02 / ACTIVITY`) + 18px title + 1px `--line` rule + content directly on the canvas. Use a bordered **Panel** only for tables, forms and charts. Never nest cards inside cards.

**A.5 Ledger tables.** Tables sit full width (no outer card unless inside a Panel). Header: 11px uppercase `--ink-muted` with a **2px `--ink` bottom rule**. Rows: 44px, 1px `--line` dividers, IDs and dates in mono, numbers right-aligned in mono. Hover: `--accent-soft`. Selected: `--accent-soft` + 3px `--accent` left rule.

**A.6 Asymmetric grid.** On screens ≥1200px, pages with detail use a **2fr main column + 1fr "inspection rail"** (details of the selected item, recent activity, or filters). Content is left-aligned, never a narrow centred column.

**A.7 Calibrated charts.** No box around charts unless inside a Panel. Baseline in `--line-strong` with small tick marks; range and unit labelled in mono ("0–40 tasks · last 8 weeks"); direct labels on bars/lines instead of a legend when ≤3 series.

**A.8 Status as text first.** Status always reads as a word with a small mark: `● Overdue` in `--danger` on `--danger-soft`. Never colour alone.

**The generic test** (fail any one → fix with D7):
- 3+ equal boxes in a row? → readout strip
- a card around every section? → sections on canvas
- icon-only sidebar? → numbered text nav
- everything centred in a narrow column? → left-aligned 2fr/1fr grid
- no index labels or rule under the header? → §A.2

---

## 0. How to use this file

1. Read the actual product brief and identify the primary user, main task, core data, and success state.
2. Use **Forest Modern**. Do not switch themes.
3. Build the tokens and application shell before detailed screens.
4. Implement the main user journey end-to-end before adding optional polish.
5. Use realistic sample data when live data is unavailable; clearly label it as sample data.
6. Do not invent backend behavior. If an integration cannot be completed, make the limitation explicit and provide a useful local/demo state.
7. Keep all required functionality reachable, responsive, keyboard-accessible, and visually coherent.

### Hard constraints

- Next.js 15 App Router, React 19, TypeScript.
- Tailwind CSS v4.1.18. Define theme tokens in `globals.css` using `@theme`; do not use the old v3 `tailwind.config.js` pattern unless the repository already requires it.
- Node.js 18 compatibility.
- Offline/firewalled environment: no Google Fonts, remote fonts, CDNs, external images, or runtime network dependencies.
- Use system font stacks. Use inline SVG or an already-installed icon library such as `lucide-react`; do not use emoji as interface icons.
- Prefer native browser APIs, CSS, SVG, Canvas, and pure JavaScript libraries compatible with Node 18.
- Respect reduced-motion preferences and WCAG AA contrast targets.
- Keep the UI polished but feasible to build quickly. Implement the visual core first; add optional effects only after the main flow works.

---

## 1. Product design principles

### 1.1 Visual thesis: “An instrument, not a template”

The application should feel like a carefully designed working instrument: composed, legible, and quietly confident. Use the visual language of architectural drawings, modernist publishing, technical manuals, and well-made physical tools.

- **Warm, not sterile:** slightly warm neutral backgrounds instead of default cool gray.
- **Structured, not card-heavy:** use page sections, divider lines, tables, and aligned columns. Cards are for genuinely grouped information, not every item.
- **Precise, not decorative:** strong spacing, fine borders, clear labels, deliberate type sizes.
- **Distinctive, not loud:** one main accent, supported by limited status/chart colors.
- **Dense when useful:** data-rich screens should stay readable and scannable, not be artificially spacious.
- **Honest:** show loading, empty, error, stale, partial, and success states. Never imply an action succeeded when it did not.

### 1.2 Layout and hierarchy

Each screen should answer these questions in order:

1. Where am I?
2. What is this page for?
3. What is the most important thing I can do?
4. What information should I inspect next?
5. What happened after my action?

Use a consistent page frame:

- Persistent navigation when the product has multiple primary sections.
- A compact top bar for global search, contextual actions, and account controls.
- A clear page title with a short explanatory line only when useful.
- One primary action per major context; secondary actions should not compete with it.
- Content sections separated by whitespace, rules, and typography.
- Contextual detail panels or dialogs for secondary workflows, rather than sending users away unnecessarily.

### 1.3 Color budget

For the default Forest Modern theme, aim for roughly:

- **75%** warm neutrals and surfaces.
- **18%** ink, navigation, and structural text.
- **7%** accent and status colors combined.

Do not treat these ratios as a literal requirement; use them as a guardrail against an over-colored UI.

---

## 2. Theme tokens

Use semantic tokens in CSS. Components should reference roles such as `bg-canvas`, `text-ink`, `border-line`, and `bg-accent`, never hard-code palette hex values in multiple places.

### 2.1 Default theme: Forest Modern

Personality: grounded, precise, editorial, quietly premium.

| Role | Token | Hex |
|---|---|---|
| App background | `--canvas` | `#F4F1E8` |
| Main surface | `--surface` | `#FBF9F3` |
| Raised surface | `--surface-raised` | `#FFFFFF` |
| Sunken surface | `--surface-sunken` | `#EAE6DB` |
| Main text | `--ink` | `#202821` |
| Secondary text | `--ink-muted` | `#59635A` |
| Subtle text | `--ink-subtle` | `#737B73` |
| Standard border | `--line` | `#D3D0C5` |
| Strong border | `--line-strong` | `#A9ACA0` |
| Main accent | `--accent` | `#245B45` |
| Accent hover | `--accent-hover` | `#194633` |
| Accent pressed | `--accent-pressed` | `#123728` |
| Text on accent | `--accent-ink` | `#FFFFFF` |
| Soft accent surface | `--accent-soft` | `#E0EAE1` |
| Accent border | `--accent-line` | `#9CB6A1` |
| Success | `--success` | `#236747` |
| Success soft | `--success-soft` | `#E1EEE5` |
| Warning | `--warning` | `#8A4B08` |
| Warning soft | `--warning-soft` | `#F5E8D1` |
| Danger | `--danger` | `#A12F2F` |
| Danger soft | `--danger-soft` | `#F5E3DF` |
| Info | `--info` | `#285A73` |
| Info soft | `--info-soft` | `#E0EBF0` |

Chart palette, ordered for differentiation:

1. Forest `#245B45`
2. Terracotta `#B94D35`
3. Steel blue `#326C91`
4. Ochre `#9A7628`
5. Muted plum `#745A80`
6. Sage `#658A83`

Use chart colors for data series only; do not repurpose them randomly across UI components. Pair color with labels, markers, line patterns, or direct annotations so meaning is not color-only.

### 2.2 Alternate theme: Neon Night

Personality: technical, bold, developer-first. Best for developer tools, AI tools, observability, and data-heavy products that suit a dark interface.

| Role | Hex |
|---|---|
| App background | `#0B0D0C` |
| Main surface | `#131715` |
| Raised surface | `#1A201C` |
| Sunken surface | `#080A09` |
| Main text | `#F0F5F1` |
| Secondary text | `#A0ACA4` |
| Subtle text | `#7C8980` |
| Standard border | `#303A33` |
| Strong border | `#4C5B50` |
| Main accent | `#34D59A` |
| Accent hover | `#25B981` |
| Accent pressed | `#188A60` |
| Text on accent | `#062016` |
| Soft accent surface | `#172B20` |
| Accent border | `#2F7658` |
| Success | `#55D6A1` |
| Success soft | `#163528` |
| Warning | `#F6C667` |
| Warning soft | `#352B17` |
| Danger | `#FF7777` |
| Danger soft | `#381D1D` |
| Info | `#7BC8F2` |
| Info soft | `#162B38` |

Neon rules:
- Reserve bright green for primary actions, selected states, and the most important data.
- Do not give every icon, chart, and border a neon color.
- Use clear border contrast and large enough text; dark themes still need readable secondary text.
- Do not use glow shadows as a substitute for hierarchy.

### 2.3 Alternate theme: Paper & Graphite

Personality: calm, content-first, professional. Best for research, reporting, productivity, and information-heavy tools.

| Role | Hex |
|---|---|
| App background | `#F7F6F3` |
| Main surface | `#FFFFFF` |
| Raised surface | `#F0F2EF` |
| Sunken surface | `#ECEDE8` |
| Main text | `#252522` |
| Secondary text | `#565E59` |
| Subtle text | `#707772` |
| Standard border | `#D4D9D3` |
| Strong border | `#AAB4AC` |
| Main accent | `#176B65` |
| Accent hover | `#10534E` |
| Accent pressed | `#0B403C` |
| Text on accent | `#FFFFFF` |
| Soft accent surface | `#E0EFEB` |
| Accent border | `#9BC8C0` |
| Success | `#216749` |
| Success soft | `#E1EEE5` |
| Warning | `#8A570C` |
| Warning soft | `#F5E8D1` |
| Danger | `#A12F2F` |
| Danger soft | `#F5E3DF` |
| Info | `#285A73` |
| Info soft | `#E0EBF0` |

### 2.4 Alternate theme: Coral & Ink

Personality: warm, confident, memorable. Best for creative workflows, customer-facing products, and tools that benefit from a distinctive brand accent.

| Role | Hex |
|---|---|
| App background | `#F3F4F2` |
| Main surface | `#FFFFFF` |
| Raised surface | `#F8F7F4` |
| Sunken surface | `#E9EBE7` |
| Main text | `#202321` |
| Secondary text | `#555F59` |
| Subtle text | `#717A73` |
| Standard border | `#D4D9D4` |
| Strong border | `#A8B0A9` |
| Main accent | `#C94F38` |
| Accent hover | `#A93C2A` |
| Accent pressed | `#85301F` |
| Text on accent | `#FFFFFF` |
| Soft accent surface | `#F9E7E3` |
| Accent border | `#E6A69A` |
| Success | `#216749` |
| Success soft | `#E1EEE5` |
| Warning | `#80520E` |
| Warning soft | `#F5E8D1` |
| Danger | `#B42332` |
| Danger soft | `#F7E1E3` |
| Info | `#285A73` |
| Info soft | `#E0EBF0` |

### 2.5 Tailwind CSS v4 implementation

Put tokens in `app/globals.css` (or the global stylesheet already used by the repository). This is a working starting point for Forest Modern. Keep the semantic variables and adapt the project’s existing import structure as needed.

```css
@import "tailwindcss";

:root {
  color-scheme: light;

  --canvas: #F4F1E8;
  --surface: #FBF9F3;
  --surface-raised: #FFFFFF;
  --surface-sunken: #EAE6DB;
  --ink: #202821;
  --ink-muted: #59635A;
  --ink-subtle: #737B73;
  --line: #D3D0C5;
  --line-strong: #A9ACA0;
  --accent: #245B45;
  --accent-hover: #194633;
  --accent-pressed: #123728;
  --accent-ink: #FFFFFF;
  --accent-soft: #E0EAE1;
  --accent-line: #9CB6A1;
  --success: #236747;
  --success-soft: #E1EEE5;
  --warning: #8A4B08;
  --warning-soft: #F5E8D1;
  --danger: #A12F2F;
  --danger-soft: #F5E3DF;
  --info: #285A73;
  --info-soft: #E0EBF0;
  --chart-1: #245B45;
  --chart-2: #B94D35;
  --chart-3: #326C91;
  --chart-4: #9A7628;
  --chart-5: #745A80;
  --chart-6: #658A83;

  --radius-control: 4px;
  --radius-panel: 5px;
  --radius-dialog: 8px;
  --shadow-menu: 0 8px 24px rgb(32 40 33 / 10%);
  --shadow-dialog: 0 20px 56px rgb(32 40 33 / 16%);
}

@theme inline {
  --color-canvas: var(--canvas);
  --color-surface: var(--surface);
  --color-surface-raised: var(--surface-raised);
  --color-surface-sunken: var(--surface-sunken);
  --color-ink: var(--ink);
  --color-ink-muted: var(--ink-muted);
  --color-ink-subtle: var(--ink-subtle);
  --color-line: var(--line);
  --color-line-strong: var(--line-strong);
  --color-accent: var(--accent);
  --color-accent-hover: var(--accent-hover);
  --color-accent-pressed: var(--accent-pressed);
  --color-accent-ink: var(--accent-ink);
  --color-accent-soft: var(--accent-soft);
  --color-accent-line: var(--accent-line);
  --color-success: var(--success);
  --color-success-soft: var(--success-soft);
  --color-warning: var(--warning);
  --color-warning-soft: var(--warning-soft);
  --color-danger: var(--danger);
  --color-danger-soft: var(--danger-soft);
  --color-info: var(--info);
  --color-info-soft: var(--info-soft);
  --color-chart-1: var(--chart-1);
  --color-chart-2: var(--chart-2);
  --color-chart-3: var(--chart-3);
  --color-chart-4: var(--chart-4);
  --color-chart-5: var(--chart-5);
  --color-chart-6: var(--chart-6);

  --font-sans: Arial, "Liberation Sans", "DejaVu Sans", sans-serif;
  --font-mono: "Ubuntu Mono", "DejaVu Sans Mono", "Liberation Mono", monospace;

  --radius-control: 4px;
  --radius-panel: 5px;
  --radius-dialog: 8px;
}

* {
  box-sizing: border-box;
}

html {
  min-height: 100%;
  background: var(--canvas);
}

body {
  min-height: 100vh;
  margin: 0;
  background: var(--canvas);
  color: var(--ink);
  font-family: Arial, "Liberation Sans", "DejaVu Sans", sans-serif;
  font-size: 14px;
  line-height: 1.5;
  -webkit-font-smoothing: antialiased;
  text-rendering: optimizeLegibility;
}

::selection {
  background: var(--accent-soft);
  color: var(--ink);
}

:focus-visible {
  outline: 3px solid var(--info);
  outline-offset: 2px;
}

@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    scroll-behavior: auto !important;
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

**Dark-theme note:** if implementing a theme switcher, override the semantic CSS variables under `[data-theme="neon"]` or a class on the root. Do not duplicate component styles per theme. `color-scheme` should match the active theme so native controls are legible.

---

## 3. Typography

### 3.1 Font stacks

No network fonts. Prefer this stack:

```css
font-family: Arial, "Liberation Sans", "DejaVu Sans", sans-serif;
```

For numeric IDs, timestamps, and code-like values:

```css
font-family: "Ubuntu Mono", "DejaVu Sans Mono", "Liberation Mono", monospace;
font-variant-numeric: tabular-nums;
```

Use available system fonts only. Do not rely on a specific font being installed.

### 3.2 Type scale

| Role | Size | Weight | Line height | Usage |
|---|---:|---:|---:|---|
| Page title | 28–32px | 700 | 1.15 | Main screen heading |
| Section title | 18–20px | 700 | 1.25 | Content group heading |
| Panel title | 15–16px | 700 | 1.3 | Table/panel heading |
| Body | 14px | 400 | 1.5 | Default interface copy |
| Body emphasis | 14px | 600 | 1.45 | Important labels |
| Supporting | 12–13px | 400 | 1.45 | Helper text, descriptions |
| Eyebrow/metadata | 10–11px | 700 | 1.3 | Section labels, uppercase metadata |
| KPI value | 26–34px | 700 | 1.05 | Main metric |
| Table number | 13–14px | 500 | 1.4 | Tabular values |

Rules:
- Use uppercase labels sparingly, with modest letter spacing (`0.04em` to `0.08em`).
- Avoid making every label tiny or uppercase.
- Do not use light font weights for essential content.
- Use `font-variant-numeric: tabular-nums` for aligned numeric columns.
- Keep paragraphs to a comfortable line length; avoid wide text blocks when the page is mostly data.

---

## 4. Spacing, grid, and responsive layout

Use a consistent 4px spacing base.

| Token | Value | Typical use |
|---|---:|---|
| `space-1` | 4px | Icon gap, tiny separation |
| `space-2` | 8px | Compact control spacing |
| `space-3` | 12px | Tight groups |
| `space-4` | 16px | Standard component padding |
| `space-5` | 20px | Group separation |
| `space-6` | 24px | Panel padding |
| `space-8` | 32px | Section separation |
| `space-10` | 40px | Major page spacing |
| `space-12` | 48px | Large section separation |
| `space-16` | 64px | Large screen-only gaps |

Suggested shell:
- Sidebar: 224–248px on desktop; collapsible when useful.
- Top bar: 56–64px high.
- Main content max width: 1440px for broad dashboards; 1120px for form- or text-heavy workflows.
- Main content horizontal padding: 24–32px desktop, 16–20px tablet, 16px mobile.
- Section gap: 24–32px.
- Grid gap: 16px; 20–24px for spacious editorial layouts.

Responsive behavior:
- Desktop: persistent sidebar and multi-column content where useful.
- Tablet: sidebar collapses or becomes a drawer; reduce grid columns.
- Mobile: one column, full-width controls, horizontally scrollable data tables only when necessary, with clear indication that more columns exist.
- Never let fixed-width content create horizontal page overflow.
- On narrow screens, prioritize the main task over secondary dashboard decoration.

---

## 5. Shape, borders, and elevation

### 5.1 Radius

Use small, controlled radii to reinforce precision.

- Controls: 4px.
- Standard panels/cards: 4–5px.
- Dialogs: 8px.
- Pills: reserved for compact status labels only when the shape makes sense.
- Do not make every surface a rounded card.
- Avoid giant rounded containers and fully pill-shaped primary buttons.

### 5.2 Borders

- Standard divider: 1px solid `var(--line)`.
- Strong divider: 1px solid `var(--line-strong)`.
- Use borders to structure tables, panel headers, and form groups.
- Prefer a clear rule and whitespace over a shadow when separating sections.
- Do not use low-contrast borders that disappear on the background.

### 5.3 Shadows

Shadows are for layering, not decoration:
- Most cards and panels: no shadow.
- Menus/popovers: `var(--shadow-menu)`.
- Modal dialogs: `var(--shadow-dialog)`.
- No soft glow shadows.
- No glassmorphism or backdrop blur.

---

## 6. Core application shell

### 6.1 Sidebar

> **See §A.1:** numbered text navigation takes priority over the icon guidance below.


- Background: `var(--surface)` or a deliberate dark navigation surface if it helps hierarchy.
- Brand area: compact, aligned to the content grid; no oversized decorative logo.
- Navigation items: 36–40px tall with 8–12px horizontal padding.
- Use a small inline SVG icon and a clear text label.
- Active item: `var(--accent-soft)` background and `var(--accent)` text or a 3px left indicator. Use both a color/shape cue and accessible semantics.
- Hover: subtle background change, no bounce.
- Groups: separate with spacing and small uppercase section labels only if there are enough items.
- Bottom area: settings/help/account actions, separated from primary navigation.
- On mobile: use a drawer with a visible close control and focus management.

### 6.2 Top bar

Include only the controls that matter to the product:
- Breadcrumbs or current workspace.
- Global search or command search if there are many destinations.
- Contextual actions such as “Create”, “Import”, or “Export”.
- Notifications/profile only if relevant.

Keep the top bar compact. Avoid a row of decorative icon-only buttons. Icon-only buttons require accessible names and tooltips.

### 6.3 Page header

Pattern:

- Eyebrow or breadcrumb (optional).
- Page title.
- One short explanatory sentence when it helps orientation.
- Primary action aligned with title on wide screens; stacked on mobile.
- Optional filter/date range controls below the title.

Do not use a generic centered hero with a giant headline and blob artwork for an internal productivity app. Put the user’s task and data first.

---

## 7. Component specifications

### 7.1 Buttons

Variants:
- **Primary:** solid accent background, high-contrast text.
- **Secondary:** surface background, strong border, ink text.
- **Tertiary/ghost:** transparent background, no prominent border; for low-priority actions.
- **Danger:** danger color, used only for destructive actions.
- **Icon button:** compact square control, accessible name required.

Sizing:
- Small: 30–32px high, 12px text.
- Default: 36–40px high, 13–14px text.
- Large: 44–48px high, 14–15px text.
- Horizontal padding: 12–16px.

States:
- Hover: change background/border, not size.
- Active: slightly darker.
- Focus: visible 3px outline with offset.
- Disabled: muted and non-interactive; do not rely only on opacity if text becomes unreadable.
- Loading: preserve button width, show spinner or progress label, prevent duplicate submissions.

Copy:
- Prefer verbs: “Create report”, “Save changes”, “Import file”.
- Avoid vague “Submit” unless context makes the action obvious.
- Destructive buttons should name the outcome: “Delete workspace”.

### 7.2 Inputs and selects

- Minimum height 38–40px.
- Background: `var(--surface-raised)`.
- Border: 1px solid `var(--line-strong)` or `var(--line)` if the boundary is still clearly visible.
- Radius: 4px.
- Horizontal padding: 10–12px.
- Label above field; never use placeholder as the only label.
- Helper text sits below field.
- Error text appears near the field and is connected with `aria-describedby`.
- Focus state uses a visible outline and border change.
- Disabled fields must still have readable labels and values.
- For search fields, show a search icon only if it helps recognition.

### 7.3 Checkboxes, radios, and toggles

- Use native controls where possible.
- Associate each control with a visible label.
- Minimum hit area: 32px; aim for 40px where space permits.
- Checked state must be identifiable beyond color alone.
- Toggles are for immediate on/off settings; use checkboxes for choices in a form that are saved together.
- Avoid custom controls unless their keyboard and screen-reader behavior is implemented correctly.

### 7.4 Tabs

- Use tabs only for sibling views of the same content context.
- Active tab: strong text and a 2px underline or a restrained accent background.
- Keep labels concise.
- Support keyboard navigation and correct `tablist`, `tab`, and `tabpanel` semantics when implementing ARIA tabs.
- If the tabs navigate to separate pages, use links styled as tabs instead.

### 7.5 Tables

Tables are first-class product surfaces.

- Header row: slightly tinted background or clear bottom rule; 11–12px semibold text.
- Body rows: 42–48px high depending on density.
- Use `font-variant-numeric: tabular-nums` for numbers.
- Right-align numeric values; left-align names and descriptions.
- Keep column widths stable; truncate only when a useful tooltip or detail view is available.
- Use subtle row hover and a clear selected-row state.
- Avoid vertical gridlines unless the data genuinely benefits from them.
- Show sort state with a clear icon and accessible label.
- Use pagination or progressive loading when the data set is large.
- On mobile, prioritize columns or allow a labelled horizontal scroll region.
- Provide a useful empty state and a visible loading state.

### 7.6 Cards and metric tiles

> **Overridden by §A.3:** summary numbers use the readout strip, not a grid of metric cards.


Use a card when a set of information needs a distinct boundary. Do not wrap every section in a card.

Metric tile anatomy:
- Small label.
- Prominent value.
- Optional unit or comparison.
- Optional sparkline if it adds context.
- Comparison label such as “vs previous 7 days”.
- Avoid decorative icon tiles unless the icon has real semantic value.

Prefer aligned metrics in a shared row with consistent baselines. Avoid excessive gradients, shadows, and giant radii.

### 7.7 Badges and status labels

- Compact height: 22–26px.
- Use a light status surface with dark readable text and optional icon.
- Always include a text label such as “Failed”, “In progress”, or “Complete”.
- Do not communicate state by color alone.
- Keep status meanings consistent across the entire product.

Suggested meanings:
- Success: completed, synced, verified.
- Warning: attention needed, nearing a limit, partial result.
- Danger: failed, destructive, blocked.
- Info: neutral update, informational state.

### 7.8 Filters and search

- Show active filters as removable tokens or a concise summary.
- Make “Clear filters” easy to find when filters are active.
- Preserve filter state when navigating back from a detail view where feasible.
- Search should have a clear label or accessible name, empty state, loading state, and no-results state.
- Debounce remote/local search only when useful; do not add complexity without a real need.

### 7.9 Dialogs and drawers

- Use dialogs for focused tasks requiring a decision or short form.
- Use drawers for contextual detail or longer side-by-side workflows.
- Title the dialog and explain the consequence when it is destructive.
- Provide an obvious close/cancel action.
- Keep keyboard focus inside a modal dialog while open and restore focus to the trigger when closed.
- Escape should close a non-destructive dialog when appropriate.
- Do not use a modal for simple confirmation if an inline undo action is safer and clearer.

### 7.10 Toasts and alerts

- Toasts confirm transient actions: “Report exported”.
- Inline alerts explain persistent problems or important context.
- Use text plus an icon or clear heading; do not rely on color alone.
- Do not show a toast for every minor interaction.
- Error messages should say what happened and what the user can do next.
- Do not automatically dismiss critical errors before the user can read them.

### 7.11 Tooltips

- Use for icon-only controls or genuinely abbreviated labels.
- Short, direct, and not required to understand a core task.
- Appear on hover and keyboard focus.
- Do not use tooltips as the only location for essential instructions.
- Avoid delayed or animated tooltip effects that slow down frequent workflows.

### 7.12 Empty, loading, and error states

**Empty state**
- Say what is empty.
- Explain the next useful action.
- Include one primary CTA when applicable.
- Use a small SVG illustration only if it clarifies the state; do not use generic blobs or emoji.

**Loading state**
- Prefer skeletons that match the eventual content structure.
- Use a spinner for a single short action, not for a full page that could use a skeleton.
- Announce important asynchronous updates to assistive technology when appropriate.
- Avoid fake progress percentages.

**Error state**
- Explain the issue in plain language.
- Preserve user input whenever possible.
- Offer retry, recovery, or an alternate path.
- Show technical details only when they help the user or are appropriate for a developer-facing tool.
- Never display a success state for a failed operation.

### 7.13 Pagination and bulk actions

- Keep pagination near the table footer.
- Show current page and total count if known.
- Disable unavailable previous/next actions visibly and semantically.
- Bulk actions appear only when rows are selected.
- Make the number of selected items explicit.
- Provide a clear way to clear selection.

---

## 8. Data visualization

Charts should explain the data, not decorate the page.

### 8.1 General rules

- Give every chart a clear title and useful time range or unit.
- Label axes and units when they are needed to interpret the chart.
- Prefer direct labels over a distant legend when there are only a few series.
- Use chart palette tokens consistently.
- Do not encode meaning with color alone: add labels, line styles, symbols, or patterns where appropriate.
- Use a visible baseline for bar charts.
- Keep gridlines subtle but visible.
- Avoid 3D charts, gratuitous animation, and decorative gradients.
- Include a text/table alternative for essential chart information where feasible.
- Distinguish zero, missing, and unavailable data.
- Never fabricate a data point to make a chart look complete.

### 8.2 Bar charts

- Use for comparing discrete categories or values across time.
- Bars should have consistent width and spacing.
- Start the value axis at zero unless there is a clearly explained exception.
- Show values on hover/focus and label key values directly when practical.

### 8.3 Line charts

- Use for trends over continuous time or ordered values.
- Avoid excessive smoothing that implies data not present.
- Use visible points for sparse data.
- Highlight a selected point with a marker and a readable tooltip.
- Use solid/dashed line differences for multiple series if colors alone are insufficient.

### 8.4 Donut and pie charts

- Use only for a small number of meaningful parts of a whole.
- Avoid too many slices.
- Include total and labels; do not force users to estimate from color.
- Prefer a bar chart when comparing similar values accurately matters.

### 8.5 Sparklines

- Use sparklines as supplementary trend context in a metric tile or table.
- Do not use them as the only representation of important data.
- Avoid axes when the sparkline is intentionally compact, but provide accessible text describing the trend.

### 8.6 Chart interactions

- Hover and keyboard focus should reveal equivalent details.
- Tooltips must remain within the viewport.
- Selection should be visible and reversible.
- Avoid complex chart libraries unless they are already installed or materially reduce implementation time.
- SVG is a strong default for simple charts; Canvas is appropriate for larger point sets.

---

## 9. Iconography and imagery

### 9.1 Icons

- Use one consistent icon family if already available, such as `lucide-react`.
- If unavailable, use small inline SVGs with consistent stroke width (typically 1.75–2px).
- Typical size: 16px for compact controls, 18px for navigation, 20px for prominent actions.
- Align icons optically with text.
- Do not mix filled emoji, outline icons, and unrelated icon styles.
- Every icon-only button needs an accessible name and a tooltip when the action is not obvious.
- Icons should clarify an action or category, not fill empty space.

### 9.2 Imagery

- The app must work without external imagery.
- Use local assets only if they are provided in the repository or can be generated as part of the project.
- Prefer data, diagrams, SVG, and typographic composition over stock photos for tool interfaces.
- Do not reference remote images or font URLs.
- Do not use placeholder images that could be mistaken for actual user data.

---

## 10. Microcopy and interaction feedback

Voice: concise, calm, direct, helpful.

Prefer:
- “Create report”
- “Changes saved”
- “No records match these filters”
- “Import CSV”
- “Try again”
- “You have unsaved changes”

Avoid:
- “Oopsie!”
- “Yay!!!”
- “Magic happens here”
- “Submit” when a more specific verb is available.
- Long explanations inside button labels.
- Fake conversational personality in professional workflows.

Every action should give feedback:
- Save: saved confirmation or inline error.
- Delete: clear confirmation, then success/error feedback.
- Import: validation summary, progress if real, and result summary.
- Search: result count or helpful no-results state.
- Async operation: pending state and a final result.

Do not simulate network success when no request happened. If using local demo data, label it as demo/sample data where that distinction matters.

---

## 11. Motion and transitions

Motion is optional polish, not a requirement.

Recommended:
- Hover/background transitions: 120–160ms.
- Dialog/drawer entry: 160–220ms.
- Small state transitions: 120–180ms.
- Use ease-out for entry and ease-in for exit.
- Prefer opacity and small translations over scaling entire layouts.
- No bounce, springy overshoot, parallax, or attention-seeking perpetual animation.
- Never animate content in a way that harms reading or task completion.
- Respect `prefers-reduced-motion: reduce` globally.

Use motion only when it helps users understand a state change or spatial relationship.

---

## 12. Accessibility requirements

Target WCAG 2.2 AA where practical. At minimum:

- Text contrast: 4.5:1 for normal text; 3:1 for large text.
- Non-text UI components and meaningful graphical boundaries: 3:1 against adjacent colors where required.
- All functionality available by keyboard.
- Visible focus indicators, never removed without a replacement.
- Semantic HTML before ARIA.
- Correct accessible names for controls.
- Labels for every form input.
- Errors linked to the relevant input.
- Status messages announced appropriately when content changes asynchronously.
- Do not use color as the only signal.
- Respect reduced motion.
- Use logical heading order.
- Ensure dialogs manage focus correctly.
- Avoid tiny targets; aim for approximately 40px controls and comfortable hit areas.
- Do not make essential content hover-only.
- Use `aria-live="polite"` for non-urgent dynamic status where appropriate; do not overuse live regions.
- Test keyboard navigation, zoom, and narrow viewports.

**Contrast note:** palette choices are starting points, not a guarantee. Verify actual text/background pairs in the rendered UI. For small text, especially on accent colors and status surfaces, calculate contrast and adjust the color if necessary.

---

## 13. Performance and implementation discipline

- Keep components focused and typed.
- Use Server Components by default; add `"use client"` only when interactivity requires it.
- Avoid unnecessary client-side state and dependencies.
- Reuse existing project patterns instead of replacing working infrastructure.
- Do not install a large UI framework for a handful of controls.
- Avoid runtime network requests unless the product brief requires them and the environment supports them.
- Use local deterministic sample data for demos and tests.
- Add useful empty/error/loading states to asynchronous views.
- Avoid hydration mismatches; keep date/time and random values deterministic where they render on both server and client.
- Use `next/image` only when it is appropriate for local images; do not introduce remote image dependencies.
- Check the terminal after implementation and resolve build/type errors.

---

## 14. Forbidden visual defaults

Do **not** use:
- Glassmorphism or `backdrop-filter: blur(...)`.
- Purple, violet, or indigo as the default primary accent.
- Generic dark-gray + bright-blue SaaS styling without a deliberate reason.
- Gradient-filled hero sections or gradient cards.
- Soft glow shadows.
- Excessive rounded corners or pill buttons everywhere.
- Emoji as interface icons.
- A generic centered hero with abstract blobs for a productivity dashboard.
- A card around every section and every data point.
- Tiny low-contrast metadata as the primary way to communicate information.
- Random accent colors without semantic meaning.
- Fake charts, fake metrics, or fabricated success states.
- External font/CDN/image dependencies.
- Decorative animation that delays or distracts from the main task.
- Default component-library styling left uncustomized.

---

## 15. Optional “wow” details

These are optional. Implement only after the main workflow, responsive layout, and states are complete. Pick **one or two**, not all of them.

| Detail | Effect | Implementation | Effort |
|---|---|---|---|
| Instrument Index | Makes a dense screen feel like a technical instrument | Small uppercase section IDs, e.g. `01 / OVERVIEW`, `02 / ACTIVITY` | 10–15 min |
| Registration Rule | Creates a strong editorial identity | Fine horizontal rules aligned with section headings and table headers | 5–10 min |
| Live Result Tally | Makes filters feel responsive and useful | Show “24 results” beside filter controls; update from real local state | 10–15 min |
| Change Ledger | Builds trust for edits | Small chronological list of recent changes with time, actor/demo label, and event | 15–20 min |
| Chart Annotation Rail | Makes trends easier to interpret | One or two compact notes beside a chart, derived from actual data | 15–20 min |
| Precision Hover | Adds tactile clarity without visual noise | Subtle row background and border transition; no movement or glow | 5–10 min |
| Command Search | Speeds up navigation | `Ctrl/Cmd + K` opens a simple accessible command dialog | 15–25 min |
| Focused Record Panel | Keeps context during inspection | Clicking a table row opens a right-side detail drawer | 15–25 min |
| Keyboard Row Navigation | Makes tables feel like a professional tool | Arrow-key navigation with visible selected row and correct focus handling | 15–25 min |
| Print-Style Section Headers | Adds editorial polish | Small numbered uppercase label above a strong heading | 5–10 min |
| Delta Strip | Summarizes performance at a glance | Compact row showing current value, change, and comparison period | 10–15 min |
| Confidence-Aware Data States | Makes data quality explicit | Labels for sample, partial, stale, or unavailable data | 10–15 min |
| Inspection Coordinates | Adds subtle instrument character | Display record ID, range, or selected item count in monospace metadata | 5–10 min |
| Audit Trail | Makes actions feel accountable | Record real local actions with timestamps; do not invent remote actors | 15–25 min |
| Calibrated Chart Scale | Improves analytical credibility | Label units, baseline, range, and normalization choices | 10–20 min |

### Best exam-safe picks

1. **Instrument Index** — fast visual identity with low implementation risk.
2. **Live Result Tally** — high perceived usefulness if filtering exists.
3. **Focused Record Panel** — impressive if the product is table/data-heavy and time permits.

Do not implement keyboard row navigation or a command palette if they risk breaking core interactions under time pressure.

---

## 16. Agent execution plan

The coding agent should follow this order.

### Phase 1 — Understand (5–10 minutes)

- Read the brief and inspect the existing repository.
- Identify the primary user journey and required features.
- Identify available dependencies and existing components.
- List assumptions that affect data or behavior.
- Do not rewrite project configuration without a reason.

### Phase 2 — Visual foundation (15–25 minutes)

- Add the selected theme tokens to the global stylesheet.
- Establish body typography, focus styles, reduced motion, and background.
- Build the responsive shell: navigation, top bar, page header, content grid.
- Create the basic button, input, badge, and panel styles.
- Confirm the app renders before adding detailed features.

### Phase 3 — Main workflow (largest share of time)

- Implement the core task end-to-end.
- Use typed models and realistic local sample data if needed.
- Build real interactions for search, filtering, sorting, selection, save, create, import, or export only where required by the brief.
- Make every primary button do something meaningful.
- Add loading, empty, success, and error states.
- Keep data state and derived views consistent.

### Phase 4 — Responsive and accessible pass (15–25 minutes)

- Test keyboard-only navigation.
- Verify visible focus, labels, and disabled states.
- Check the layout at desktop, tablet, and mobile widths.
- Fix overflow and clipped menus/dialogs.
- Check reduced-motion behavior.
- Verify status text is understandable without color.

### Phase 5 — Polish and verify (remaining time)

- Add one or two optional “wow” details only if the core is stable.
- Run the available lint/typecheck/build commands.
- Fix runtime errors and console warnings that affect behavior.
- Check all key flows manually.
- Remove dead buttons, placeholder links, and obvious template leftovers.
- Keep the result coherent rather than adding more features at the last minute.

---

## 17. Required implementation checklist

Before considering the work complete, confirm:

### Product and functionality
- [ ] The primary user journey works from start to finish.
- [ ] Every visible primary action works.
- [ ] Forms validate input and display understandable errors.
- [ ] Search/filter/sort state updates the actual displayed content.
- [ ] Destructive actions are clearly labelled and appropriately confirmed.
- [ ] Demo/sample data is not presented as live data.
- [ ] Empty, loading, error, and success states exist where needed.

### Visual quality
- [ ] One theme is used consistently.
- [ ] Semantic tokens are centralized in CSS.
- [ ] Typography hierarchy is clear.
- [ ] Spacing and alignment are consistent.
- [ ] Cards are used intentionally, not indiscriminately.
- [ ] No forbidden gradients, glow shadows, glassmorphism, emoji icons, or default purple accents.
- [ ] Icons come from one consistent family or inline SVG set.
- [ ] Hover, focus, active, disabled, and selected states are defined.

### Accessibility and responsive behavior
- [ ] Keyboard navigation works.
- [ ] Focus indicators are visible.
- [ ] Inputs have labels.
- [ ] Status and errors are not communicated by color alone.
- [ ] Reduced motion is respected.
- [ ] No page-level horizontal overflow at mobile width.
- [ ] Dialogs/drawers can be opened and closed accessibly.
- [ ] Text and controls have adequate contrast.

### Engineering
- [ ] TypeScript errors are resolved.
- [ ] The app runs with the repository’s expected command.
- [ ] No unnecessary external requests or remote assets are required.
- [ ] No obvious runtime errors remain.
- [ ] Main interactions were manually tested.

---

## 18. Copy-paste instruction for the coding agent

Use this as the initial instruction when starting work with this file:

> Read `DESIGN.md` fully and treat it as the UI design contract. First inspect the existing repository and the actual product brief. Implement the required product behavior before optional decoration. Use the Forest Modern theme and the layout signature in §A. Centralize theme tokens in the global CSS file and use Tailwind CSS v4 `@theme` mappings. Respect the specified Next.js 15 App Router, React 19, TypeScript, Tailwind CSS v4, Node 18, offline environment, system fonts, accessibility, and reduced-motion constraints. Do not add remote fonts, CDNs, external images, glassmorphism, gradients, glow shadows, excessive rounding, emoji icons, or generic template styling. Build the shell, complete the main user journey, add honest loading/empty/error/success states, then verify responsiveness and keyboard accessibility. Only after the core is stable, add one or two low-risk wow details. Run the available checks and fix errors before finishing. Do not claim an action works unless it actually works.

---

## 19. Final principle

A strong interface is not the one with the most effects. It is the one where the hierarchy is obvious, the data is trustworthy, the actions are clear, the details are consistent, and the user can finish the task without friction.

**Build the instrument. Then calibrate it.**
