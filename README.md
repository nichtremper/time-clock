# Nanny Hours Logger — Setup Guide

This guide walks you through getting Jayme's time clock up and running. You only need to do this once.

**What you'll need:** a Google account and a GitHub account.

---

## Step 1 — Set up the Google Apps Script

The Apps Script is the invisible middleman that writes Jayme's entries to your Google Sheet.

1. Open [script.google.com](https://script.google.com) in your browser and sign in with your Google account.
2. Click **New project** (top left).
3. You'll see a code editor with some default text. **Select all of it and delete it.**
4. Open the file `apps-script.js` from this project on your computer (in a text editor — TextEdit works fine). Select all the text and copy it.
5. Paste it into the script editor.
6. Click the floppy disk icon (or press Cmd+S) to save. Give the project any name you like — "Nanny Hours" is a good choice.
7. Click **Deploy** (top right) → **New deployment**.
8. Next to "Select type," click the gear icon and choose **Web app**.
9. Fill in the settings:
   - **Description:** anything you like (e.g. "v1")
   - **Execute as:** Me *(your Google account)*
   - **Who has access:** Anyone
10. Click **Deploy**.
11. Google will ask you to authorize the script. Click **Authorize access**, choose your Google account, and click through the permission screens. This is normal — you're authorizing the script (which runs as you) to create and edit Google Sheets on your behalf.
12. After authorizing, you'll see a **Web app URL** — a long link starting with `https://script.google.com/...`. **Copy this URL.** You'll need it in Step 2.

> **Where does the spreadsheet go?** The first time Jayme submits the form, the script automatically creates a Google Sheet called "Nanny Hours 2025" (or the current year) in your Google Drive. You don't need to create it manually.

---

## Step 2 — Connect the form to the script

1. Open `index.html` in a text editor (TextEdit, Notepad, or any code editor).
2. Near the top of the `<script>` section, find this line:
   ```
   const APPS_SCRIPT_URL = '';
   ```
3. Paste the Web App URL from Step 1 between the single quotes, so it looks like:
   ```
   const APPS_SCRIPT_URL = 'https://script.google.com/macros/s/ABC.../exec';
   ```
4. Save the file.

---

## Step 3 — Publish to GitHub Pages

1. In your terminal, from inside this project folder, run:
   ```
   git add index.html README.md .gitignore CLAUDE.md
   git commit -m "Add time clock app"
   git push
   ```
   *(Do **not** add `apps-script.js` — it's intentionally excluded.)*

2. Go to your repository on github.com and click **Settings** → **Pages** (in the left sidebar).
3. Under **Source**, choose **Deploy from a branch**, select the **main** branch and **/ (root)**, then click **Save**.
4. Wait about a minute, then refresh the page. GitHub will show you a URL like `https://your-username.github.io/time-clock/`. That's Jayme's form!
5. Send that URL to Jayme.

---

## Step 4 — Test it

1. Open the GitHub Pages URL.
2. Fill in today's date, a clock-in time, and a clock-out time.
3. Click **Log Hours**.
4. You should see a green success message.
5. Open Google Drive and look for a spreadsheet called **Nanny Hours 2025**. Inside, you should see a sheet tab for the current week with Jayme's entry.

---

## Year rollover (no action needed)

When Jayme submits her first entry for a new calendar year, the script automatically creates a new spreadsheet called **Nanny Hours 2026** (or whatever year). The old spreadsheet stays in your Drive untouched. You don't need to do anything.

The year is determined by the Monday of the work week — so if Jan 1 falls mid-week, the new workbook won't be created until the first full week of the new year begins.

---

## Troubleshooting

**The form shows a "Could not reach the server" error**
- Check that `APPS_SCRIPT_URL` in `index.html` is filled in correctly (no extra spaces, no missing quotes).
- Make sure the script is deployed as a **Web app** with **Who has access: Anyone** — not just "Anyone with the link."
- Re-deploy the script after making any code changes (click Deploy → Manage deployments → edit the existing deployment → Deploy).

**The sheet isn't updating**
- Check Google Drive for a spreadsheet named "Nanny Hours [year]". If it doesn't exist, try submitting the form again and check for an error message.
- Open the Apps Script at script.google.com, click **Executions** (left sidebar), and look for any recent failed runs — the error messages there will explain what went wrong.

**The entry went into the wrong week's tab**
- The week always runs Monday–Sunday. If an entry was submitted with a date that falls on, say, a Wednesday, it will appear in the tab for the Monday of that week. This is correct behavior.

**The form works locally but not on GitHub Pages**
- Make sure you committed and pushed `index.html` after adding the `APPS_SCRIPT_URL`. Open the page, view source, and confirm the URL is there.

**Google says the app isn't verified**
- This is a warning Google shows for scripts that access sensitive data and aren't submitted for review. Since this script is yours and only you authorized it, click **Advanced** → **Go to [project name] (unsafe)**. This is safe to do.
