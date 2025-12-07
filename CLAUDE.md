# CLAUDE.md

## Project Overview

Time Value Calculator - a single-file web tool that helps answer "How much can I spend making a task more efficient before I'm spending more than I'd generate?"

## Architecture

```
index.html          # Everything: HTML, CSS, JS (no build step)
```

## Key Decisions

- **Single file**: No framework, no dependencies, no build step
- **localStorage**: Salary stored locally, never in URL (privacy)
- **Currency symbol**: `¤` (generic currency sign) - neutral, universal
- **Absurd cells**: Grayed out when value < 1¤ or time exceeds half of working hours

## Formula

```
Justified Spend = (Annual Income / 2,080 working hours) × Time Freed Per Year
```

Where `Time Freed Per Year = seconds saved × occurrences per year`

## Deployment

- Hosted on Cloudflare Pages with GitHub integration
- Domain: tvc.florinpopa.dev
- Build command: `exit 0` (static file, no build)
- Output directory: `/`

## Code Style

- Vanilla JS only
- Inline CSS and JS in the HTML file
- No external dependencies
