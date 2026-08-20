# KPJU-SOP CPD Tracker

**A centralised dashboard for KPJU School of Pharmacy's CPD Committee — plan, track, and follow every Continuing Professional Development programme from first planning through to close-out.**

🔗 **Live:** [kpjucpdtracker.vercel.app](https://kpjucpdtracker.vercel.app)

---

## What it does

CPD (Continuing Professional Development) programmes at KPJU School of Pharmacy move through a long, compliance-heavy pipeline — HRD Corp levy claims, CCPD approval, CPD points applications, participant caps, trainer documentation — spread across multiple people and deadlines. This tracker pulls that process into one dashboard with three views:

**1. Programme picker**
Select a CPD year and programme to jump straight to that programme's own tracker (a linked, programme-specific Google Sheet tab), plus a direct link to upload event photos to the right SharePoint/OneDrive folder — no hunting through shared drives for the correct link.

**2. Revenue snapshot**
Live figures pulled from a published Revenue Summary sheet: total validated revenue, revenue by programme, revenue per participant by programme, progress against a RM 1,200,000 annual target (radial gauge), and a self-sponsored vs. company-sponsored participant split. Programmes with no revenue recorded yet, or figures that need checking, are surfaced separately rather than silently folded into the total — an intentional accuracy choice, not an oversight.

**3. CPD calendar**
A year-at-a-glance heat grid — every 2026 programme plotted across the calendar, filled squares marking the days a programme runs.

**4. Programme lifecycle reference**
A step-by-step guide to how a CPD programme actually moves through CCPD, independent of which specific programme is selected above. Each stage — Setup, CCPD Approval (T-60 days), Registration, Final Prep (T-14 days), Event Day, Close-out — expands into its overview, required tasks, and linked documents. This is where the institutional knowledge lives: HRD Corp levy forfeiture rules, participant claim caps by training type, the exact claim documents required (JD/14 form, T3 attendance form, PSMB-addressed invoice with SST number), and the 6-month claim submission deadline that doesn't pause even if HRD Corp raises a query.

## Why this matters

Most of what's encoded in the lifecycle reference isn't obvious and isn't written down anywhere else in one place — it's the kind of institutional knowledge that normally lives in one committee member's head and gets re-explained every cycle. Putting it in the tracker means a new committee member (or the same one, six months later) doesn't have to re-derive it.

## Architecture

Zero-backend, same pattern as Aufthority's other tools: a single static `index.html`, no server, no database. All live data — programme lists, revenue figures — comes from Google Sheets tabs published as CSV and fetched client-side, so updating a number is editing a spreadsheet cell, not touching code or redeploying.

```js
const PROGRAMME_SOURCES = {
  '2026': { type: 'static',  programmes: [ /* hardcoded array */ ] },
  '2027': { type: 'csv',     url: '<published CSV link>' },
};
```

This dual-mode design is deliberate: a hardcoded list works fine for a small, stable set of programmes (2026), while a CSV source lets a future year scale to many programmes as a spreadsheet edit — add a row, not a deploy. Each programme entry can optionally carry a `photoFolderUrl`; when present, the "Upload Event Photos" button appears automatically for that programme with no other code changes.

- **Programme data / revenue data**: published-CSV fetch with an RFC 4180–compliant parser (handles quoted fields, embedded commas/newlines)
- **PWA**: installable, manifest + icons, standalone display
- No frameworks — vanilla HTML/CSS/JS

## Data setup

Each data source is a Google Sheet tab published via **File → Share → Publish to web → [tab] → CSV → Publish**:

| Source | Feeds |
|---|---|
| Programme index (per year) | Programme picker dropdown |
| Revenue Summary tab | Revenue snapshot section |

Column names are matched flexibly (`findCol` checks several likely header variants per field), so the underlying sheet can be restructured — columns reordered or renamed — without breaking the fetch, as long as a recognisable header is still present.

## Known constraints

- Dependent on Google Sheets "Publish to web" links staying live and correctly scoped — if a tab is unpublished or its sharing settings change, that section shows a status message rather than failing silently.
- Revenue figures reflect whatever the underlying sheet reports; the "needs review" flag is only as good as whoever is flagging rows in the source sheet.
- Static architecture means this is a read/reporting layer on top of the Sheets, not a system of record — the sheets remain the source of truth.
