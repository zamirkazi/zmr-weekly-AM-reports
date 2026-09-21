# ZMR Capital â Weekly AM Report Pipeline

## Implementation Guide & System Log

**Owner:** Zamir Kazi (zamir@zmrcapital.com)
**Portfolio:** 19 properties
**Created:** April 14, 2026
**Last Updated:** September 21, 2026

---

## 1. System Overview

This pipeline automatically generates a weekly asset management report for ZMR Capital's 19-property portfolio. It pulls data from Fireflies meeting transcripts, builds an HTML dashboard, deploys it to a live URL, archives structured data for future rollups, and distributes via Slack and email.

### Pipeline Flow

```
Fireflies Transcripts â Extract Data â Build HTML Dashboard â Deploy to GitHub Pages
                                                            â Archive JSON (weekly)
                                                            â Send Slack Summary
                                                            â Draft Gmail to Team
```

### Key URLs

- **Live Dashboard:** https://zamirkazi.github.io/zmr-weekly-AM-reports/
- **GitHub Repo:** https://github.com/zamirkazi/zmr-weekly-AM-reports
- **Slack Channel:** DM to Zamir (U01N25J7789)

---

## 2. Repository Structure

```
zmr-weekly-AM-reports/
âââ index.html                    # Current week's dashboard (deployed via GitHub Pages)
âââ archive/
â   âââ 2026-W15.json            # Weekly structured data archive
âââ IMPLEMENTATION_GUIDE.md       # This file
âââ README.md                     # Repo description
```

### GitHub Pages Configuration

- **Source:** main branch, root folder (/)
- **Custom domain:** None (using default github.io)
- **Build:** Static HTML, no build step required

---

## 3. Dashboard Specification

### File: `index.html`

Pure HTML/CSS, no external dependencies, fully self-contained.

**Sections:**
1. Portfolio Snapshot â high-level KPIs (occupancy, collections, delinquency)
2. Property Performance Table â 19 rows with status indicators (green/amber/red)
3. Portfolio Narrative â qualitative summary from meeting transcripts
4. Top 5 Risks â ranked risk items with severity indicators
5. What's Working â portfolio wins and positive trends
6. Asset Deep Dives â detailed cards for all 19 properties
7. Strategic Recommendations â forward-looking action items
8. Action Items by Owner â tasks assigned to specific team members
9. Coverage Gaps â properties not discussed in meetings that week

### CSS Variables

```css
--bg-primary: #fafaf7;
--green-ok: #10b981;
--amber-warn: #f59e0b;
--red-crit: #ef4444;
--green-dark: #047857;
```

### Design Constraints

- Max width: 900px, centered
- Responsive layout
- Print-friendly
- Status badges: green (performing), amber (watchlist), red (critical)

---

## 4. JSON Archive Schema

Each week's data is stored in `archive/YYYY-WNN.json` for future monthly, quarterly, biannual, and annual rollups.

```json
{
  "meta": {
    "week": "2026-W15",
    "weekEnding": "2026-04-10",
    "generatedAt": "2026-04-14T...",
    "meetingCount": 4,
    "meetingIds": ["id1", "id2", "..."]
  },
  "portfolioSnapshot": {
    "totalUnits": 3050,
    "avgOccupancy": 0.93,
    "avgCollections": 0.96,
    "avgDelinquency": 0.04,
    "propertiesGreen": 10,
    "propertiesAmber": 5,
    "propertiesRed": 3
  },
  "properties": [
    {
      "name": "Property Name",
      "market": "City, ST",
      "occupancy": 0.95,
      "collections": 0.97,
      "delinquency": 0.03,
      "status": "green|amber|red",
      "tier": "performing|watchlist|critical",
      "notes": "Key observations from meetings"
    }
  ],
  "risks": [
    {
      "rank": 1,
      "property": "Property Name",
      "description": "Risk description",
      "severity": "high|medium|low",
      "mitigation": "Planned response"
    }
  ],
  "actionItems": [
    {
      "owner": "Name",
      "task": "Description",
      "property": "Property Name",
      "priority": "critical|high|medium",
      "dueDate": "2026-04-18"
    }
  ],
  "wins": [
    {
      "property": "Property Name",
      "description": "What's working"
    }
  ]
}
```

