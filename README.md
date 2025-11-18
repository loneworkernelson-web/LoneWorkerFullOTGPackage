# Project Specification: On-the-Go AppSuite v2.3

## 1. Overview

This document specifies a complete, free, self-hosted On-the-Go AppSuite (OTG AppSuite). The system is **modular**, allowing a "Deployment Administrator" to deploy either:

1.  **Simple Mode (Safety-Only):** A classic Lone Worker Safety (LWS) system with a worker app, monitor app, and safety logging.
2.  **Advanced Mode (Safety + Reporting):** The full suite, which adds visit reporting, custom checklists (including numerical data entry), AI-powered grammar correction, PDF report generation, longitudinal data tracking, and "Quick Stat" logging.

The system is delivered via a single `setup.html` deployment tool that generates the required application files based on the administrator's choice.

## 2. Core Components & File Structure

The project repository (`lws-deploy-tool`) contains the setup tool and a `templates` folder.
/ (Repository Root) 
├── setup.html 
│ └── templates/ 
         ├── apps_script_simple.txt 
         ├── apps_script_advanced.txt 
         ├── worker_app_simple.txt 
         ├── worker_app_advanced.txt 
         ├── monitor_app_template.txt 
         ├── manifest.json 
         ├── sw.js 
         ├── Deployment_Admin_Guide.md 
         ├── Installer Creation Guide.md 
         └── Promotional Blurb.md
## 3. The Deployment Tool (`setup.html`)

This is the central tool for generating the custom apps.

### 3.1 UI and Workflow

The tool is a 6-step web-based wizard:

* **Step 1: Welcome:** Fetches all required template files from the `/templates/` folder on `window.onload`.
* **Step 2: Sheet Setup:** Instructs the user to create a Google Sheet.
    * **`Visits` Sheet:** All users must create this tab.
    * **`Checklists` Sheet:** This instruction is hidden by default and only appears if "Advanced Reporting" is enabled in Step 3.
    * **"Copy Headers" Button:** This button's behavior is dynamic.
        * If "Advanced Reporting" is OFF, it copies the 19 headers for the Simple build.
        * If "Advanced Reporting" is ON, it copies the 21 headers for the Advanced build (adding `Company Name` and `Visit Report Data`).
* **Step 3: Configure & Deploy Script:**
    * **`Organisation Name` (Input):** Required. Used for branding.
    * **`Secret Key` (Input):** Required. Used for Monitor App auth.
    * **`[ ] Enable Advanced Visit Reporting` (Checkbox):**
        * `onchange` event calls `toggleReporting()`.
    * **`Advanced Reporting Options` (Hidden Div):**
        * Contains `Gemini API Key` (Input). This div is shown/hidden by `toggleReporting()`.
    * **`toggleReporting()` Function:** When the checkbox is clicked, this function updates `headersDisplayText`, `checklistsInstruction` visibility, and the `scriptPreview`.
    * **"Copy Script" Button:** Calls `copyInjectedScript()`.
    * **`copyInjectedScript()` Function:** Reads the "Advanced" checkbox. It selects `apps_script_simple.txt` or `apps_script_advanced.txt`, injects all `%%...%%` placeholders, and copies the result to the clipboard.
* **Step 4: Configure Apps:**
    * **`Web App URL` (Input):** User pastes their deployed URL here.
    * **Other Inputs:** `firstAlert`, `escalationMinutes`, `enableCheckin`, `checkinInterval`, `logoUrl`.
    * **"Test Connection" Button:** Calls `testAndProceed()`.
    * **`testAndProceed()` Function:** Performs a JSONP "ping" (`?callback=...&token=...`) to the user's `Web app URL` to validate the URL and `Secret Key`.
* **Step 5: Download Apps:**
    * **`generateZip()` Function:** This core function gathers all config values and:
        1.  Selects the correct `worker_app_...txt` and `apps_script_...txt` based on the "Advanced" checkbox.
        2.  Replaces all placeholders (see Section 8) in all template files.
        3.  Creates a `.zip` file containing `WorkerApp/`, `MonitorApp/`, and `01-PASTE_THIS_INTO_APPS_SCRIPT.txt`.
* **Step 6: Finish:** Links to the `Documentation.pdf`.

---

## 4. Backend (`apps_script_...txt`)

### 4.1. Common Features (Both Scripts)
* **`doPost(e)`:** The main entry point for all data from the Worker App. It finds the correct row (by `Worker Name` + `Arrival Time`) or appends a new one, saving all data. It parses the `Notes` field for `Battery: XX%` and saves the number to the `Battery Level` column.
    * **Alert Timestamping:** When a `PANIC`, `DURESS`, or `MISSED_CHECKIN` alert is received, this function writes the current `new Date().toISOString()` to the `GPS Timestamp` column to act as the "alert start time" for escalation.
