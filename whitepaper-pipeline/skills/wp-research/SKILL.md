---
name: wp-research
description: "Phase 2 of the whitepaper prospect pipeline. Reads company names from the Staging tab, runs identity and employee size gates, verifies whitepaper publishing, collects all research fields, and writes qualifying prospects to the Master tab (first tab) of the InfoBrandz Whitepaper Discovery Google Sheet. Does NOT generate outreach angles — writes PENDING to angle columns so n8n can trigger wp-angles via Claude API. Hard cap: 15 companies per session. Use when the user says 'run whitepaper research', 'process the staging tab', 'research whitepaper companies', or when n8n triggers this skill after harvest completes."
---

## PURPOSE
Read company names from the Staging tab.
Gate each company (identity + employee size + acquisition check).
Research each company that passes.
Write the full row to the Master tab immediately after each company.
Write PENDING to angle columns — angles are generated separately via API.
Stop after 15 companies are written to the Master tab.
When done, write "RESEARCH COMPLETE" to cell B1 of the Staging tab to trigger n8n.

## SHEETS
URL: https://docs.google.com/spreadsheets/d/12SapJRIxDn0HKhZyWWSO3qZgmwZvGiWHmQ7nsy2jpb8/

SOURCE tab: Staging (second tab)
Read from Column A (Company Name) and Column B (Website URL).
Only process rows where Column D (Gate Result) is blank.
After gating each company, write the result to Column D of that Staging row:
- Write "PASSED" if company passes all gates and gets written to Master tab
- Write "REJECTED — [reason]" if company fails any gate

OUTPUT tab: Master (first tab, named "Whitepaper")
Columns:
A — Company Name
B — Website URL
C — Geography
D — Employee Size (clean number only — e.g. "58" not "~58 (51-200)")
E — Industry
F — White Paper Count (how many found)
G — Most Recent White Paper (title + approximate date)
H — Resources Page URL
I — Hiring (Yes/No + roles if yes)
J — Recent News
K — Decision Maker Name
L — Decision Maker Title
M — Person LinkedIn URL
N — Outreach Angle Idea 1 (write: PENDING)
O — Outreach Angle Idea 2 (write: PENDING)
P — Outreach Angle Idea 3 (write: PENDING)
Q — Notes
R — Status (write: "Angle needed" — this is the n8n trigger for wp-angles)

## HARD SESSION CAP
Stop after writing 15 companies to the Master tab.
After the 15th row is written, write "RESEARCH COMPLETE" to cell B1 of the Staging tab.
This triggers n8n to launch the angle generation flow.
Do not continue past 15. Do not ask the user. Just stop and write the trigger.

## EXECUTION ORDER
Open both tabs in the browser before starting — Staging tab and Master tab.
Keep both open throughout the session.

Process Staging tab rows from top to bottom.
Skip any row where Column D already has a value (already processed).
Process one company fully before moving to the next.

For each company:
1. Run all gates in order (cheapest first)
2. If any gate fails → write rejection reason to Staging Column D → move to next
3. If all gates pass → run full data collection → write to Master tab → mark Staging Column D as PASSED

Use web_search for all lookups. Not the browser.

================================================
GATE 1 — IDENTITY VERIFICATION
================================================
Search: "[Company Name] official website B2B"

REJECT if:
- Multiple unrelated companies share the same name
- Solo operator or individual consultant
- Cannot confirm a single clear entity

Only proceed if one clear company is confirmed.

================================================
GATE 2 — EMPLOYEE SIZE
================================================
Search: "how many employees does [Company Name] have"

ACCEPT: 2–200 employees inclusive
REJECT: under 2 OR 201 or more
REJECT: unknown after one search (do not guess)

Write the clean number to Master Column D — e.g. "58" not "~58 (51-200)".
When sources disagree, use the highest credible number.
If that number exceeds 200 — REJECT. No exceptions. No "borderline" category.

================================================
ACQUISITION CHECK
================================================
Search: "[Company Name] acquired OR acquisition 2023 OR 2024 OR 2025 OR 2026"

