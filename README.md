# Main Stage

A single-page dashboard that shows tour guide shoutouts — "Rockstar Moments" — as
they come in. It reads a Google Sheet live, auto-refreshing every 30 seconds, and
renders four views: **Just In** (the five newest moments), **The Setlist**
(every moment), **Artist Lookup** (all moments logged for one guide), and
**The Charts** (running counts per person).

Everything is in `index.html` — no build step, no frameworks, no backend. Open
the file locally or serve it from GitHub Pages and it works.

## Where the data comes from

Submissions are written by a Google Form, which feeds its linked responses sheet.
The page reads that sheet through Google's public `gviz/tq` endpoint via JSONP,
so the sheet must be shared as **Anyone with the link → Viewer**. No API key and
no third-party service is involved.

You can log a moment without leaving the page: the "Log a Rockstar Moment" button
opens an inline form that POSTs directly to the Google Form's `formResponse`
endpoint. It is the same form and the same responses tab — only the UI differs,
so entries made here and entries made in Google Forms land in exactly one place.

Expected columns, in order:

| Column | Question | Notes |
| --- | --- | --- |
| A | Timestamp | Auto-filled by the form |
| B | Who are you reporting on? | The guide being recognized |
| C | What was their Rockstar Moment? | The story |
| D | Who's Reporting? | Optional — shown as "spotted by …", hidden when blank |

Rows with no name in column B are skipped.

## The Charts

Two rankings, both listing **every** name on the sheet — these are full lists,
not top-10 cuts:

- **Shoutouts Received** — how many moments each guide has been recognized for.
- **Shoutouts Given** — how many moments each reporter has logged. Moments
  submitted without a name in column D are simply not counted here, rather than
  being lumped into an "Anonymous" row.

Names are grouped case-insensitively and ignoring extra spaces, so `jordan`,
`Jordan` and `  JORDAN  ` are one person. The displayed spelling prefers a
name-cased variant someone actually typed; a name only ever entered in lowercase
stays lowercase, since inventing capitalization would mangle names like
`McDonald` or `de Vries`.

## Duplicate protection

Before posting, an entry is checked against the sheet and against everything
already submitted in that browser session. A match blocks the post instead of
creating a second row. Comparison ignores case and extra spaces, and covers
guide + moment + reporter together — so one person cannot post the same moment
twice, but two different people can independently say the same thing about the
same guide.

This guards against the common accidents: a double-tapped submit button, or
re-submitting after the sheet was slow to show the first attempt. It is not
account-based, so it does not stop someone deliberately re-posting the same
praise with different wording, or the same text from a different browser.

## Pointing it at a different sheet or tab

Both live at the top of the `<script>` block in `index.html`:

```js
const SHEET_ID = "1IWwy3ygxsq81p9VRUW8MTv-tixjRWeb4BfcA-sT8YI8";
const SHEET_GID = "944135761";
const FORM_URL  = "https://docs.google.com/forms/d/e/1FAIpQLS…/viewform";
```

- **`SHEET_ID`** — the long id in the sheet URL, between `/d/` and `/edit`.
- **`SHEET_GID`** — the numeric tab id, the `gid=` value in the URL when that tab
  is open. Use the gid rather than the tab name: renaming a tab breaks a
  name-based lookup, but the gid stays the same for the life of the tab.
- **`FORM_URL`** — the form's `viewform` link. `FORM_ACTION` is derived from it
  automatically, and it doubles as the fallback link inside the entry panel.

## If you change the form's questions

The inline form posts to per-question field ids, which sit just below `FORM_URL`:

```js
const FIELD_GUIDE     = "entry.1211982758";  // Who are you reporting on?
const FIELD_MOMENT    = "entry.113659133";   // What was their "Rockstar Moment?"
const FIELD_NOMINATOR = "entry.733750023";   // Who's Reporting? (optional)
```

Editing a question's wording is safe. **Deleting a question and re-adding it
gives it a new id**, which silently breaks posting for that field. To re-read the
ids: open the live form, View Source, and search for `entry.` — each question's
id appears in the `FB_PUBLIC_LOAD_DATA_` blob near its question text.

Adding a question also appends a new column to the sheet at the far right, in
creation order — not in the order the questions appear on the form. If you add a
fifth field, it becomes column E, and `parseRows()` needs `cells[4]` to read it.

## Troubleshooting

- **"Couldn't reach the responses sheet"** — check the sharing setting first,
  then confirm the gid still matches the responses tab.
- **"Sent, but it hasn't appeared yet"** — the POST left the browser but the row
  didn't show up within about 14 seconds. Usually Sheets lagging; if it happens
  every time, the field ids above are likely stale.