* **`checkOverdueWorkers()`:** The time-triggered function. Scans the `Visits` sheet.
    * **Resolved Statuses:** It maintains a `resolvedStatuses` array (e.g., `['DEPARTED', 'MONITOR_CLEARED_ALERT', 'DATA_ENTRY_ONLY']`) and ignores any worker with these statuses.
    * **New Overdue:** Finds `ON SITE` workers whose `Anticipated Departure Time` has passed, changes their status to `EMAIL_1_SENT`, and sets their `GPS Timestamp` to start the escalation clock.
    * **Escalation Logic:** Finds any *unresolved* alert (e.g., `PANIC`, `EMAIL_1_SENT`). It checks the "alert age" (by comparing `now()` to the `GPS Timestamp`) against the `ESCALATION_MINUTES` variable. If the age exceeds the threshold, it sends a new alert (email + SMS) to the *secondary* contact and updates the status (e.g., to `ESCALATION_SENT` or `EMAIL_2_SENT`).
* **`sendSmsViaGateway(phoneNumber, message)`:** (New in v2.0) Sends a free SMS alert using the TextBelt API's free tier (`key: "textbelt"`).
    * **Fallback:** If `!result.success`, it logs the error and sends a **fallback email** to the script owner (`Session.getEffectiveUser().getEmail()`) notifying them of the SMS failure.
* **`sendAlertEmail(workerData, alertType, isEscalation)`:** Sends formatted HTML emails. It determines the correct recipient (primary or escalation contact) based on the `isEscalation` boolean. For critical alerts, it also generates a short SMS summary and calls `sendSmsViaGateway`.

### 4.2. `apps_script_simple.txt` (Simple Mode)
* **Placeholders:** `%%SECRET_KEY%%`, `%%ORGANISATION_NAME%%`, `%%ESCALATION_MINUTES%%`, `%%FIRST_ALERT_MINUTES%%`.
* **`doGet(e)`:** Only handles the Monitor App. It validates the `token` and returns all data from the `Visits` sheet as JSONP.
* **Schema:** Expects the 19-column `Visits` sheet.

### 4.3. `apps_script_advanced.txt` (Advanced Mode)
* **Placeholders:** `%%SECRET_KEY%%`, `%%ORGANISATION_NAME%%`, `%%ESCALATION_MINUTES%%`, `%%FIRST_ALERT_MINUTES%%`, `%%GEMINI_API_KEY%%`.
* **Config Vars:** `DEFAULT_REPORT_TEMPLATE_ID`, `PDF_OUTPUT_FOLDER_ID`, `LONGITUDINAL_ID_HEADER`.
* **Schema:** Expects the 21-column `Visits` sheet and the `Checklists` sheet.
* **`doGet(e)`:** Handles two request types:
    * **Monitor (JSONP):** Same as simple mode.
    * **Worker (JSON):** If `action=getForms`, it performs a **keyless** read of the `Checklists` sheet for the given `companyName`. It parses questions and returns a JSON array of objects:
        * `"#"` -> `{type: "header", ...}`
        * `"%"` -> `{type: "textarea", ...}`
        * `"$"` -> `{type: "number", ...}`
        * Other -> `{type: "checkbox", ...}`
* **Reporting Functions:**
    * **`_aggregateDataForMonth()`:** A private helper function. It aggregates data from the `Visits` sheet, calculates `questionTally` (checkboxes) and **`numericTally`** (SUM of all `$` fields), and compiles notes.
    * **`generateMasterMonthlyReport()`:** Calls aggregation, runs AI correction, and creates spreadsheet tabs. It adds a specific section for "Numeric Totals."
    * **`generateAllPdfReports()`:** Calls aggregation, runs AI correction, then merges data into Google Doc templates. It supports the new `{{NumericTotalsTable}}` tag.
    * **`runAllLongitudinalReports()`:** Appends summary data (including numeric totals) to external spreadsheets defined in the `Checklists` sheet.
    * **`createLongitudinalWorkbook()`:** Creates the external sheet and adds columns for all `$` fields found in the template.
* **Database Maintenance:**
    * **`archiveOldData()`:** A time-triggered function that moves rows older than 60 days from `Visits` to a separate `OTG_Archive_[Year]` spreadsheet to maintain performance.

---

## 5. Worker App (PWA)

### 5.1. Common Features
* **Screen Wake Lock:** Implements the `navigator.wakeLock` API to prevent the phone from sleeping/freezing the app while a visit is active.
* **Safety Logic:** `handleArriveLongPress`, `triggerPanicAlert` (with GPS timestamp fix), `handleExtendLongPress`.

