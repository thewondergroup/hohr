# Hell or High Rollers — Dashboard Setup

A three-section performance dashboard for HOHR. Hosted on GitHub Pages, fed by a Google Sheet with two tabs.

The dashboard is structured around three questions:

1. **Is the maths working?** — Patreon MRR, new subs, cancellations, cost per new sub
2. **Is the audience growing?** — Listeners, IG, TikTok, FB
3. **What's working?** — Active and recent campaigns

Plus a "monthly read" at the top — a plain-English paragraph or three written by us each month explaining what the numbers actually mean.

The Money section deliberately shows acquisition and cancellations side by side, so net growth is always visible. The "+£X net after cancellations" pill under the cancellations tile shows whether we're actually growing or just churning in place.

---

## Setup

### 1. Create the Google Sheet

You need **two tabs**, each with a specific column structure.

#### Tab 1: `Weekly` (case-sensitive)

| Column | What goes here | Example |
|---|---|---|
| `week_starting` | Monday of the week, `YYYY-MM-DD` | `2026-05-11` |
| `meta_spend` | Total Meta ad spend that week (£) | `245` |
| `meta_clicks` | Total Meta link clicks | `5180` |
| `new_patreon_subs` | New paid Patreon subs that week | `13` |
| `paid_cancellations` | Paid Patreon subs that cancelled this week | `7` |
| `spotify_listeners` | Total Spotify followers/listeners | `11240` |
| `patreon_total_subs` | Total active paid Patreon subs | `803` |
| `patreon_mrr` | Patreon monthly recurring revenue (£, gross) | `5319` |
| `ig_followers` | Total Instagram followers | `4742` |
| `fb_followers` | Total Facebook page followers | `3068` |
| `tiktok_followers` | Total TikTok followers | `1935` |
| `notes` | Optional context for the week | `Sam Riegel ep dropped` |

#### Tab 2: `Campaigns` (case-sensitive)

| Column | What goes here | Example |
|---|---|---|
| `name` | Campaign / creative name | `Henley Reel S2` |
| `platform` | Where it ran | `Spotify`, `Patreon`, `Instagram` |
| `status` | One of: `live`, `paused`, `ended` | `live` |
| `start_date` | When it started, `YYYY-MM-DD` | `2026-04-06` |
| `end_date` | When it ended (blank if still running) | `2026-04-13` |
| `spend` | Total £ spent on this campaign | `280.76` |
| `primary_result` | The main number (clicks / visits / follows) | `12236` |
| `result_label` | What that number is | `clicks`, `profile visits`, `subs` |
| `secondary_metric` | Efficiency string | `2p / click`, `£7.50 / sub` |

### 2. Publish each tab as CSV

For **each** tab separately:
- File → Share → Publish to web
- "Link" tab, select the specific tab from the dropdown (Weekly or Campaigns)
- Format: Comma-separated values (.csv)
- Click Publish, copy the URL

You'll end up with **two CSV URLs**.

### 3. Wire it up

Open `index.html`, find the `CONFIG` block near the top of the `<script>`, paste both URLs:

```js
const CONFIG = {
  WEEKLY_CSV_URL: 'https://docs.google.com/.../pub?gid=0&single=true&output=csv',
  CAMPAIGNS_CSV_URL: 'https://docs.google.com/.../pub?gid=12345&single=true&output=csv',
  SHEET_EDIT_URL: 'https://docs.google.com/spreadsheets/d/.../edit',
  ...
};
```

### 4. Update the monthly read

Each month, edit the `MONTHLY_READ` block in the same `CONFIG`:

```js
MONTHLY_READ: {
  period: 'May 2026',
  title: "Acquisition is strong, retention needs a look",
  paragraphs: [
    'First paragraph — the headline finding for the month.',
    'Second paragraph — context on what happened.',
    'Third paragraph — what to do about it.'
  ]
}
```

You can use `<span class="hl">highlighted term</span>` inside paragraphs to draw the eye to specific campaign names or important numbers.

### 5. Deploy to GitHub Pages

