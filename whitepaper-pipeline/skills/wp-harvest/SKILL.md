---
name: wp-harvest
description: "Phase 1 of the whitepaper prospect pipeline. Harvests company names that publish white papers and writes them to the Staging tab of the InfoBrandz Whitepaper Discovery Google Sheet. Does NOT research companies — names and URLs only. Triggers n8n webhook when done to launch wp-research. Use when the user says 'run whitepaper harvest', 'harvest whitepaper companies', 'start the whitepaper pipeline', or 'fill the staging tab'."
---

## PURPOSE
Collect company names and URLs that appear to publish white papers.
Write them immediately to the Staging tab of the Google Sheet.
Do not research them. Do not gate them. Names and URLs only.
Stop at 100 names or when the user says stop.
When done, write "HARVEST COMPLETE" to cell A1 of the Staging tab to trigger n8n.

## SHEET
URL: https://docs.google.com/spreadsheets/d/1vyQMI4k_rS3zT5nq29hfIHNu6LDN1eNoW9bPzMfBNrI/
Tab:  Staging tab

Staging tab columns:
A — Company Name
B — Website URL
C — Source (write "web-search" for all rows this skill writes)
D — Gate Result (leave blank — wp-research fills this)
E — Research Status (leave blank — wp-research fills this)

## HARD SESSION CAP
Stop writing after 100 companies regardless of how many names remain.
After the 100th row is written, write "HARVEST COMPLETE" to cell A1 of the Staging tab.
Do not continue past 100. Do not ask the user. Just stop and write the trigger.

## EXECUTION

Open the Google Sheet Staging tab in the browser before starting.
Leave it open for the full session so each write is fast.

Use web_search only — not the browser — to find companies.
The browser is only for writing to the sheet.

Run searches in varied batches. Do not repeat the same query twice.
Rotate across: industry terms, white paper synonyms, year, region, company size signals.

Example search patterns to rotate through:
- "[industry] company white paper download 2025"
- "[industry] B2B SaaS research report gated PDF 2024"
- "[industry] "resources" OR "white papers" site:*.com"
- "[vertical] tech company whitepaper library"
- "fintech OR healthtech OR cybersecurity white paper published 2025"
- "B2B SaaS "download our white paper" 2025"
- "[industry] benchmark report download 2024 2025"

Industries to rotate through: B2B SaaS, AI/ML, Fintech, Healthtech,
Cybersecurity, HR Tech, RegTech, LegalTech, DevTools, Supply Chain,
Enterprise Tech, Data Analytics, InsurTech, MarTech.

For each search result:
- Look for company names in titles and URLs that appear to be the author/publisher
- Note the URL of their resources or white papers page if visible
- Skip: analyst firms, consulting giants, news sites, aggregators, universities
- Capture only companies that appear to be the author/publisher

## DEDUPLICATION BEFORE WRITING
Before writing each company, scan Column A of the Staging tab for an existing match.
Also check Column A of the Master tab (first tab) for an existing match.
If the company already exists in either tab — skip it, do not write.

## WRITE EACH NAME IMMEDIATELY
Do not batch. Write each company as soon as you identify it.
Do not wait until you have several.

Do NOT use the Tab key to move between columns. Tab does not work reliably
here and causes all data to pile into a single cell (column A). Instead,
write each field to its EXACT cell address using the Name Box, one cell
at a time.

For a company going into the next empty row N of the Staging tab:
- Name Box → type A{N} → Enter → type Company Name → Enter
- Name Box → type B{N} → Enter → type Website URL → Enter
- Name Box → type C{N} → Enter → type web-search → Enter

Always one cell at a time via the Name Box. Never use Tab.
After writing, click cell B{N} and confirm it shows the URL, not empty.
If B{N} is empty, the write failed — retry that row.

## TIMING
- Wait 3–5 seconds between web searches
- No questions to the user mid-session
- No check-ins, no progress reports, no "should I continue?" — just run

## WHEN DONE
After the 100th company is written (or if the user says stop):
1. Navigate to cell A1 of the Staging tab
2. Type exactly: HARVEST COMPLETE
3. Press Enter
4. Stop. Do not start researching. Do not open any other tab.

This "HARVEST COMPLETE" value in A1 is the n8n trigger.
n8n watches this cell and launches wp-research automatically when it sees this value.

## ERROR HANDLING
- Sheet won't load → retry once, then report to user and stop
- Search returns no usable results → run a different query, do not stop
- Company name is ambiguous → skip it, move to next result
