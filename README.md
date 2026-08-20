# Nursing Open Lab Sign-Up

A free, no-login sign-up page for nursing open lab hours. Students pick a 30-minute
slot; each slot closes automatically once 15 people have signed up. Instructors manage
the schedule (days, times, capacity) directly in a Google Sheet — no code required to
add a new day or change hours.

Hosted for free on GitHub Pages, embedded in Canvas via an iframe, backed by a free
Google Apps Script "web app" attached to a Google Sheet — same pattern as the Writing
Marathon Leaderboard.

---

## Step 1: Set up the Google Sheet

1. Create a new Google Sheet. Name it something like **"Open Lab Sign-Up"**.
2. Rename the first tab to exactly: **`Schedule`**
   In row 1, add these headers exactly:

   | Day | Start Time | End Time | Capacity |
   |-----|-----------|----------|----------|

   Then add one row per recurring lab block. For the hours you described:

   | Day | Start Time | End Time | Capacity |
   |-----|-----------|----------|----------|
   | Wednesday | 12:00 PM | 3:00 PM | 15 |
   | Thursday | 8:00 AM | 3:00 PM | 15 |

   **This is the only tab instructors need to touch.** To add a day, add a row. To
   change hours or capacity for a day, edit that row. To remove a day, delete the row.
   Times must be typed like `12:00 PM` or `8:00 AM` (hour, colon, two-digit minutes,
   space, AM/PM).

3. Add a second tab and rename it to exactly: **`Signups`**
   In row 1, add these headers exactly:

   | Timestamp | Name | Email | Date | Time Slot |
   |-----------|------|-------|------|-----------|

   Leave this tab otherwise empty — the script will fill it in automatically as
   students sign up. This becomes your permanent record of who signed up, for what
   slot, and when they submitted it. Don't hand-edit it while the page is live.

4. **Optional:** if you expect to need one-off exceptions (a cancelled day, or one
   date with different hours), add a third tab named exactly: **`Date Overrides`**
   In row 1, add these headers exactly:

   | Date | Start Time | End Time | Capacity | Note |
   |------|-----------|----------|----------|------|

   See **"Handling one-off exceptions"** below for how to use it. You can skip this
   tab entirely if you don't need it yet — the page works fine without it and you
   can add it later.

## Step 2: Install the Apps Script

1. In your Sheet, go to **Extensions > Apps Script**.
2. Delete any placeholder code in the editor.
3. Copy everything from `Code.gs` (in this repo) and paste it in.
4. Click the save icon and name the project something like **"Open Lab Backend"**.

## Step 3: Deploy as a Web App

