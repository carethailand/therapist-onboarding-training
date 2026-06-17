# Setting up the Google Sheet to receive exam results

This makes every submitted exam land as a new row in a Google Sheet you own — trainee ID, score, pass/fail, and time used. ~15 minutes, free, no coding experience needed. Follow in order.

---

## Step 1 — Create the Sheet
1. Go to <https://sheets.google.com> and create a blank spreadsheet.
2. Name it e.g. **"Therapist Exam Results"**.
3. In row 1, type these headers across columns A–H (exactly, one per cell):

   `Submitted At` | `Trainee ID` | `Trainee Name` | `Score` | `Total` | `Percent` | `Pass/Fail` | `Time Used (min)`

## Step 2 — Add the script
1. In the Sheet menu: **Extensions → Apps Script**.
2. Delete whatever code is there, and paste this in:

```javascript
function doPost(e) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheets()[0];
  var d = JSON.parse(e.postData.contents);
  sheet.appendRow([
    d.submittedAt, d.traineeId, d.traineeName,
    d.score, d.total, d.percent,
    d.passed ? "PASS" : "FAIL", d.timeUsedMinutes
  ]);
  return ContentService.createTextOutput("ok");
}
```

3. Click the **Save** icon (💾).

## Step 3 — Publish it as a Web App
1. Top right: **Deploy → New deployment**.
2. Click the gear icon next to "Select type" → choose **Web app**.
3. Set:
   - **Description:** Exam results
   - **Execute as:** *Me*
   - **Who has access:** *Anyone*  ← important, so the exam page can post to it
4. Click **Deploy**. Approve the permissions prompt (it's your own script).
5. Copy the **Web app URL** it gives you. It looks like:
   `https://script.google.com/macros/s/AKfy....../exec`

## Step 4 — Plug it into the exam
1. Open `index.html`, find this line near the top of the `<script>` section:
   ```javascript
   const SUBMIT_ENDPOINT = "";
   ```
2. Paste your URL between the quotes:
   ```javascript
   const SUBMIT_ENDPOINT = "https://script.google.com/macros/s/AKfy....../exec";
   ```
3. Save, commit to GitHub. Done — results now flow into your Sheet automatically.

---

## How to test it
- Log in as the demo trainee, take the exam, submit.
- Within a few seconds a new row should appear in your Sheet.
- If nothing appears: re-check that "Who has access" is **Anyone**, and that the URL ends in `/exec` (not `/dev`).

## Notes
- The trainee still only sees "Thank you — we'll be in touch." Nothing on their screen reveals the score.
- If you ever change the script, you must **Deploy → Manage deployments → Edit → New version** for changes to take effect (the URL stays the same).
- This is fine for the internal practice exam. For the graded ABAT exam, results and the answer key should move fully server-side (Phase 2 in the spec) so nothing is gradeable in the browser.
