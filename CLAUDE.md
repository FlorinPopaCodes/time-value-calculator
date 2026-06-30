# CLAUDE.md

## Project Overview

A small **suite** of single-purpose decision calculators. Each calculator is its
own self-contained page, deployed to its own subdomain, so it can be promoted
separately. A shared switcher ties them together as a discoverable family.

Calculators today:
- **Time Value** (`time-value/`) — "How much can I spend making a task more
  efficient before I'm spending more than I'd generate?"
- **Kelly Stake** (`kelly/`) — Kelly-criterion stake sizing (bankroll + fraction).
- **Expected Value** (`ev/`) — expected value of a stake (`stake × edge`).
- **Growth Curve** (`growth/`) — how many times your MRR multiplies across a
  grid of weekly growth rates × horizons (`(1 + weekly rate) ^ weeks`). The
  multiple is starting-point-independent, so there's no input — it's a universal
  reference table, heat-shaded by magnitude.
- **Runway Split** (`runway/`) — for a founder running several experiments: how
  much of your runway to put behind *each* one before going all-in, without going
  bankrupt. Multi-outcome Kelly over a power-law outcome ladder
  (`f★ maximises Σ pᵢ·ln(1 + f·bᵢ)`, solved by bisection on `G'(f)`). The single
  input is **ambition** (bootstrap → moonshot), which interpolates between two
  anchor distributions so the per-outcome odds rebalance automatically; a ½-Kelly
  default (¼/⅓/½ toggle) tempers the size.

> The two betting calcs were split out of a single combined "Bet Sizing" page:
> Kelly answers *how much to bet*, EV answers *what a bet is worth*. They share
> the same probability × odds grid and `edge = p × (odds + 1) − 1`.

## Architecture

```
time-value/index.html    # Time Value calculator (HTML + CSS + JS, no build step)
kelly/index.html         # Kelly Stake calculator (HTML + CSS + JS, no build step)
ev/index.html            # Expected Value calculator (HTML + CSS + JS, no build step)
growth/index.html        # Growth Curve calculator (HTML + CSS + JS, no build step)
runway/index.html        # Runway Split calculator (multi-outcome Kelly, no build step)
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

Every page carries an identical switcher (the `.sw-*` CSS block + the trailing
`<script>` IIFE). It opens via the visible pill at the top, the `/` key, or
`⌘K`/`Ctrl+K`, and on touch becomes a top-anchored sheet.

- **Letters-only jump, no search box.** The open panel is a plain keyed list,
  not a search input: each sibling row shows a `kbd` badge with its jump key
  (the first letter of its name — `T`/`K`/`E`/`G` today, all unique), and pressing
  that letter navigates immediately. `Esc` closes. Rows stay real `<a>` links,
  so `Tab`+`Enter` (and screen readers) work natively without our own arrow/
  cursor handling. Keys are hidden on touch (no keyboard). With only a handful
  of calculators a search box was ceremony — this mirrors the static hub's
  "the page *is* the directory" simplicity.
- **Key collisions are out of scope** until the roster actually grows two
  calculators with the same first letter; `keyFor` is one line and can grow a
  fallback (e.g. position number) then.

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

Growth Curve shows the **multiple** weekly compounding yields across the grid:

```
Multiple = (1 + weekly rate) ^ weeks    (weeks = months × 52/12)
```

There's no MRR in the formula — the multiple is starting-point-independent, so
the page takes **no input**; it's a universal reference table, the same for
everyone. Each cell is heat-shaded on a log scale (pale `1×` → deepest brand-blue
at the explosive corner) so the bend in the curve is visible without reading
every number. Cells past **1e9× (a billion-fold)** show `1B×+` at the deepest
tint — the peak of the ramp, not a grayed-out dead cell. Unlike Time Value there
is no negligible gray; every positive weekly rate beats it, so the only extreme
is the explosive one.

Runway Split sizes a single bet against a **distribution** of outcomes, not a
binary win/lose, so it can't use Kelly's `b:1` grid:

```
f★ = argmax_f  Σ pᵢ · ln(1 + f · bᵢ)        (bᵢ = outcome multiple − 1)
```

solved by bisection on `G'(f) = Σ pᵢbᵢ / (1 + f·bᵢ) = 0`. Outcomes are a fixed
named ladder (`0× … 1000×`, each with a real-world exemplar); only their
probabilities move, by linear interpolation between a **modest** anchor
(`[35, 40, 18, 6.5, 0.5, 0]%`, soft failures + capped upside) and a **moonshot**
anchor (`[65, 18, 10, 5, 1.8, 0.2]%`, more zeros + a fat tail) keyed off one
"ambition" slider — so the odds always sum to 1 with nothing to hand-edit. The
displayed fraction is `f★ × {¼,⅓,½}` (½ default); `≈ 1/f★` is the number of shots
the runway splits into. The headline insight: **the bigger you swing, the smaller
each bet must be** — moonshots force diversification (≈12% per shot at bootstrap,
≈3.5% at moonshot, at ½-Kelly). A "plausible range" band sweeps the moonshot odds
±3× and is hidden when degenerate (full bootstrap has no tail to vary).

