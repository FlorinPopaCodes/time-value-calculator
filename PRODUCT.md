# Product

## Register

product

## Users

People making a quick "is this worth it?" decision — whether to automate a task,
how big to bet, what to charge. Technically literate, often dropping in from a
link (xkcd 1205, a tweet, a bookmark) for one specific answer, then leaving.
Context: a browser tab opened mid-thought, not a daily-driver app.

## Product Purpose

A small collection of single-purpose decision calculators (Time Value, Bet
Sizing, and growing toward 5-6). Each answers one question fast. As the set
grows, users need a quiet way to move between tools without the site turning into
an app. Success = a user finds the calculator they want in seconds, gets the
answer, and the interface never gets in the way.

## Brand Personality

Understated, utilitarian, quietly nerdy. xkcd-adjacent wit carried by restraint,
not decoration: the generic `¤` currency sign, monospace numbers, a default
salary of 65536. Trustworthy because it's plain. The tool disappears into the task.

## Anti-references

- Generic SaaS dashboards — no sidebar app-shell, no hero-metric cards, no
  enterprise-admin chrome.
- Card-grid landings — no repeated icon + heading + text tiles.
- Heavy or animated marketing pages — no orchestrated page-load motion, no
  glassmorphism, no decorative flourish.

## Design Principles

- **One screen, one tool.** Deep focus on a single calculator; the hub is
  navigation, never a destination that competes for attention.
- **The switcher is furniture.** Getting between calculators should be quiet and
  obvious, never the loudest thing on the page.
- **Plain is the brand.** Restraint and a couple of deliberate nerd-jokes do the
  identity work; chrome and ornament don't.
- **Respect the drop-in.** Most visits are one calculator deep from an external
  link. Don't tax that path with a lobby.

## Accessibility & Inclusion

Keyboard-navigable switching, visible focus states, motion limited to short
state-change transitions with a `prefers-reduced-motion` fallback. Body text and
controls meet WCAG AA contrast (≥4.5:1).