### 5.2. `worker_app_simple.txt` (Simple Mode)
* **Placeholders:** `%%ORGANISATION_NAME%%`, `%%FIRST_ALERT_MINUTES%%`, `%%ESCALATION_MINUTES%%`, `%%CHECKIN_ENABLED%%`, `%%CHECKIN_INTERVAL%%`, `%%GOOGLE_SHEET_URL%%`, `https://i.postimg.cc/dVmVg3Pn/favicon-logo-for-LWSApp.png`
* **Location Modal:** Only collects `Location Name` and `Location Address`.
* **`depart()`:** Simply sends `DEPARTED` status and `Battery:` note.

### 5.3. `worker_app_advanced.txt` (Advanced Mode)
* **Placeholders:** (Same as Simple Mode)
* **`localStorage` State:** Adds `cachedForms: {}` and `pendingUploads: []`.
* **UI:** Adds a **"Quick Stat"** button to the main page next to the "Start" button.
* **Location Modal:** Collects `Location Name`, `Location Address`, **`Report As: (Company Name)`**, **`Use Template: (Optional)`**, and **`[ ] No Report Required`**.
* **Logic Branching:**
    * **"Travelling" or "No Report Required":** Skips the report modal entirely on Depart/Safe.
    * **"Quick Stat" Button:** Opens the report modal immediately (no timer), sets status to `DATA_ENTRY_ONLY`.
    * **Standard Visit:** Opens the report modal on Depart/Safe.
* **`buildReportForm(items)`:** Renders HTML based on `item.type`:
    * `header` -> `<h3>`
    * `textarea` -> `<textarea class="wsa-custom-note">`
    * `number` -> `<input type="number" class="wsa-custom-number">`
    * `checkbox` -> `<input type="checkbox">`
* **`gatherFormData()`:** Collects checklist, notes, and numeric entries into a JSON object.
* **`submitQuickStatReport()`:** Sends data with `DATA_ENTRY_ONLY` status and `Arrival/Departure` time as `now()`.
* **Troubleshooting:** A "Clear Cached Forms & Data" button in Settings uses a custom `showConfirm` modal (instead of `window.confirm`) for safety.

---

## 6. Monitor App (`monitor_app_template.txt`)
* **Placeholders:** `%%LOGO_URL%%`, `%%ORGANISATION_NAME%%`.
* **Login:** `setupPage` is shown first. `saveUrlBtn` tests the URL/Key with a JSONP ping.
* **`processData()`:** Filters for `activeStatuses`. The list **excludes** `DATA_ENTRY_ONLY`, so Quick Stats do not appear on the dashboard.
* **`renderWorkers()`:**
    * Renders one card per worker.
    * **Reads `Battery Level` column** for battery status.
    * Shows "STALE GPS" warning if `Location Name` is "Travelling" and `GPS Timestamp` is > 30 mins old.
* **Audio:** Uses `Tone.js` for alarms and chimes. Requires user interaction to initialize AudioContext.

---

## 7. Shared PWA Files

* **`manifest.json`:** Standard PWA manifest.
    * Placeholders: `%%ORGANISATION_NAME%%` (in `name`), `https://i.postimg.cc/dVmVg3Pn/favicon-logo-for-LWSApp.png` (as default logo URL).
* **`sw.js`:** A standard cache-first service worker. Caches `index.html`, Tailwind, Tone.js, and Google Fonts.

---

## 8. Master Placeholder List

### Injected by Setup Tool:
* `%%ORGANISATION_NAME%%` (All apps, manifest, backend scripts)
* `%%SECRET_KEY%%` (Both backend scripts)
* `%%ESCALATION_MINUTES%%` (Both backend scripts)
* `%%FIRST_ALERT_MINUTES%%` (Both backend scripts)
* `%%GEMINI_API_KEY%%` (Advanced backend script only)
* `%%GOOGLE_SHEET_URL%%` (Both worker apps)
* `%%CHECKIN_ENABLED%%` (Both worker apps)
* `%%CHECKIN_INTERVAL%%` (Both worker apps)
* `%%LOGO_URL%%` (Default: `https://i.postimg.cc/dVmVg3Pn/favicon-logo-for-LWSApp.png`) (All apps, manifest)

### Injected by User (in Google Doc Template):
* `{{CompanyName}}`
* `{{ReportMonth}}`
* `{{TotalVisits}}`
* `{{TotalHours}}`
* `{{ChecklistTable}}`
* `{{NumericTotalsTable}}`
* `{{NotesTable}}`

---

## 9. Documentation

The `templates` folder must contain the three source `.md` files for the final `Documentation.pdf`:
* `Deployment_Admin_Guide.md`
* `Installer Creation Guide.md`
* `Promotional Blurb.md`
