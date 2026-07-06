# Whitepaper Pipeline Plugin

Three-phase whitepaper prospect discovery pipeline for InfoBrandz.

## Skills

| Skill | What it does | Cap | Output |
|---|---|---|---|
| wp-harvest | Finds company names via web search, writes to Staging tab | 100 names | Staging tab rows + "HARVEST COMPLETE" in Staging A1 |
| wp-research | Gates and researches each company, writes full rows to Master tab | 15 companies | Master tab rows + "RESEARCH COMPLETE" in Staging B1 |
| wp-angles | Calls Claude API per row, writes 3 angles to Master tab | All "Angle needed" rows | Angles in columns N/O/P + "ANGLES COMPLETE" in Master C1 |

## How skills chain via n8n

Each skill writes a trigger value to a specific sheet cell when done.
n8n watches those cells and launches the next skill automatically.

### Trigger cells

| Cell | Value | n8n action |
|---|---|---|
| Staging tab — A1 | HARVEST COMPLETE | Launch wp-research on Mac Mini |
| Staging tab — B1 | RESEARCH COMPLETE | Launch wp-angles on Mac Mini |
| Master tab — C1 | ANGLES COMPLETE | Launch RocketReach skill on Mac Mini |
| Master tab — D1 | (set by RocketReach skill) | Push to Instantly campaign |

### n8n workflow setup (Kanishka)

1. Install n8n on Mac Mini (self-hosted, free)
2. Create a workflow with a Google Sheets trigger watching each cell above
3. On trigger: send a local webhook or run a shell command that opens the next skill in Cowork
4. Each workflow node polls its trigger cell every 5 minutes

## Sheet structure

Google Sheet ID: 12SapJRIxDn0HKhZyWWSO3qZgmwZvGiWHmQ7nsy2jpb8

### Staging tab (second tab)
| Column | Field | Written by |
|---|---|---|
| A | Company Name | wp-harvest |
| B | Website URL | wp-harvest |
| C | Source | wp-harvest |
| D | Gate Result | wp-research |
| E | Research Status | wp-research |

### Master tab — "Whitepaper" (first tab)
| Column | Field | Written by |
|---|---|---|
| A | Company Name | wp-research |
| B | Website URL | wp-research |
| C | Geography | wp-research |
| D | Employee Size (clean number) | wp-research |
| E | Industry | wp-research |
| F | White Paper Count | wp-research |
| G | Most Recent White Paper | wp-research |
| H | Resources Page URL | wp-research |
| I | Hiring signal | wp-research |
| J | Recent News | wp-research |
| K | Decision Maker Name | wp-research |
| L | Decision Maker Title | wp-research |
| M | Person LinkedIn URL | wp-research |
| N | Outreach Angle 1 | wp-angles |
| O | Outreach Angle 2 | wp-angles |
| P | Outreach Angle 3 | wp-angles |
| Q | Notes | wp-research |
| R | Status | wp-research + wp-angles |

### Status flow
Angle needed → Angle ready → Email found → In sequence

## RocketReach skill
Already built separately. Triggered by n8n when Master tab Column R = "Angle ready".
Finds email via LinkedIn URL, writes to Column S.
Updates Column R to "Email found".

## What does not change from the original skill
- All gate logic (identity, employee size, acquisition check)
- All research fields and what gets written to each column
- Whitepaper verification logic
- Decision maker cascade (CMO → VP → Head → CEO/Founder)
- Evidence rule (no invented data, unknown over blank guesses)
- Timing rules between searches
- Outreach angle content and framing (same three lenses)