1. Click **Deploy > New deployment**.
2. Click the gear icon next to "Select type" and choose **Web app**.
3. Description: "Open Lab Sign-Up API".
4. **Execute as:** Me (your account).
5. **Who has access:** Anyone.
   (This does *not* give anyone access to your Drive — it only lets the page read
   availability and add rows to the Signups tab through the script's own logic.)
6. Click **Deploy**. Google will ask you to authorize access — click through
   ("Advanced" > "Go to Open Lab Backend (unsafe)" is expected for a script you wrote
   yourself).
7. Copy the **Web app URL**. It ends in `/exec`.

> Any time you edit `Code.gs` later, you need to **Deploy > Manage deployments >
> edit (pencil) > New version > Deploy** for the changes to go live. Editing the
> script alone doesn't update the live version.

## Step 4: Connect the HTML to the script

1. Open `index.html` from this repo.
2. Find this line near the top of the `<script>` section:
   ```js
   const SCRIPT_URL = 'YOUR_GOOGLE_APPS_SCRIPT_WEB_APP_URL_HERE';
   ```
3. Replace the placeholder with the URL you copied in Step 3, keeping the quotes.

## Step 5: Host it on GitHub Pages

1. Create a new **public** GitHub repository (e.g. `Nursing-Open-Lab-Signup`).
2. Upload the edited `index.html` to the repo (just this one file needs to go live —
   `Code.gs` and this README stay in the repo for reference/future edits but aren't
   needed by the page itself).
3. Go to the repo's **Settings > Pages**.
4. Under "Build and deployment," set **Source: Deploy from a branch**, branch
   **main**, folder **/(root)**. Save.
5. GitHub gives you a URL like:
   `https://yourusername.github.io/Nursing-Open-Lab-Signup/`
   It can take a minute or two to go live the first time.

## Step 6: Embed it in Canvas

In the Canvas Rich Content Editor, switch to the **HTML editor** view and paste:

```html
<iframe src="https://yourusername.github.io/Nursing-Open-Lab-Signup/"
        style="width:100%; height:900px; border:none;"
        title="Nursing Open Lab Sign-Up"></iframe>
```

Adjust `height` if the page feels cramped or oversized once you see it live —
900px is a reasonable starting point.

---

## How it works day to day

- **Students** open the Canvas page, expand a date, pick an open time, and enter
  their name and email. The slot count updates immediately for everyone.
- **A slot closes automatically** once 15 sign-ups exist for it — the button grays
  out and shows "Full." This is enforced on the server side (in the script), so two
  students can't both grab the last seat at the same moment.
- **Instructors** edit the `Schedule` tab any time to add days, change hours, or
  change capacity — no redeploying, no code. Changes show up the next time a student
  loads or refreshes the page.
- **The `Signups` tab is your attendance record** — timestamp, name, email, date, and
  slot for every sign-up, in the order they came in.
- The page shows the **next 21 days** of availability by default. Change
  `DAYS_AHEAD` near the top of `index.html`'s script if you want more or less lead
  time.

## Handling one-off exceptions

For an exception to just one specific date — canceling a single lab day, or giving
one date different hours or capacity — without touching the regular weekly pattern,
use the **`Date Overrides`** tab (see Step 1.4 above if you haven't created it yet).

**To cancel one date entirely:**
Add a row with just the date filled in — leave Start Time and End Time blank.
That date won't show any slots at all, even though it's normally a scheduled day.

| Date | Start Time | End Time | Capacity | Note |
|------|-----------|----------|----------|------|
| 11/26/2026 | | | | Thanksgiving break |

**To change hours or capacity for one date only:**
Add a row with the date plus new Start Time, End Time, and Capacity. That date uses
these hours instead of the Schedule tab's normal rows — every other date on the
recurring schedule is unaffected.

| Date | Start Time | End Time | Capacity | Note |
|------|-----------|----------|----------|------|
| 12/10/2026 | 9:00 AM | 12:00 PM | 10 | Shortened for finals week |

A few notes:
- The `Note` column is just for your own reference — the page ignores it.
- An override only affects the one date in that row. Nothing else on the recurring
  Schedule tab changes.
- Like the Schedule tab, changes here show up the next time a student loads or
  refreshes the page — no redeploying needed.
- Students who already signed up for a date before you cancel or change it won't
  be automatically notified — you'd still want to let them know directly (their
  emails are in the `Signups` tab).

**Format matters here — an unrecognized format fails silently.** If the Date
column isn't typed in a way Google Sheets recognizes as an actual date, the
override just won't match anything, and that date will quietly fall back to the
normal recurring schedule with no error shown. To avoid that:
- **Date:** type as `M/D/YYYY`, e.g. `11/26/2026`. Avoid formats like "Nov 26" —
  those risk being stored as plain text instead of a real date.
- **Start Time / End Time:** same as the Schedule tab — `12:00 PM`, `9:00 AM`, etc.
- **Capacity:** a plain number, e.g. `10`.

After adding an override row, it's worth loading the live page once to confirm the
date shows the expected (or no) slots before telling students about the change.

## Things worth knowing

- There's no login — anyone with the Canvas page link can sign up. That matches how
  the Writing Marathon Leaderboard tool works and keeps this free with no accounts
  to manage, but it also means there's no verification that the "Name" typed in
  matches the student submitting it.
- If you want to cap how far in advance students can sign up, or want the page to
  only show one week at a time, that's the `DAYS_AHEAD` constant.
- If you ever want a slot's capacity to differ from the rest of that day's block
  (say, one Wednesday slot needs to hold fewer than 15), the current design applies
  one capacity per Schedule row, not per individual slot. Flagging that as a
  known limitation in case it comes up.
