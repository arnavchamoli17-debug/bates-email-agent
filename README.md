# Bates Number Email Agent (Google Apps Script)

An automated Google Apps Script agent that runs periodically on a timer, searches Gmail for target emails, extracts Bates numbers (from subject lines, attachment filenames, or message bodies), and logs the findings directly to a Google Sheet.

---

## Files in this Project

- **`Code.gs`**: The core script containing constants, search and extraction logic, `testRun()`, and automated 15-minute trigger configuration.
- **`appsscript.json`**: Project manifest configured for the modern V8 runtime with the required OAuth scopes (`gmail.modify`, `spreadsheets`, `script.scriptapp`).

---

## Step-by-Step Deployment Instructions

Follow these exact steps to deploy and run this project in your Google account:

### Step 1: Open Google Apps Script
1. Navigate to [script.google.com](https://script.google.com/) in your web browser.
2. In the top-left corner, click **+ New project**.
3. (Optional) Rename your project from *"Untitled project"* to something descriptive like **"Bates Email Agent"** by clicking the title at the top.

> **Alternative method:** You can also open the target Google Sheet, click **Extensions** in the top menu bar, and choose **Apps Script**.

---

### Step 2: Configure the Manifest (`appsscript.json`)
By default, Google Apps Script hides the manifest file. To show and edit it:

1. In the left navigation panel of the Apps Script editor, click the **Gear icon (Project Settings)**.
2. Under the **General Settings** section, check the box:
   - ☑️ **"Show 'appsscript.json' manifest file in editor"**.
3. In the left panel, click the **`< >` (Editor)** icon to return to the code editor.
4. You will now see `appsscript.json` listed under **Files**. Click it.
5. Replace everything in `appsscript.json` with the contents of the local [`appsscript.json`](./appsscript.json) file:
   ```json
   {
     "timeZone": "Etc/GMT",
     "dependencies": {},
     "exceptionLogging": "STACKDRIVER",
     "runtimeVersion": "V8",
     "oauthScopes": [
       "https://www.googleapis.com/auth/gmail.modify",
       "https://www.googleapis.com/auth/spreadsheets",
       "https://www.googleapis.com/auth/script.scriptapp"
     ]
   }
   ```
6. Click the **Save project** button (💾 floppy disk icon or press `Cmd + S` / `Ctrl + S`).

---

### Step 3: Add the Script Code (`Code.gs`)
1. In the left file list, click on **`Code.gs`**.
2. Select all existing text (`Cmd + A` / `Ctrl + A`) and delete it.
3. Copy and paste the entire contents of [`Code.gs`](./Code.gs) into the editor.
4. Review the constants at the top:
   - **`SHEET_ID`**: Set to your Google Sheet ID (`1lGkffWbN08ufS_AEJTL-Mz7Aixh26m-yrqkOiGIIPB8`).
   - **`KEYWORD`**: The keyword to look for in the subject (default: `'qwert'`).
   - **`LABEL_NAME`**: Label applied after processing (default: `'bates-logged'`).
   - **`BATES_REGEX`**: The regular expression used to find Bates numbers (default: `/\b[A-Z]{2,6}-\d{5,8}\b/i`).
   - **`SHEET_TAB_NAME`**: (Optional) Specify a tab name, or leave empty (`''`) for the first tab.
5. Click the **Save project** button (💾).

---

### Step 4: Authorize and Run a Test (`testRun()`)
Before enabling the recurring timer, run the test function to verify permissions, email parsing, and sheet logging:

1. Ensure you have at least one test email in Gmail:
   - Send an email to yourself with **`qwert`** in the subject line.
   - Include a Bates number (e.g. `ABC-000123`) in the subject, email body, or as an attachment file name.
   - Make sure this email does **not** already have the `bates-logged` label.
2. In the toolbar at the top of the Apps Script editor, locate the function dropdown menu (it may currently say `processBatesEmails` or `myFunction`).
3. Select **`testRun`** from the dropdown.
4. Click the **▶ Run** button.
5. **Authorization Flow (First time only):**
   - A dialog titled **"Authorization required"** will appear. Click **Review permissions**.
   - Select your Google Account.
   - A warning will say *"Google hasn’t verified this app"*. Click **Advanced** at the bottom left of the modal.
   - Click **Go to Bates Email Agent (unsafe)**.
   - Click **Allow** to grant permission to access Gmail and Google Sheets.
6. Once authorized, the script will run `testRun()`.
7. Check the **Execution log** panel at the bottom of the editor. You will see:
   - Email subject and received date
   - Detected attachment names
   - Extracted Bates number and where it was found
   - Confirmation that a row was appended to your Google Sheet
   - Confirmation that the `bates-logged` label was applied
8. Open your Google Sheet to verify the new row.

> **Tip for re-testing:** If you want to test again on the same email, go to Gmail and remove the `bates-logged` label from that conversation thread.

---

### Step 5: Enable the 15-Minute Automation (`createTrigger()`)
Once you have confirmed that `testRun()` works as expected:

1. In the function dropdown in the toolbar, select **`createTrigger`**.
2. Click **▶ Run**.
3. The script will remove any existing duplicate triggers and create a time-driven trigger that runs `processBatesEmails()` automatically every 15 minutes.
4. **To verify the trigger:**
   - Click the **Alarm clock icon (Triggers)** on the left sidebar.
   - You will see `processBatesEmails` listed with event source **Time-driven**, type **Minutes timer**, and frequency **Every 15 minutes**.

---

## Configuration & Customization

### Modifying the Bates Number Pattern
The Bates pattern is governed by the `BATES_REGEX` constant at the top of `Code.gs`:

```javascript
// Default: 2 to 6 letters, a hyphen, followed by 5 to 8 digits (e.g. ABC-000123)
const BATES_REGEX = /\b[A-Z]{2,6}-\d{5,8}\b/i;
```

**Alternative pattern examples:**
- If your numbers have variable prefix lengths (e.g. `A-001`, `PLTF-000456`):
  ```javascript
  const BATES_REGEX = /\b[A-Z]+-\d+\b/i;
  ```
- If your numbers can use spaces, hyphens, or underscores (e.g. `ABC 000123`, `ABC_000123`):
  ```javascript
  const BATES_REGEX = /\b[A-Z]{2,6}[-_ ]\d{4,8}\b/i;
  ```
- If your Bates numbers are purely numeric with 6 to 9 digits:
  ```javascript
  const BATES_REGEX = /\b\d{6,9}\b/;
  ```

---

## How to Stop or Pause the Agent
If you ever want to stop the 15-minute background automation:
1. Open the project at [script.google.com](https://script.google.com/).
2. Click the **Alarm clock icon (Triggers)** in the left sidebar.
3. Hover over the trigger for `processBatesEmails`.
4. Click the **three vertical dots** on the right side and select **Delete trigger**.