## Deployment

- Hosted on Cloudflare Pages with GitHub integration
- **One Pages project per folder**, each with its **root directory** set to that
  folder (e.g. `time-value/`, `kelly/`, `ev/`, `growth/`, `runway/`, `napkin/`)
- Build command: `exit 0` (static files, no build); output directory: `/`
- Subdomains (canonical URLs; keep in sync with each page's
  `CALCULATOR_REGISTRY` and the hub list). Every calculator is nested **under the
  hub's parent** (`*.napkin.florinpopa.dev`) so the suite reads as a family in the
  address bar; the hub itself sits at that parent apex:
  - Time Value → `tv.napkin.florinpopa.dev` *(live)*
  - Kelly Stake → `kelly.napkin.florinpopa.dev` *(set up when deploying)*
  - Expected Value → `ev.napkin.florinpopa.dev` *(set up when deploying)*
  - Growth Curve → `growth.napkin.florinpopa.dev` *(set up when deploying)*
  - Runway Split → `runway.napkin.florinpopa.dev` *(set up when deploying)*
  - Hub → `napkin.florinpopa.dev` *(set up when deploying)*
- ⚠️ Nesting note: hosts are **two levels deep** (`tv.napkin.…`, not `tv.…`).
  Cloudflare's free Universal SSL only covers `florinpopa.dev` + `*.florinpopa.dev`
  (one wildcard level), **not** `*.napkin.florinpopa.dev`. This works anyway
  because **Pages issues a dedicated cert per custom hostname** when you add it —
  verified live on `tv`. No Advanced Certificate Manager needed.
- ⚠️ Restructure note: hosts moved from flat `*.florinpopa.dev` (e.g. the old
  `tvc.florinpopa.dev`) to nested `*.napkin.florinpopa.dev`. Nothing was linked
  publicly at the time, so the old flat hosts were **cut, not redirected**. Folder
  names are unchanged (`time-value/`, `kelly/`, `ev/`) — only the custom domains
  moved. Each existing Pages project just needs its **custom domain** swapped.
- ⚠️ Split note: the combined `bet-sizing/` page was removed in favour of
  `kelly/` + `ev/`. If a `bet-sizing.florinpopa.dev` Pages project was ever
  created, delete it (or redirect it); create fresh projects for `kelly/` and
  `ev/`.

### Provisioning is manual — on purpose

We considered automating subdomain provisioning (Terraform / a CLI script) and
**chose not to**: there are only a handful of projects, they change rarely, and
with GitHub integration the GitHub↔Cloudflare OAuth link is dashboard-only
anyway, so neither Terraform nor `wrangler` can own the whole lifecycle. Ran
through Time Value, automation lost to clicking by an order of magnitude. The
six dashboard steps per project (do them by hand):

1. **Create a Pages project**, connect this GitHub repo.
2. Set **root directory** to the folder (`time-value/`, `kelly/`, `ev/`, `growth/`, `runway/`, `napkin/`).
3. **Build command** `exit 0`.
4. **Output directory** `/`.
5. **Add a custom domain** (e.g. `tv.napkin.florinpopa.dev`). Because the
   `florinpopa.dev` zone is on the **same** Cloudflare account, the nested `CNAME`
   is created automatically and Pages issues a per-host cert — no separate DNS or
   certificate step (see the Nesting note above).
6. Wait for the first deploy, then confirm the subdomain resolves.

## Code Style

- Vanilla JS only
- Inline CSS and JS in each HTML file
- No external dependencies
