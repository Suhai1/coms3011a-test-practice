# DESIGN.md — Visual style rules

> TONIGHT: pick Direction 1 or Direction 2, delete the other one, and change any colour you don't like.
> The AI must use ONLY these tokens. No new colours, no improvising.

---

## Direction 1 — "Forest Brutalist" (deep green + concrete architecture)
Mood: a modern concrete building with a green garden wall. Heavy structure, flat surfaces, strong lines, calm.

```css
:root {
  --bg:        #EEEDE8;  /* warm concrete */
  --surface:   #FFFFFF;  /* cards, panels */
  --ink:       #161A16;  /* main text */
  --muted:     #5E655E;  /* secondary text */
  --line:      #161A16;  /* borders — solid, visible */
  --accent:    #1F5C3A;  /* forest green — primary buttons, links, selected */
  --accent-ink:#FFFFFF;  /* text on accent */
  --warn:      #B5431E;  /* overdue / danger — burnt orange */
  --ok:        #2F7D4F;  /* complete / success */
  --soft:      #DCE8DF;  /* pale green highlight */
}
```
- Borders: **2px solid `--line`** on cards and inputs. Corners: **0px** (square).
- Headings: uppercase, letter-spacing 0.04em, bold.
- Buttons: solid fill, square, border 2px. Hover = invert colours (no glow, no shadow).
- Shadows: none, or a hard offset shadow `4px 4px 0 var(--line)` on the main card only.

## Direction 2 — "Swiss Paper" (off-white + ink + one signal colour)
Mood: a printed timetable / Swiss poster. Clean grid, lots of white space, typography does the work.

```css
:root {
  --bg:        #F7F5F0;  /* paper */
  --surface:   #FFFFFF;
  --ink:       #111111;
  --muted:     #6B6B6B;
  --line:      #D9D6CE;  /* hairline borders */
  --accent:    #D7261E;  /* signal red */
  --accent-ink:#FFFFFF;
  --warn:      #D7261E;
  --ok:        #1E6E4E;
  --soft:      #EFECE4;
}
```
- Borders: **1px solid `--line`**. Corners: **2px** max.
- Strong left-aligned grid. Large bold page titles, small uppercase labels.
- Accent colour used sparingly: primary action + one highlight per screen.
- No shadows.

---

## Shared rules (both directions)

**Fonts** (system fonts only — no Google Fonts, the network is firewalled):
```css
--font-sans: ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
--font-mono: ui-monospace, "SF Mono", Menlo, Consolas, monospace;
```
- Body 15–16px, line-height 1.5. Page title 28–32px bold. Labels 12px uppercase, letter-spacing 0.06em.
- Numbers, dates, IDs and counts in `--font-mono`.

**Spacing scale:** only 4, 8, 12, 16, 24, 32, 48 px. Page padding 24px (16px on phones).

**Layout:**
- Max content width ~1100px. Left-aligned, not everything centred.
- Navigation: a simple top bar or left sidebar with text labels.
- Lists are dense tables/rows, not giant cards with lots of padding.

**States:**
- Every list has an empty state: one sentence saying what to do next + the primary button.
- Loading: simple text "Loading…" or a thin bar. No spinners in the middle of the screen.
- Errors: inline, in `--warn`, saying what went wrong in plain words.
- Colour is never the only signal: overdue = `--warn` colour **plus** a text label/icon.
- Visible keyboard focus: `outline: 2px solid var(--accent); outline-offset: 2px;`

**Dark mode** (only if time allows): swap `--bg`/`--surface` to dark greys (#121412 / #1B1E1B), `--ink` to #ECEDE8, keep accent.

## BANNED (the default "AI look")
- Purple, violet, indigo, or blue-purple gradients
- Any gradient backgrounds or gradient text
- Glassmorphism / `backdrop-blur` / translucent cards
- Big soft drop shadows, glowing buttons
- Large rounded corners (`rounded-xl`, `rounded-2xl`, pill buttons everywhere)
- Emoji used as icons
- Centred hero section with a big headline and blob shapes

## Using these tokens in Tailwind (Stack A)
Put this in `src/app/globals.css` (Tailwind v4):
```css
@import "tailwindcss";

@theme {
  --color-bg: #EEEDE8;
  --color-surface: #FFFFFF;
  --color-ink: #161A16;
  --color-muted: #5E655E;
  --color-line: #161A16;
  --color-accent: #1F5C3A;
  --color-accent-ink: #FFFFFF;
  --color-warn: #B5431E;
  --color-ok: #2F7D4F;
  --color-soft: #DCE8DF;
  --font-sans: ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
  --font-mono: ui-monospace, "SF Mono", Menlo, Consolas, monospace;
  --radius-DEFAULT: 0px;
}

body { background: var(--color-bg); color: var(--color-ink); font-family: var(--font-sans); }
```
Then classes like `bg-accent`, `text-muted`, `border-line`, `font-mono` work. (Swap the hex values if you chose Direction 2.)

## Using these tokens in plain CSS (Stack B)
Copy the `:root { ... }` block into `static/style.css` and use `var(--accent)` etc.
