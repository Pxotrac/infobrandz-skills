---
name: wp-angles
description: "Phase 3 of the whitepaper prospect pipeline. Reads qualifying companies from the Master tab where Status = 'Angle needed', calls Claude API to generate three outreach angles per company, and writes the angles back to columns N, O, P. Updates status to 'Angle ready' when done. This skill is triggered by n8n after wp-research completes. Use when the user says 'generate angles', 'run wp-angles', 'write outreach angles for the whitepaper sheet', or when n8n triggers this skill after research completes."
---

## PURPOSE
Read rows from the Master tab where Column R = "Angle needed".
For each row, call the Claude API with that company's research data.
Write the three returned angles to Columns N, O, P.
Update Column R to "Angle ready".
This triggers n8n to send the row to RocketReach for email finding.

## SHEET
URL: https://docs.google.com/spreadsheets/d/12SapJRIxDn0HKhZyWWSO3qZgmwZvGiWHmQ7nsy2jpb8/
Tab: Whitepaper (Master tab — first tab)

Read columns:
A — Company Name
B — Website URL
C — Geography
D — Employee Size
E — Industry
F — White Paper Count
G — Most Recent White Paper
H — Resources Page URL
I — Hiring signal
J — Recent News
K — Decision Maker Name
L — Decision Maker Title
M — Person LinkedIn URL

Write columns:
N — Outreach Angle Idea 1
O — Outreach Angle Idea 2
P — Outreach Angle Idea 3
R — Status (update to "Angle ready" when angles are written)

## EXECUTION

Open the Master tab in the browser.
Scan Column R for rows where value = "Angle needed".
Process each such row one at a time, top to bottom.

For each row:
1. Read all data from Columns A through M for that row
2. Call Claude API with that data (see API CALL below)
3. Parse the three angles from the JSON response
4. Write Angle 1 → Column N, Angle 2 → Column O, Angle 3 → Column P
5. Write "Angle ready" → Column R
6. Move to the next row

## API CALL

Make an HTTP POST request to the Anthropic API:

Endpoint: https://api.anthropic.com/v1/messages

Headers:
- x-api-key: [Anthropic API key from settings]
- anthropic-version: 2023-06-01
- content-type: application/json

Body:
{
  "model": "claude-sonnet-4-6",
  "max_tokens": 1000,
  "messages": [
    {
      "role": "user",
      "content": "You are writing outreach angles for Vikas Agrawal, founder of InfoBrandz — a flat-fee creative design subscription ($5,000/month) used by B2B companies for white papers, reports, decks, and marketing assets.\n\nCompany data:\nCompany: [Column A]\nWebsite: [Column B]\nGeography: [Column C]\nEmployee Size: [Column D]\nIndustry: [Column E]\nWhite Paper Count: [Column F]\nMost Recent White Paper: [Column G]\nResources Page: [Column H]\nHiring Signal: [Column I]\nRecent News: [Column J]\nDecision Maker: [Column K], [Column L]\n\nGenerate exactly THREE outreach angles. Return ONLY valid JSON, no preamble, no markdown.\n\nAngle 1 — THE CONTENT-DESIGN GAP\nLens: their research is strong but the visual design does not match it.\nAnchor on the most recent white paper title if known.\nShape: name the specific paper, contrast its substance with production quality, offer to close that gap.\nIf no title known, anchor on paper count instead.\n\nAngle 2 — SCALE WITHOUT A FULL-TIME HIRE\nLens: capacity and cost. A design subscription covers the white paper and collateral load for less than a salaried hire.\nIf hiring a design/content/marketing role: reference that open role directly.\nIf not hiring: anchor on team size.\n\nAngle 3 — REPURPOSE AND EXTEND REACH\nLens: ROI and distribution. One white paper becomes a whole campaign.\nAnchor on recent news or funding if known.\nIf no news: anchor on publishing cadence.\n\nRules for all angles:\n- 1-2 sentences each\n- Specific to this company's actual data\n- Peer to peer tone, not vendor to buyer\n- No exclamation marks\n- No corporate language\n- Each angle must be distinct — do not repeat the same pitch three ways\n\nReturn this exact JSON structure:\n{\"angle1\": \"...\", \"angle2\": \"...\", \"angle3\": \"...\"}"
    }
  ]
}

Replace [Column A], [Column B] etc. with the actual cell values for that row.

## PARSING THE RESPONSE

The API returns a JSON response. Extract the content from:
response.content[0].text

Parse that text as JSON to get angle1, angle2, angle3.

If JSON parsing fails — write "ANGLE GENERATION FAILED" to Column N and skip Columns O and P.
Write "Angle error — retry" to Column R.
Move to the next row. Do not stop the session.

## WRITE IMMEDIATELY AFTER EACH ROW
Do not batch. Write angles for one company, then move to the next.

Navigate to the correct row in the sheet.
Write angle1 → Column N
Write angle2 → Column O
Write angle3 → Column P
Write "Angle ready" → Column R

## TIMING
- Wait 3–5 seconds between API calls to avoid rate limiting
- No questions to the user mid-session
- No check-ins

## WHEN DONE
After all "Angle needed" rows are processed:
Write "ANGLES COMPLETE" to cell C1 of the Master tab.
Stop. n8n watches this cell and triggers the RocketReach email finding flow.

## ERROR HANDLING
- API call fails → retry once with 10 second wait → if still fails write "ANGLE GENERATION FAILED" to Column N, "Angle error — retry" to Column R, continue to next row
- Row has blank Company Name → skip silently
- Column R already says "Angle ready" → skip (already processed)
