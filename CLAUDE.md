# CLAUDE.md

## Project Overview

A small **suite** of single-purpose decision calculators. Each calculator is its
own self-contained page, deployed to its own subdomain, so it can be promoted
separately. A shared switcher ties them together as a discoverable family.

Calculators today:
- **Time Value** (`time-value/`) — "How much can I spend making a task more
  efficient before I'm spending more than I'd generate?"
- **Bet Sizing** (`bet-sizing/`) — Kelly-criterion stake sizing.

## Architecture

```
time-value/index.html    # Time Value calculator (HTML + CSS + JS, no build step)
bet-sizing/index.html    # Bet Sizing calculator (HTML + CSS + JS, no build step)
PRODUCT.md               # Strategic context (impeccable)
```

One folder per calculator. Each folder is a **separate Cloudflare Pages
project** mapped to its own subdomain (see Deployment). There is no shared
bundle — each page is standalone.

## The switcher (cross-calculator navigation)

Every page carries an identical command-palette switcher (the `.sw-*` CSS block
+ the trailing `<script>` IIFE). It opens via the visible pill at the top, the
`/` key, or `⌘K`/`Ctrl+K`, and on touch becomes a top-anchored sheet (no
autofocus, so the list stays tappable).

- **Inline registry**: the list of calculators lives in a `CALCS` array
  duplicated in every page. Each entry is `{ id, name, desc, url }` with an
  **absolute** `url` (calculators are on different origins).
- **`CURRENT`**: a per-page constant naming which calculator this page is. The
  current entry renders as "you're here" (not a link); the rest are `<a>` links.
- **Per-origin state**: `localStorage` does not cross subdomains (and `*.pages.dev`
  is on the Public Suffix List, so cookies can't either). Nothing is shared today;
  if a global value is ever needed, use custom-domain subdomains + a parent-domain
  cookie.

### Adding a calculator

1. Create `<new-calc>/index.html`, following an existing page's structure
   (shared `.sw-*` CSS, the `.sw-bar` markup with the right `sw-cur` label, and
   the switcher `<script>` with `CURRENT = "<new-calc>"`).
2. Append the new entry to the `CALCS` array **in every page** (inline registry —
   there is no single source of truth by design).
3. Create a new Cloudflare Pages project with root directory `<new-calc>/` and
   map its subdomain.

## Key Decisions

- **One file per calculator**: No framework, no dependencies, no build step
- **One subdomain per calculator**: independent deploys + separate promotion
- **Inline registry over shared JSON**: zero deps / no CORS; cost is editing
  every page when the roster changes (rare)
- **localStorage**: inputs stored locally, never in URL (privacy); per-origin
- **Currency symbol**: `¤` (generic currency sign) - neutral, universal
- **Absurd cells**: Grayed out when value < 1¤ or time exceeds half of working hours

## Formula

```
Justified Spend = (Annual Income / 2,080 working hours) × Time Freed Per Year
```

Where `Time Freed Per Year = seconds saved × occurrences per year`

## Deployment

- Hosted on Cloudflare Pages with GitHub integration
- **One Pages project per calculator**, each with its **root directory** set to
  that calculator's folder (e.g. `time-value/`, `bet-sizing/`)
- Build command: `exit 0` (static files, no build); output directory: `/`
- Subdomains (canonical URLs; keep in sync with each page's `CALCS` registry):
  - Time Value → `tvc.florinpopa.dev`
  - Bet Sizing → `bet-sizing.florinpopa.dev` *(set up when deploying)*
- ⚠️ Restructure note: Time Value moved from repo root to `time-value/`. Its
  existing Pages project must have its **root directory updated to `time-value/`**
  or the deploy will 404.

## Code Style

- Vanilla JS only
- Inline CSS and JS in each HTML file
- No external dependencies