Push to a repo, enable Pages, share the URL with HOHR. The dashboard has `noindex,nofollow` set so it won't show up in search results.

---

## How metrics behave in the monthly view

When you toggle to the Monthly view, the dashboard aggregates weekly data:

- **Additive metrics** (summed across the weeks of the month): `meta_spend`, `meta_clicks`, `new_patreon_subs`, `paid_cancellations`
- **Stock metrics** (last value of the month): `spotify_listeners`, `patreon_total_subs`, `patreon_mrr`, all follower counts

This is the correct treatment — you don't "add up" followers across weeks; you take the latest snapshot. But you do add up spend.

---

## Cost per new sub — how it's calculated

The dashboard calculates this client-side:

```
cost_per_new_sub = meta_spend / new_patreon_subs
```

The tile shows the cost alongside the average lifetime value of a paying sub (£98 from the Patreon analysis), so you can see at a glance whether each acquisition is profitable. A £20 cost-per-sub against a £98 LTV gives a ~5× return on each acquisition.

Two caveats worth knowing:

1. **Attribution is not perfect.** Meta drives traffic to Patreon, but not every new sub came from an ad. Some are organic (word of mouth, listening to the podcast and converting on their own). The number is a *blended* cost, not a strict Meta-attributed one.
2. **The number is gross, not net.** "New subs" counts joins; some of those will cancel. The Cancellations tile alongside shows the net picture — that's where to look for the honest acquisition rate.

The dashboard's delta colour is **inverted** for cost per sub — going down (cheaper) shows as green, going up (more expensive) shows as red. Same for cancellations (more is worse, so increases show red). Other metrics use the conventional direction.

---

## Adding data each week

1. Add one row to the `Weekly` tab with the week's numbers
2. If we launched, paused, or ended campaigns, update the `Campaigns` tab
3. Refresh the dashboard

The dashboard re-fetches the CSVs on every load with `cache: 'no-store'`, so updates appear immediately. If they don't, hard-refresh (Ctrl+Shift+R / Cmd+Shift+R).

---

## Refreshing the historical growth chart

`CONFIG.MONTHLY_JOINERS` powers the long-arc growth chart. Refresh it quarterly:

1. Export the All-patrons CSV from Patreon
2. Group rows by month, counting paid (`Status == "Paid"`) vs total
3. Paste the updated array into the HTML

**Gotcha worth knowing about:** Patreon writes September as **"Sept"** (4 letters), not "Sep" like every other 3-letter month abbreviation. If your script uses a standard `%b` date parser, 100% of your September rows will silently fail and disappear from the analysis. Always do a quick sanity check: do you have a bar for every month, or are there suspicious gaps? The fix is one line — normalise `"Sept"` to `"Sep"` before parsing.

---

## What's not yet measured

The dashboard is built around the data we can reliably pull each week. A few things we're not tracking yet but could add:

- **Per-episode performance**. Most podcast hosts (and Spotify for Podcasters) show downloads per episode. Useful when a specific guest or content drop performs unusually well.
- **Direct ad-to-sub attribution**. The dashboard shows a blended cost-per-sub. For exact attribution we'd need UTM tracking on Patreon links or use of Patreon's API.
- **Cohort retention**. The monthly cancellation count is shown; cohort survival (do 2025 joiners stay longer than 2026 joiners?) would need more granular Patreon data.

These are all addable when the relevant data becomes available.

---

## Troubleshooting

**"Some data couldn't load"** banner: one of the CSV URLs is wrong or not published.

**Dashboard shows demo data**: both `WEEKLY_CSV_URL` and `CAMPAIGNS_CSV_URL` are empty in CONFIG.

**New rows don't appear**: browser cache. Hard refresh.

**Cost per sub shows `—`**: either `meta_spend` or `new_patreon_subs` is empty for that week, or `new_patreon_subs` is zero (can't divide by zero).

**Monthly view shows weird MRR jumps**: MRR is a stock metric so the dashboard takes the last week's value of the month. If MRR is empty in the most recent week of a month, it'll fall back to whatever the last non-empty value was — keep MRR filled in every week to avoid surprises.
