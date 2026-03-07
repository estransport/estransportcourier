# ES Transportation — Ride Tracking System Setup Guide
## One-time setup. Takes about 20 minutes.

---

## STEP 1 — Create Your Google Sheet

1. Go to sheets.google.com and create a new sheet
2. Name it: **ES Transportation Orders**
3. Add these exact headers in Row 1:

| A | B | C | D | E | F | G | H |
|---|---|---|---|---|---|---|---|
| Order # | Name | Pickup | Drop-off | Date/Time | Status | Notes | Extra Stop |

- **Extra Stop** column: enter `yes` if the ride has an extra stop, leave blank or `no` if not.

4. Add a test row:

| ES-1001 | Test Customer | 123 Main St Katy | IAH Terminal A | Mar 10 9:00AM | Submitted | | no |

---

## STEP 2 — Publish the Sheet as CSV (for tracking.html)

1. In Google Sheets → **File → Share → Publish to web**
2. Choose **Sheet1** and **Comma-separated values (.csv)**
3. Click **Publish** → copy the URL
4. Open **tracking.html**, find this line and replace it:
   ```
   const SHEET_CSV_URL = 'YOUR_GOOGLE_SHEET_CSV_URL_HERE';
   ```

---

## STEP 3 — Create a Google Apps Script (for admin.html)

1. In your Google Sheet → **Extensions → Apps Script**
2. Delete existing code and paste this:

```javascript
function doGet(e) {
  const sheet     = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  const order     = e.parameter.order.toUpperCase();
  const status    = e.parameter.status;
  const notes     = e.parameter.notes     || '';
  const extraStop = e.parameter.extra_stop || 'no';
  const data      = sheet.getDataRange().getValues();

  for (let i = 1; i < data.length; i++) {
    if (data[i][0].toString().toUpperCase() === order) {
      sheet.getRange(i + 1, 6).setValue(status);     // Column F = Status
      sheet.getRange(i + 1, 7).setValue(notes);      // Column G = Notes
      sheet.getRange(i + 1, 8).setValue(extraStop);  // Column H = Extra Stop
      return ContentService.createTextOutput('OK');
    }
  }
  return ContentService.createTextOutput('NOT FOUND');
}
```

3. Click **Save** → **Deploy → New deployment**
4. Type: **Web app** · Execute as: **Me** · Who has access: **Anyone**
5. Click **Deploy** → copy the Web App URL
6. Open **admin.html**, find this line and replace it:
   ```
   const APPS_SCRIPT_URL = 'YOUR_APPS_SCRIPT_URL_HERE';
   ```

---

## STEP 4 — Change Your Admin PIN

In **admin.html**, find this line and set your own PIN:
```
const ADMIN_PIN = '1234';
```

---

## STEP 5 — Add New Orders to the Sheet

Every new booking gets a row:
- **Order #**: ES-1001, ES-1002, etc.
- **Status**: Start with `Submitted`
- **Extra Stop**: `yes` or `no`
- Text the customer their order number after booking

---

## STEP 6 — Upload to GitHub Pages

Upload both files to your existing GitHub repo:
- `tracking.html`
- `admin.html`

Customer tracking link:
`https://estransport.github.io/estransportcourier/tracking.html`

Keep admin.html private — only share with your partner.

---

## STATUS STAGES

**Standard ride (no extra stop):**
Submitted → Reviewed / Approved → Booking Paid → Driver En Route → At Pickup → In Route → At Drop-off → Completed

**Ride with extra stop:**
Submitted → Reviewed / Approved → Booking Paid → Driver En Route → At Pickup → Extra Stop → In Route → At Drop-off → Completed

---

## DAILY USE

1. Open admin.html on your phone
2. Enter order number
3. Check "Extra Stop" if applicable
4. Tap the new status
5. Add a note if helpful (e.g. "Driver is 5 min away")
6. Tap Update — done in 10 seconds
