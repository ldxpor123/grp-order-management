# Group Order Tracker

A single page (`index.html`) where a friend types their name or Telegram @ and sees **only their own** orders. It reads two tabs of a Google Sheet that are published as CSV. You edit the sheet and the page follows.

> The order data (xlsx/csv) is deliberately **not** in this repo (see `.gitignore`).

---

## 1. Import the xlsx into Google Sheets

1. Go to <https://sheets.google.com>, then **Blank spreadsheet**.
2. **File → Import → Upload**, then pick `orders_for_google_sheets.xlsx`.
3. Choose **Replace spreadsheet**, then **Import data**.
4. Check that you have two tabs, **Orders** and **Handles**. Keep the header row (row 1) exactly as it is.
5. In **Handles**, fill in column B with each friend's Telegram handle, e.g. `@phoebe_xx`.
   For several handles, use commas: `@one, @two`. The `@` is optional.

## 2. Publish both tabs as CSV and paste the links into the page

1. In the sheet: **File → Share → Publish to web**.
2. In the first dropdown pick **Orders**. In the second pick **Comma-separated values (.csv)**.
3. Click **Publish**, confirm, and copy the link. It ends in `output=csv`.
4. Change the first dropdown to **Handles**, keep **.csv**, and copy that link too.
5. Leave **"Automatically republish when changes are made"** ticked (under *Published content & settings*).
6. Open `index.html` and paste the links into the CONFIG block near the top:

   ```js
   const ORDERS_CSV_URL  = "https://docs.google.com/spreadsheets/d/e/2PACX-.../pub?gid=0&single=true&output=csv";
   const HANDLES_CSV_URL = "https://docs.google.com/spreadsheets/d/e/2PACX-.../pub?gid=123456&single=true&output=csv";
   ```

   Edit it on GitHub: open `index.html`, click the ✏️ pencil, paste, then **Commit changes**.

## 3. Host it for free

### Option A: GitHub Pages (preferred)

Free GitHub Pages only works on **public** repos. This repo is private, so either:

- **make it public** (Settings → General → Danger Zone → Change visibility). This is safe because the repo only contains the page, not the data. Or
- create a new **public** repo just for the page (e.g. `order-tracker`) and upload `index.html` to it.

Then:

1. Repo → **Settings → Pages**.
2. **Source: Deploy from a branch**, **Branch: `main`**, folder **`/ (root)`**, then **Save**.
   (If `index.html` is on another branch, merge it into `main` first, or pick that branch here.)
3. After a minute or two the page is at `https://<your-username>.github.io/<repo-name>/`.
4. Send that link to your friends. They can also bookmark `…/#their_handle` to jump straight to their orders.

### Option B: Netlify Drop

1. Put `index.html` (with the URLs pasted in) in a folder on your computer.
2. Go to <https://app.netlify.com/drop> and drag the folder onto the page.
3. You get a link straight away. Sign up (free) if you want it to stay up and get a nicer name.
   To update, drag the folder again.

## 4. Day to day

You only edit the **Orders** tab, and mostly just two columns (both have dropdowns):

| Column | Values | Meaning |
|---|---|---|
| **Payment status** | `NOT PAID` | Counts toward "still to pay" |
| | `PAID` | Done, ignored in the total |
| | `DEDUCTED` | Settled by netting off, ignored in the total |
| | `CREDIT` | You owe them. Use a **negative** SGD, which reduces their total |
| | *(blank)* | Not confirmed yet. Shows "TO CONFIRM", ignored in the total |
| **Stage** | `ORDERED` → `AT WAREHOUSE` → `SHIPPING TO SG` → `DELIVERED` | Moves the 4-step tracker |
| | `NO UPDATE` | Tracker stays grey ("No update yet") |
| | *(blank)* | Money-only line. Shown under "Credits & adjustments" with no tracker |

When a consolidation batch is sent off, set **Batch** (e.g. `Batch 4`) and move those rows' Stage along.
For a new drop, add rows at the bottom: one row per person per item, the same as the existing ones.
For a new friend, add a row to **Handles** as well.

Changes show up on the page after Google re-publishes, which usually takes **about 5 minutes**. Friends just refresh.

## One workbook for everything (optional, recommended)

`DX_JP_Goods_combined.xlsx` (not in git) holds all your original tabs **plus** Orders and Handles:

- **Orders** amounts are live formulas, e.g. SGD `=ROUND('kubo grad con goods'!I29,2)`, JPY `='kubo grad con goods'!H29`. Change a drop tab and Orders (and the website) follow.
- **Summary** status cells read from Orders (`=IF(Orders!G2="","",Orders!G2)`) and each "AMOUNT YET TO PAY" is a `SUMIFS` over Orders. So you change a status in **one place: Orders**.

**Publishing:** in *Publish to web → Published content & settings*, choose **only Orders and Handles**, not "Entire document". Otherwise every tab is public.

**Don't sort the Orders tab** (Summary points at specific rows). Use *Data → Filter views* to sort or filter just for yourself. Add new rows at the bottom.

### Adding a new drop

1. Make the drop tab the way you always do (per-person blocks with a GRAND TOTAL).
2. In Orders, add one row per person: copy a row from an older drop and change the tab name and cells,
   e.g. SGD `=ROUND('new drop'!I24,2)`, JPY `='new drop'!H24`, Rate `='new drop'!D4`.
3. If you keep a Summary block for that friend, add a line with status `=IF(Orders!G66="","",Orders!G66)`; their total updates by itself.

## Privacy, in plain words

- The page only ever shows the rows for the exact name or handle typed. It never lists, suggests or autocompletes other people.
- But the published CSV link is public. Anyone who opens the page source can find it and see the whole sheet.
  Only names, items and amounts are in it, so that's the trade-off for "no logins". Don't put anything more sensitive in the sheet.
- Anyone who knows or guesses a friend's exact name can see that friend's orders. Using Telegram handles helps.
- The page has `noindex`, so search engines are asked not to list it.
