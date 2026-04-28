# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

A nanny hour-logging web app. A single `index.html` is hosted on GitHub Pages. On submit, it POSTs JSON to a Google Apps Script Web App, which writes a row to a Google Sheet. No credentials exist in this repo.

## Files

- `index.html` — the entire front end; no build step, no dependencies
- `apps-script.js` — Google Apps Script code; **gitignored, never pushed**; user pastes it into script.google.com manually
- `.gitignore` — must include `apps-script.js`
- `README.md` — non-developer setup guide

## Architecture

The front end connects to the back end via a single constant at the top of `index.html`:

```js
const APPS_SCRIPT_URL = '';  // user fills this in after deploying the Apps Script
```

The Apps Script runs as the repo owner's Google account on Google's servers. The Web App URL it generates is the only connection; it is not a secret.

## Business Logic

**Pay period:** Monday–Sunday of the week containing the submitted date.  
**Payday:** Friday after that Sunday (Monday of the week + 11 days).  
**Hours format:** Decimal, two decimal places (e.g., `7.50`).

**Workbook naming:** `Nanny Hours YYYY` — one Google Spreadsheet per calendar year. The year is determined by the Monday of the submitted week (e.g., a date of Jan 2 that falls in the week of Dec 29 belongs to the prior year's workbook).

**Sheet tab naming:** `Week of Mon DD, YYYY` (e.g., `Week of May 12, 2025`).

The Apps Script uses `PropertiesService.getScriptProperties()` to persist the current spreadsheet ID and year across invocations.

## Key Constraints

- No Google credentials or API keys anywhere in the repo or in `index.html`
- `index.html` must work entirely client-side — no Node, no build step
- GitHub Pages serves static files only; no server-side code
- `apps-script.js` must never be committed (gitignored)
