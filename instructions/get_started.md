# Nanny Hours Logger — Instructions for Claude Code

Read this entire file before doing anything. These are the requirements for a nanny hour-logging web app. Build everything described here.

---

## What This Is

A simple web form hosted on GitHub Pages where a nanny can log her daily hours. When she submits the form, her entry is written to a Google Sheet automatically. No Google credentials should exist anywhere in the GitHub repository.

---

## Architecture

Use Google Apps Script as a middleman. The form POSTs data to a Google Apps Script Web App URL. The script runs as the repo owner's Google account (on Google's servers) and writes to Google Sheets. The Apps Script URL is not a secret — it just accepts data. Nothing sensitive lives in the repo.

The flow is:
1. Nanny fills out the form on GitHub Pages
2. Form POSTs JSON to the Google Apps Script Web App URL
3. Apps Script appends a row to the correct Google Sheet

---

## The Web Form (index.html)

Create a single `index.html` file. This is the only file that will be in the repo.

### Fields

- **Date** — date picker, defaults to today
- **Clock In** — time picker
- **Clock Out** — time picker
- **Notes** — optional text area (example placeholder: "Took Myles to the park")

### Behavior

- As soon as both clock-in and clock-out are filled in, calculate and display the total hours live on the page (no submit needed to see the hours)
- Also display the pay period (Monday–Sunday) and payday (the Friday after that Sunday) based on the selected date, updating live as the date changes
- On submit, POST the data as JSON to a Apps Script URL that will be stored in a JavaScript constant at the top of the script called `APPS_SCRIPT_URL`. Leave it as an empty string for now — the user will fill it in after deploying the Apps Script
- Show a clear success message on submit (including how many hours were logged), or a clear error message if something goes wrong
- Validate that clock-out is after clock-in before submitting; show an inline error if not
- Disable the submit button while the request is in flight

### Design

- Mobile-first. The nanny will likely use her phone.
- Warm, friendly, and clean. This is a home/domestic context, not a corporate app.
- Use a distinctive font pairing from Google Fonts — avoid Inter, Roboto, or Arial.
- Show a subtitle under the page title that says "Log your hours for the Tremper family"
- The overall feel should be welcoming and easy to use, not intimidating.

---

## The Google Apps Script

Create a second file called `apps-script.js`. This file will NOT be pushed to GitHub — it is for the user to copy and paste into script.google.com. Add a comment at the top of the file making this clear.

### What the script must do

**On receiving a POST request:**

1. Parse the JSON body: `{ date, clockIn, clockOut, notes }`
2. Validate that all required fields are present and that clock-out is after clock-in
3. Determine which Monday the submitted date falls in (Monday = start of pay period)
4. Determine the year of that Monday (not necessarily the calendar year of the date itself — a date of January 2 might fall in the week of December 29, which belongs to the prior year's workbook)
5. Get or create the correct Google Spreadsheet for that year (see workbook logic below)
6. Get or create the correct sheet tab for that week (see sheet logic below)
7. Append a row with: Date, Clock In, Clock Out, Hours (calculated), Notes, Pay Period (formatted date range), Payday (formatted date)
8. Return JSON: `{ success: true, hours: "X.XX" }` or `{ success: false, error: "..." }`

### Workbook logic (Google Spreadsheet files)

- There is one Google Spreadsheet per calendar year, named `Nanny Hours YYYY`
- A new workbook is created when the Monday of the submitted week falls in a new year
  - Example: if January 1, 2027 is a Friday, the first Monday of 2027 is January 4. The new 2027 workbook is created when the first entry for the week of January 4 comes in — not on January 1.
- Store the current spreadsheet ID and current year in Apps Script's `PropertiesService.getScriptProperties()` so it persists across invocations
- When creating a new workbook, clean up the blank default sheet that Google creates automatically

### Sheet tab logic

- One sheet tab per week, named `Week of Mon DD, YYYY` (e.g. "Week of May 12, 2025")
- The week is always Monday–Sunday
- If the sheet for that week doesn't exist yet, create it with a styled header row and frozen first row
- Header columns: Date, Clock In, Clock Out, Hours, Notes, Pay Period, Payday

### Pay period and payday calculation

- Pay period: the Monday of the submitted week through the following Sunday
- Payday: the Friday after that Sunday (Monday + 11 days)

---

## What to Create

1. `index.html` — the web form, ready to deploy to GitHub Pages. Leave `APPS_SCRIPT_URL` as an empty string.
2. `apps-script.js` — the Apps Script code with a comment at the top saying this file should NOT be pushed to GitHub and should be pasted into script.google.com instead.
3. `README.md` — step-by-step setup instructions written for a non-developer. Cover:
   - How to paste the Apps Script into script.google.com and deploy it as a Web App (execute as: Me, access: Anyone)
   - How to copy the Web App URL into `index.html`
   - How to create a GitHub repo, push the files (excluding `apps-script.js`), and enable GitHub Pages
   - How to test that everything is working
   - What happens automatically at year rollover (no action needed)
   - A troubleshooting section for the most common issues (CORS errors, sheet not updating, wrong week's sheet)
4. `.gitignore` — containing `apps-script.js` so it is never accidentally pushed

---

## Important Constraints

- No Google credentials or API keys anywhere in the repo or in `index.html`
- The form must work entirely client-side (no Node, no build step, no dependencies to install)
- GitHub Pages serves static files only — no server-side code
- The `APPS_SCRIPT_URL` constant in `index.html` is the only connection between the front end and the back end