### Rollup Strategy

| Period     | Source               | Aggregation |
|-----------|----------------------|-------------|
| Monthly   | 4â5 weekly JSONs     | Average KPIs, merge action items, trend occupancy/collections |
| Quarterly | 3 monthly rollups    | Trend analysis, YoY comparison |
| Biannual  | 2 quarterly rollups  | Strategic review, portfolio health trajectory |
| Annual    | 4 quarterly rollups  | Full year summary, board-ready metrics |

---

## 5. Team Distribution

### Email Recipients (Gmail Draft)

| Name            | Email                    | Role                |
|-----------------|--------------------------|---------------------|
| Zamir Kazi      | zamir@zmrcapital.com     | Principal           |
| Nicole          | nicole@zmrcapital.com    | Team                |
| Chip            | chip@zmrcapital.com      | Team                |
| Sid Martins     | sid@zmrcapital.com       | Team                |
| Mike Weiner     | mikew@zmrcapital.com     | Team                |
| Michael Regan   | mregan@zmrcapital.com    | Team                |
| Megan Burrows   | megan@zmrcapital.com     | Team                |

### Slack Distribution

- **Target:** Zamir's DM (User ID: U01N25J7789)
- **Format:** Detailed message with portfolio snapshot, risks, wins, and action items

---

## 6. Scheduled Automation

### Weekly Task Configuration

- **Schedule:** Every Monday at 9:00 AM local time
- **Cron Expression:** `0 9 * * 1`
- **Task ID:** `weekly-am-report`

### Task Prompt

The automated task should:

1. **Extract data** from Fireflies transcripts for the prior week (MondayâFriday)
   - Use `fireflies_get_transcripts` filtered to the past 7 days
   - Get summaries via `fireflies_get_summary` for each meeting
2. **Build the HTML dashboard** using the template structure in Section 3
3. **Deploy to GitHub Pages** by updating `index.html` in the repo
4. **Archive weekly data** as `archive/YYYY-WNN.json` following the schema in Section 4
5. **Send Slack message** to Zamir (U01N25J7789) with detailed summary
6. **Create Gmail draft** to all 7 team members with live dashboard link
7. **Update this guide** with the new week's entry in the Change Log

### MCP Tools Required

| Tool | Purpose |
|------|---------|
| Fireflies MCP | `fireflies_get_transcripts`, `fireflies_get_summary` |
| Chrome MCP | Browser automation for GitHub upload |
| Slack MCP | `slack_send_message` to user DM |
| Gmail MCP | `create_draft` with HTML body |
| Scheduled Tasks MCP | `create_scheduled_task` for automation |

---

## 7. Technical Notes

### Transferring Files to GitHub

The sandbox and browser run in completely separate environments with no shared filesystem. To upload files from the sandbox to GitHub:

**Method A â Fetch + Fix from GitHub (preferred for edits):**
1. Use `javascript_tool` in Chrome to fetch current file from `raw.githubusercontent.com`
2. Apply fixes via JavaScript string operations in the browser
3. Create a File object and inject into GitHub's upload form via DataTransfer API
4. Commit directly to main

**Method B â Gzip + Base64 injection (for new files):**
1. In sandbox: `gzip -c file.html | base64 -w0 > gz_b64.txt`
2. Split into chunks and inject into browser via `javascript_tool`
3. Decompress in browser using `DecompressionStream` API. IMPORTANT: do NOT use `new Blob([bytes]).stream().pipeThrough(ds)` - it stalls indefinitely on GitHub pages. Write to the stream's own writer instead: `const ds = new DecompressionStream("gzip"); const w = ds.writable.getWriter(); w.write(bytes); w.close(); await new Response(ds.readable).arrayBuffer();`
4. Create File from result and inject into upload form

