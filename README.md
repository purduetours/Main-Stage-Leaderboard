# Main Stage

A single-page dashboard that shows tour guide shoutouts — "Rockstar Moments" — as
they come in. It reads a Google Sheet live, auto-refreshing every 30 seconds, and
renders three views: **Just In** (the five newest moments), **The Setlist**
(every moment, searchable by guide name or keyword), and **Artist Lookup** (all
moments logged for one guide).

Everything is in `index.html` — no build step, no frameworks, no backend. Open
the file locally or serve it from GitHub Pages and it works.

## Where the data comes from

Submissions arrive through a Google Form, which writes to its linked responses
sheet. The page reads that sheet through Google's public `gviz/tq` endpoint via
JSONP, so the sheet must be shared as **Anyone with the link → Viewer**. No API
key and no third-party service is involved.

Expected columns, in order:

| Column | Question | Notes |
| --- | --- | --- |
| A | Timestamp | Auto-filled by the form |
| B | Who are you reporting on? | The guide being recognized |
| C | What was their Rockstar Moment? | The story |
| D | Who's Reporting? | Optional — shown as "spotted by …", hidden when blank |

Rows with no name in column B are skipped.

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
- **`FORM_URL`** — where the "Log a Rockstar Moment" button sends people.

If the page shows the "Couldn't reach the responses sheet" error, check the
sharing setting first, then confirm the gid still matches the responses tab.
