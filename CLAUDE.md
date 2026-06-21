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
napkin/index.html        # Hub: fallback index of the whole suite (static, no JS)
PRODUCT.md               # Strategic context (impeccable)
```

One folder per deployable. Each folder is a **separate Cloudflare Pages
project** mapped to its own subdomain (see Deployment). There is no shared
bundle — each page is standalone.

## The hub (`napkin/`)

A **fallback index**, not a front door. It exists for someone who lands on the
bare `napkin.florinpopa.dev` (a lost bookmark, a typed guess) — drop-ins from
external links still go straight to a calculator subdomain and never see it.

- It is the switcher rendered **static**: a plain `<ul>` of every calculator, no
  pill, no `⌘K`, no search. The page *is* the directory, so an interactive
  switcher on it would be circular.
- **No JS.** It has zero interactivity, so it carries no `CALCULATOR_REGISTRY`
  array — adding a calculator means adding one `<li>`. (A JS array + render loop
  would be dead ceremony and would render blank without JS, defeating a page
  whose only job is to work *whenever* someone lands there.)
- It is **not** an entry in any `CALCULATOR_REGISTRY`; the switcher lists the
  calculators directly, so a "see all" row pointing back at the index would be
  redundant.

## The switcher (cross-calculator navigation)

Every page carries an identical command-palette switcher (the `.sw-*` CSS block
+ the trailing `<script>` IIFE). It opens via the visible pill at the top, the
`/` key, or `⌘K`/`Ctrl+K`, and on touch becomes a top-anchored sheet (no
autofocus, so the list stays tappable).

- **Inline registry**: the list of calculators lives in a `CALCULATOR_REGISTRY`
  array duplicated in every calculator page. Each entry is
  `{ id, name, desc, url }` with an **absolute** `url` (calculators are on
  different origins). (The hub mirrors the same list as static markup — see
  "The hub" above.)
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
2. Append the new entry to the `CALCULATOR_REGISTRY` array **in every calculator
   page** (inline registry — there is no single source of truth by design), and
   add a matching `<li>` row to `napkin/index.html`.
3. Create a new Cloudflare Pages project (see Deployment for the manual steps)
   with root directory `<new-calc>/` and map its subdomain.

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
- **One Pages project per folder**, each with its **root directory** set to that
  folder (e.g. `time-value/`, `bet-sizing/`, `napkin/`)
- Build command: `exit 0` (static files, no build); output directory: `/`
- Subdomains (canonical URLs; keep in sync with each page's
  `CALCULATOR_REGISTRY` and the hub list):
  - Time Value → `tvc.florinpopa.dev`
  - Bet Sizing → `bet-sizing.florinpopa.dev` *(set up when deploying)*
  - Hub → `napkin.florinpopa.dev` *(set up when deploying)*
- ⚠️ Restructure note: Time Value moved from repo root to `time-value/`. Its
  existing Pages project must have its **root directory updated to `time-value/`**
  or the deploy will 404.

### Provisioning is manual — on purpose

We considered automating subdomain provisioning (Terraform / a CLI script) and
**chose not to**: there are only a handful of projects, they change rarely, and
with GitHub integration the GitHub↔Cloudflare OAuth link is dashboard-only
anyway, so neither Terraform nor `wrangler` can own the whole lifecycle. Ran
through Time Value, automation lost to clicking by an order of magnitude. The
six dashboard steps per project (do them by hand):

1. **Create a Pages project**, connect this GitHub repo.
2. Set **root directory** to the folder (`time-value/`, `bet-sizing/`, `napkin/`).
3. **Build command** `exit 0`.
4. **Output directory** `/`.
5. **Add a custom domain** (e.g. `napkin.florinpopa.dev`). Because the
   `florinpopa.dev` zone is on the **same** Cloudflare account, the `CNAME` is
   created automatically — no separate DNS step.
6. Wait for the first deploy, then confirm the subdomain resolves.

## Code Style

- Vanilla JS only
- Inline CSS and JS in each HTML file
- No external dependencies