### Known Limitations

- **Scheduled task session restriction:** `create_scheduled_task` cannot be called from within a scheduled task session. The task must be created from a regular Cowork session.
- **Slack message validation:** Avoid special characters (tildes, em dashes, emoji shortcodes with colons) in Slack block messages to prevent `invalid_blocks` errors.
- **Base64 chunk alignment:** When splitting base64 for browser injection, the final concatenated string must have length divisible by 4.

---



## 10. Master Property Reference

**Source:** [Master Property List](https://docs.google.com/spreadsheets/d/1TD9UU2VTPWcYwL7AaHyHOLUeFE70MBx-DLCrRrPJpHg/edit?gid=0#gid=0) (Google Sheet, owned by Melanie Toquica)

**Total: 19 properties | 5,651 units | 5 markets**

CRITICAL: The dashboard and all reports MUST use the market (City, State) listed below. Do NOT infer markets from meeting discussion groupings.

### ARIZONA (3 properties, 672 units)

| # | Property Name | Former Name | Market | Units | Asset Manager |
|---|--------------|-------------|--------|-------|---------------|
| 1 | The Flats | District Flats | Mesa, AZ | 112 | Mike |
| 2 | Nightingale on 25th | Tides on 25th | Deer Valley, AZ | 240 | Mike |
| 3 | The Julia | — | Mesa, AZ | 320 | Mike |

### FLORIDA (8 properties, 2,741 units)

| # | Property Name | Former Name | Market | Units | Asset Manager |
|---|--------------|-------------|--------|-------|---------------|
| 4 | The Boardwalk | — | Fort Myers, FL | 338 | Nicole |
| 5 | Hanley Place | — | Tampa, FL | 400 | Nicole |
| 6 | Skye at Conway | — | Orlando, FL | 220 | Nicole |
| 7 | Skye Oaks | Palms at Palisades | Brandon, FL | 125 | Nicole |
| 8 | Skye Oaks | Brandon Oaks | Brandon, FL | 160 | Nicole |
| 9 | Skye Reserve | Reserve at Brandon | Brandon, FL | 982 | Nicole |
| 10 | Preserve at Riverwalk | — | Bradenton, FL | 300 | Nicole |
| 11 | Skye at Hunter's Creek | Sonoma Pointe | Kissimmee, FL | 216 | Nicole |

### GEORGIA (2 properties, 466 units)

| # | Property Name | Former Name | Market | Units | Asset Manager |
|---|--------------|-------------|--------|-------|---------------|
| 12 | Park at Peachtree Hills | — | Atlanta, GA | 118 | Mike |
| 13 | Upland Townhomes | — | Mableton, GA | 348 | Mike |

### TEXAS (5 properties, 1,556 units)

| # | Property Name | Former Name | Market | Units | Asset Manager |
|---|--------------|-------------|--------|-------|---------------|
| 14 | Ninety-Nine 44 | — | Dallas, TX | 260 | Mike |
| 15 | Parks At Walnut | — | Dallas, TX | 308 | Mike |
| 16 | Chimney Hill | — | Dallas, TX | 240 | Mike |
| 17 | Skye Isle | Bayou Bend | Dallas, TX | 308 | Nicole |
| 18 | Skye at Love | Pecan Square | Dallas, TX | 440 | Nicole |

### LAS VEGAS (1 property, 216 units)

| # | Property Name | Former Name | Market | Units | Asset Manager |
|---|--------------|-------------|--------|-------|---------------|
| 19 | Skye Ridge | Sunridge | Las Vegas, NV | 216 | Nicole |

### Common Market Mistakes to Avoid

- Chimney Hill is in DALLAS, TX (not Atlanta)
- Skye at Hunter's Creek is in KISSIMMEE, FL (not Atlanta)
- Skye at Conway is in ORLANDO, FL (not Dallas)
- Skye Reserve is in BRANDON, FL (not Dallas)
- Skye Ridge is in LAS VEGAS, NV (not Dallas)
- Skye Oaks (both) are in BRANDON, FL (not Tampa)
- The Boardwalk is in FORT MYERS, FL (not Tampa)

## 11. Change Log

| Date       | Change | Details |
|-----------|--------|---------|
| 2026-04-14 | Initial build | Created 96KB HTML dashboard from 4 Fireflies transcripts covering 11 of 19 properties |
| 2026-04-14 | GitHub Pages deployment | Deployed to https://zamirkazi.github.io/zmr-weekly-AM-reports/ via main branch |
| 2026-04-14 | JSON archive | Created `archive/2026-W15.json` with structured data for Week 15 |
| 2026-04-14 | Slack distribution | Sent detailed message to Zamir DM (U01N25J7789) |
| 2026-04-14 | Gmail draft | Created draft to all 7 team members with live dashboard link |
| 2026-04-14 | Bold formatting fix | Fixed 10 unclosed `<strong>` tags in Action Items sections causing body paragraphs to render bold. Commit `5230d94`. |
| 2026-04-14 | Implementation guide | Created this document for portability and knowledge preservation |
| 2026-04-14 | Property reference added | Added Section 10 with all 19 properties, correct markets from Master Property List Google Sheet. Fixed market misattributions (Chimney Hill, Hunter's Creek, Conway, Reserve, Ridge, Oaks, Boardwalk). Updated portfolio count from 18 to 19. |
| 2026-09-21 | Week 38 report generated | Built dashboard from 47 Fireflies transcripts (2026-09-14 to 2026-09-21). All 19 properties covered; 16 reported an occupancy figure. Portfolio: 88.1% unit-weighted occupancy across 4,783 reporting units, 88.0% average collections, 3 green / 12 amber / 4 red. |
| 2026-09-21 | GitHub Pages deployment | Committed `index.html` (60,733 bytes) to main. Verified live at https://zamirkazi.github.io/zmr-weekly-AM-reports/ with all 19 rows and correct markets per Section 10. |
| 2026-09-21 | JSON archive | Created `archive/2026-W38.json` (39,011 bytes) with meta, portfolioSnapshot, properties (19), risks (5), actionItems (39), wins (8), recommendations (7) and coverageGaps (6). |
| 2026-09-21 | Slack distribution | Sent Week 38 summary to Zamir DM (U01N25J7789), basic ASCII only. |
| 2026-09-21 | Gmail draft | Created draft to all 7 team members with the live dashboard link. |
| 2026-09-21 | Spec corrections | Section 3 still said 18 table rows and 11 deep-dive cards; corrected to 19 rows and all 19 properties to match Section 10. |
| 2026-09-21 | Transfer method note | `DecompressionStream` via `Blob.stream().pipeThrough()` stalls and never resolves on the GitHub upload page. Writing to the stream's own writer (`ds.writable.getWriter()`) works. See Section 7. |
| 2026-09-21 | Coverage gap follow-ups | Preserve at Riverwalk has no weekly ops call (capital/valuation coverage only) and both Skye Oaks parcels are reported as one combined campus. Recommend adding a Preserve ops call and splitting Skye Oaks by parcel. |

---

## 12. Troubleshooting

**Dashboard not updating after commit:**
GitHub Pages can take 1â5 minutes to propagate. Check deployment status at Settings â Pages in the repo.

**Slack message fails with `invalid_blocks`:**
Strip special characters from the message: no tildes (~), no em dashes (â), no emoji shortcodes (:emoji:). Keep to basic ASCII.

**Gmail draft missing recipients:**
Use `search_threads` or Slack `slack_search_users` to look up current email addresses. All addresses follow the pattern `name@zmrcapital.com`.

**Fireflies returns no transcripts:**
Check the date range filter. Transcripts are typically available within a few hours of the meeting ending. Use `fireflies_get_transcripts` with a broader date range if recent meetings aren't showing.

**File upload to GitHub fails:**
If the gzip+base64 approach corrupts data, use Method A (fetch from GitHub, modify in browser, re-upload). This avoids transferring large payloads through JavaScript string injection.
