# Session 001 Summary

## What We Built

A nanny hour-logging web app for Jayme (nanny) to log her hours for the Tremper family. Single `index.html` hosted on GitHub Pages. Submissions POST to a Google Apps Script Web App, which writes rows to a Google Sheet in Nich's Drive.

---

## Files Created

| File | Status | Notes |
|------|--------|-------|
| `index.html` | Pushed, live on GitHub Pages | The entire front end |
| `apps-script.js` | Gitignored — local only | Pasted into script.google.com manually |
| `README.md` | Pushed | Non-developer setup guide for Nich |
| `CLAUDE.md` | Pushed | Codebase guidance for future Claude sessions |
| `.gitignore` | Pushed | Added `apps-script.js` to existing Python template |

---

## Architecture

- Front end is a single static HTML file — no build step, no dependencies except Google Fonts CDN
- `const APPS_SCRIPT_URL = '...'` at the top of the `<script>` block is the only connection to the back end
- Apps Script runs as Nich's Google account on Google's servers — no credentials in the repo
- CORS workaround: fetch uses `Content-Type: text/plain` (not `application/json`) to avoid a preflight OPTIONS request that Apps Script doesn't support. The body is still JSON.

---

## Current Deployed State

- **GitHub Pages URL:** `https://nichtremper.github.io/time-clock/`
- **Apps Script:** deployed as Web App, Execute as: Me, Access: Anyone
- **Apps Script URL:** already wired into `index.html` (the long `script.google.com/macros/s/.../exec` URL)
- **Google Drive:** spreadsheets go into a folder named `nanny` (all lowercase, exact match)
- **Spreadsheet naming:** `Nanny Hours YYYY` — one per calendar year
- **Sheet tab naming:** `Week of Mon D, YYYY` (e.g. `Week of Apr 28, 2025`)
- **Sheet ordering:** newest tab is at position 0 (leftmost), reverse-chronological

---

## Design Decisions

- **Fonts:** Playfair Display (headings) + Nunito (body) — warm, domestic feel, not corporate
- **Colors:** terracotta (`#c97c5d`) primary, cream (`#fdf6ef`) background, sage green (`#7a9e87`) for sheet header rows
- **Header:** "Hi, Jayme!" + subtitle "Log your hours for the Tremper family"
- **Hours format:** decimal, 2 decimal places (e.g. `7.50`) — chosen for easy summing in Sheets
- **Pay period:** Monday–Sunday of the submitted week
- **Payday:** Monday of that week + 11 days (the Friday after the Sunday)
- **Year boundary:** the year is determined by the Monday of the work week, not the submitted date (e.g. Jan 2 that falls in the week of Dec 29 → prior year's workbook)

---

## Google Apps Script — Key Implementation Details

### PropertiesService keys used
- `CURRENT_YEAR` — string year of the active spreadsheet (e.g. `"2026"`)
- `CURRENT_SPREADSHEET_ID` — Drive file ID of the active spreadsheet
- `FOLDER_MOVE_DONE_YYYY` — set to `"true"` once a new year's spreadsheet has been successfully moved to the `nanny` folder

### New spreadsheet creation flow
1. `SpreadsheetApp.create('Nanny Hours YYYY')`
2. Immediately store ID + year in PropertiesService (so a subsequent Drive error doesn't orphan the file)
3. `DriveApp.getFoldersByName('nanny')` → `moveTo()`
4. Set `FOLDER_MOVE_DONE_YYYY = true`
5. Rename default "Sheet1" to `_delete_me`

### Folder move retry logic
When opening an existing spreadsheet (found via stored ID), if `FOLDER_MOVE_DONE_YYYY` is not set, the script retries the `moveTo` call. This self-heals the case where a Drive error interrupted the move during initial creation.

### Sheet1 cleanup
`getOrCreateSheet` deletes whichever placeholder exists — `_delete_me` (normal path) or `Sheet1` (interrupted creation path). Guards with `ss.getSheets().length > 1` to avoid deleting the only sheet.

---

## Issues Encountered and Resolved

### 1. DriveApp permission error
**Error:** `You do not have permission to call DriveApp.getFoldersByName`  
**Cause:** Adding `DriveApp` introduced a new OAuth scope not in the original authorization.  
**Fix:** Added explicit `oauthScopes` to `appsscript.json` in the script editor (timezone: `America/New_York`). Then re-deployed as a new version, which triggered re-authorization.

```json
{
  "timeZone": "America/New_York",
  "dependencies": {},
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8",
  "oauthScopes": [
    "https://www.googleapis.com/auth/spreadsheets",
    "https://www.googleapis.com/auth/drive",
    "https://www.googleapis.com/auth/script.external_request"
  ]
}
```

### 2. Sheet tabs in wrong order
**Issue:** New week tabs were appended at the end, placing a December entry between April entries.  
**Fix:** `ss.insertSheet(sheetName, 0)` — inserts at position 0 (leftmost) so newest is always first.

### 3. New-year spreadsheet Drive error
**Error:** `Service error: Drive` on the first submission to a new year (tested with 2027).  
**Cause:** Intermittent Google Drive service error during `moveTo`. The spreadsheet was created and registered, but not moved to the `nanny` folder. The `_delete_me` rename never ran either.  
**Fix:** 
- Store PropertiesService ID before `moveTo` (prevents orphaned spreadsheets on retry)
- `FOLDER_MOVE_DONE_YYYY` flag + retry logic on open (self-heals folder placement)
- Fallback cleanup of plain `Sheet1` in `getOrCreateSheet`

**Known residual state:** The `Nanny Hours 2027` test spreadsheet may still be in Drive root rather than the `nanny` folder. Nich should check and manually drag it in if so. Once re-deployed, the next 2027 submission will also attempt to move it automatically.

---

## Manual Steps Nich Has Completed

- [x] Created Apps Script project at script.google.com ("Nanny Hours")
- [x] Deployed as Web App (Execute as: Me, Access: Anyone)
- [x] Authorized Drive + Sheets permissions (including editing `appsscript.json`)
- [x] Pasted Web App URL into `index.html`
- [x] Pushed repo to GitHub
- [x] Enabled GitHub Pages (source: main branch, root)
- [x] Tested successfully — entries appear in `nanny/Nanny Hours 2026` in Drive

## Pending / To Verify

- [ ] Check if `Nanny Hours 2027` is in Drive root or `nanny` folder — move manually if needed
- [ ] Re-paste and re-deploy `apps-script.js` with the latest fixes (sheet ordering + folder retry)
- [ ] Send GitHub Pages URL to Jayme
