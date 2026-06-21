# Calculators

A small suite of single-purpose decision calculators. Each one is a standalone,
dependency-free HTML page deployed to its own subdomain, tied together by a
shared command-palette switcher (open with the top pill, `/`, or `⌘K`).

| Page | Folder | What it's for | Live |
|---|---|---|---|
| Time Value | `time-value/` | How much can you spend to save time? | [tv.napkin.florinpopa.dev](https://tv.napkin.florinpopa.dev) |
| Kelly Stake | `kelly/` | How much should you bet? (Kelly) | [kelly.napkin.florinpopa.dev](https://kelly.napkin.florinpopa.dev) |
| Expected Value | `ev/` | What is a bet worth, in cash? | [ev.napkin.florinpopa.dev](https://ev.napkin.florinpopa.dev) |
| Growth Curve | `compound/` | Where does your weekly growth rate land you? | [compound.napkin.florinpopa.dev](https://compound.napkin.florinpopa.dev) |
| Hub | `napkin/` | Plain fallback index of the suite | [napkin.florinpopa.dev](https://napkin.florinpopa.dev) |

## Structure

One folder per calculator, each its own Cloudflare Pages project / subdomain. No
build step, no dependencies, inline CSS + JS. See [CLAUDE.md](CLAUDE.md) for the
switcher registry and how to add a calculator.

## Time Value formula

```
Value = (Annual Income / 2,080 hours) × Time Saved × Occurrences Per Year
```

Inspired by [XKCD 1205: Is It Worth the Time?](https://xkcd.com/1205/)

## Growth Curve formula

```
Value = MRR × (1 + weekly rate) ^ weeks    (weeks = months × 52/12)
```

Inspired by Paul Graham's [Startup = Growth](https://paulgraham.com/growth.html)
