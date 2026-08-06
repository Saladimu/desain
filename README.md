# Home Inventory

A simple, self-contained web app to track household belongings for insurance and peace of mind. Runs on phone, tablet, and desktop — no build step, no dependencies, no backend required.

**Live demo:** [Home Inventory](https://4173-ca75939765836199.monkeycode-ai.live)

---

## Features

- Add, edit, and delete inventory items
- Fields: item name, category, room/location, quantity, value, date (`DD-MMM-YY`), notes
- Manage custom **categories** (add / rename / delete)
- Manage custom **rooms / locations** (add / rename / delete)
- Dashboard stats: total items, rooms in use, total value
- Search, filter by category/room, sort by name/value/room/date
- Data persists locally in the browser (`localStorage`)
- Optional **Google Sheets sync** — push records to a sheet and auto-load them on startup
- Responsive and touch-friendly (tablet, phone, desktop)

---

## Quick Start

No installation required. Just open `index.html` in any modern browser.

```bash
# Optional: serve locally (recommended, avoids any browser file restrictions)
python3 -m http.server 4173
# Then open http://localhost:4173
```

---

## Publishing on GitHub Pages

1. Create a GitHub repository and upload `index.html` (a `README.md` is optional but recommended).
2. Go to **Settings → Pages**.
3. Under **Build and deployment**, choose **Deploy from a branch** and select the branch containing the file (e.g., `main` / root folder).
4. Save. Your app will be available at `https://<username>.github.io/<repo>/`.

That's it — the app is a single self-contained file, so no build step or configuration is needed.

---

## Data Storage

### Local (default)

All records are stored in your browser's `localStorage` under the key `home_inventory_items`. Categories, rooms, and sync settings are stored under separate keys. Clearing browser data will erase your local records — use Google Sheets sync as a backup.

### Google Sheets sync (optional)

The app can push all records to a Google Sheet and automatically load them back when the page opens.

#### Setup steps

1. Create a Google Sheet (or use an existing one).
2. Menu: **Extensions → Apps Script**.
3. Delete any sample code and paste the script below:

```javascript
const SHEET = 'Inventory';

function doGet() {
  const sheet = sheet_();
  return out_(JSON.stringify(sheet.getDataRange().getValues()));
}

function doPost(e) {
  try {
    const payload = JSON.parse(e.postData.contents);
    const headers = ['ID', 'Name', 'Category', 'Room', 'Date', 'Quantity', 'Value', 'Notes', 'DateAdded'];
    const rows = (payload.items || []).map(function (it) {
      return [it.id, it.name, it.category, it.room, it.date, it.quantity, it.value, it.notes, it.added];
    });
    const s = sheet_();
    s.clearContents();
    s.getRange(1, 1, 1, headers.length).setValues([headers]);
    if (rows.length) {
      s.getRange(2, 1, rows.length, headers.length).setValues(rows);
    }
    return out_('OK:' + rows.length + ' rows written');
  } catch (err) {
    return out_('ERROR:' + err.message);
  }
}

function sheet_() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  return ss.getSheetByName(SHEET) || ss.insertSheet(SHEET);
}

function out_(text) {
  return ContentService.createTextOutput(text).setMimeType(ContentService.MimeType.TEXT);
}
```

4. Click **Deploy → New deployment → Web app**.
   - **Execute as:** Me
   - **Who has access:** Anyone
   - Click **Deploy** and authorize when prompted.
5. Copy the **Web app URL** (ends in `/exec`).
6. In the app, click **Settings**, paste the URL, optionally enable **Auto-sync after every change**, and click **Save**.

#### How sync works

- **On startup:** the app fetches all rows from the sheet and loads them (falls back to local records if the sheet is unavailable).
- **Sync to Sheet:** button in the toolbar pushes all current records. The sheet is rewritten (cleared then written) with the full inventory.
- **Auto-sync:** when enabled, pushes automatically after every add / edit / delete.

#### Sheet layout

| ID | Name | Category | Room | Date | Quantity | Value | Notes | DateAdded |
|----|------|----------|------|------|----------|-------|-------|-----------|
| a1 | Sofa | Furniture | Living Room | 05-Aug-26 | 1 | 500 | bought 2024 | 1700000000000 |

Column names are matched by header, so you can reorder columns in the sheet. Blank rows are skipped when loading.

---

## Project Structure

```
.
├── index.html   # The entire app (HTML, CSS, and JavaScript in one file)
└── README.md    # This file
```

---

## Browser Support

- Modern Chrome, Firefox, Edge, and Safari
- Touch devices: 44px tap targets, 16px inputs (prevents mobile focus-zoom)
- Responsive breakpoints for phone (<600px), tablet (≤900px), and desktop

---

## Notes / Limitations

- Data is stored per browser/device. Different devices have separate local records unless you use Google Sheets sync.
- The Apps Script web app URL should be treated as semi-public (anyone with the link could read/write the sheet). Use a dedicated sheet for the inventory data if this is a concern.
- Requires an internet connection only for Google Sheets sync; the app itself works fully offline.