REJECT if fully acquired and absorbed into a larger parent.
Subsidiaries still operating independently are OK — note parent in Column Q.

================================================
DATA COLLECTION (only for companies that passed all gates)
================================================

WHITE PAPER VERIFICATION
Search: "[Company Name] white paper download 2024 OR 2025 OR 2026"

If none found → REJECT (not an active white paper publisher).
If all papers are 3+ years old → REJECT (not an active publisher).

Capture:
- Approximate count (1–2, 3–5, 6+) → Column F
- Title and date of most recent paper → Column G
- Resources page URL if findable → Column H

GEOGRAPHY
From the identity and size searches already run, identify HQ location.
Write country and city if known → Column C.
Do not run an extra search for geography — use what the previous searches returned.

INDUSTRY
From searches already run, identify the company's primary industry.
Write the specific vertical → Column E.
Examples: "Cybersecurity SaaS", "HR Tech", "Fintech", "Healthtech", "DevTools"

HIRING SIGNAL
Search: "is [Company Name] hiring 2026 designer OR content OR marketing"

Write Yes/No and role names if yes → Column I.

RECENT NEWS
Search: "[Company Name] news 2025 2026 funding product launch"

Write most relevant event with approximate date → Column J.

DECISION MAKER CASCADE
Work down this order. Stop at the first tier that yields a confident name.

1. CMO: search "who is the CMO of [Company Name]"
2. VP of Marketing (only if no CMO): search "VP Marketing [Company Name]"
3. Head of Marketing (only if no VP): search "Head of Marketing [Company Name]"
4. CEO / Founder (only if none above): search "[Company Name] CEO OR founder"

Write name → Column K
Write actual title found → Column L (e.g. "Co-founder & CEO" not just "CEO")
Write "unknown" if no confident name found after cascade. Never guess.

PERSON LINKEDIN URL
Search: "[Person Name] [Company Name] LinkedIn"

Accept only a personal /in/ profile matching both name AND company.
A company page (linkedin.com/company/...) is NOT acceptable here.
Write the /in/ URL → Column M
Write "unknown" if no confident match after one search. Never construct a slug.

================================================
WRITE THIS ROW NOW
================================================
Write immediately after data collection is complete.
Do not queue. Do not wait. Write one company, then move to the next.

Write to the next empty row in the Master tab:
1. Click Name Box, type next empty row cell A address, press Enter
2. Tab through columns A to R in order
3. Columns N, O, P: write PENDING for all three
4. Column R: write "Angle needed"
5. Press Enter to commit

Then immediately go back to the Staging tab and write to that company's Column D:
- "PASSED" if written to Master
- "REJECTED — [reason]" if failed

DEDUPLICATION: Before writing to Master, scan Column A of Master for existing match.
If already present — skip the write, mark Staging Column D as "DUPLICATE — skipped".

Screenshot every 5th row written to confirm columns are correct.

================================================
EVIDENCE RULE
================================================
Every cell value must come from a real search result this session.
Write "unknown" if a search returns nothing usable.
Never invent counts, names, or locations.
A blank beats a fabricated fill.

================================================
TIMING
================================================
- Wait 3–5 seconds between web searches
- Wait 6–10 seconds between companies
- Pause 30 seconds every 10 companies
- No questions to the user mid-session

================================================
WHEN DONE
================================================
After the 15th company is written to Master tab (or user says stop):
1. Navigate to Staging tab cell B1
2. Type exactly: RESEARCH COMPLETE
3. Press Enter
4. Stop completely. Do not generate angles. Do not open anything else.

n8n watches cell B1 of the Staging tab.
When it sees "RESEARCH COMPLETE" it fires the angle generation flow
which calls Claude API separately for each row where Column R = "Angle needed".

================================================
ERROR HANDLING
================================================
- Sheet won't load → retry once, then report to user and stop
- Search returns nothing → write "unknown" and continue
- Gate result is ambiguous → default to REJECT (precision over recall)
- Row in Staging has no company name → skip it silently
