# Calculators

A small suite of single-purpose decision calculators. Each one is a standalone,
dependency-free HTML page deployed to its own subdomain, tied together by a
shared command-palette switcher (open with the top pill, `/`, or `⌘K`).

| Calculator | Folder | Question | Live |
|---|---|---|---|
| Time Value | `time-value/` | How much can you spend to save time? | [tv.napkin.florinpopa.dev](https://tv.napkin.florinpopa.dev) |
| Kelly Stake | `kelly/` | How much should you bet? (Kelly) | kelly.napkin.florinpopa.dev |
| Expected Value | `ev/` | What is a bet worth, in cash? | ev.napkin.florinpopa.dev |

Plus a hub at `napkin.florinpopa.dev` (folder `napkin/`) — a plain fallback index
of the suite for anyone who lands there directly.

## Structure

One folder per calculator, each its own Cloudflare Pages project / subdomain. No
build step, no dependencies, inline CSS + JS. See [CLAUDE.md](CLAUDE.md) for the
switcher registry and how to add a calculator.

## Time Value formula

```
Value = (Annual Income / 2,080 hours) × Time Saved × Occurrences Per Year
```

Inspired by [XKCD 1205: Is It Worth the Time?](https://xkcd.com/1205/)
