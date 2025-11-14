# Project Specification: On-the-Go AppSuite v2.0

## 1. Overview

This document specifies a complete, free, self-hosted On-the-Go AppSuite (OTG AppSuite). The system is **modular**, allowing a "Deployment Administrator" to deploy either:

1.  **Simple Mode (Safety-Only):** A classic Lone Worker Safety (LWS) system with a worker app, monitor app, and safety logging.
2.  **Advanced Mode (Safety + Reporting):** The full suite, which adds visit reporting, custom checklists, AI-powered grammar correction, PDF report generation, and longitudinal data tracking.

The system is delivered via a single `setup.html` deployment tool that generates the required application files based on the administrator's choice.

## 2. Core Components & File Structure

The project repository (`lws-deploy-tool`) contains the setup tool and a `templates` folder.

## 3. The Deployment Tool (`setup.html`)

This is the central tool for generating the custom apps.

### 3.1 UI and Workflow

The tool is a 6-step web-based wizard:

* **Step 1: Welcome:** Fetches all required template files from the `/templates/` folder on `window.onload`. The "Start Setup" button is disabled until all files are loaded.
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
    * **`toggleReporting()` Function:** When the checkbox is clicked, this function updates:
        * `headersDisplayText` in Step 2.
        * `checklistsInstruction` visibility in Step 2.
        * `scriptPreview` to show the correct `_simple` or `_advanced` script.
    * **"Copy Script" Button:** Calls `copyInjectedScript()`.
    * **`copyInjectedScript()` Function:** Reads the "Advanced" checkbox. It selects `apps_script_simple.txt` or `apps_script_advanced.txt`, injects `%%SECRET_KEY%%`, `%%ORGANISATION_NAME%%`, and (if advanced) `%%GEMINI_API_KEY%%`, then copies the result to the clipboard.
* **Step 4: Configure Apps:**
    * **`Web App URL` (Input):** User pastes their deployed URL here.
    * **Other Inputs:** `firstAlert`, `enableCheckin`, `checkinInterval`, `logoUrl`.
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
* **`checkOverdueWorkers()`:** The time-triggered function. Scans the `Visits` sheet for active workers, checks their `Anticipated Departure Time` against `now()`, and sends escalating alerts by calling `sendAlertEmail()`.
* **`sendAlertEmail()`:** Sends formatted HTML emails for all alert types (`PANIC`, `DURESS`, `MISSED_CHECKIN`, `EMAIL_1_SENT`, etc.).

### 4.2. `apps_script_simple.txt` (Simple Mode)
* **Placeholders:** `%%SECRET_KEY%%`, `%%ORGANISATION_NAME%%`.
* **`doGet(e)`:** Only handles the Monitor App. It validates the `token` and returns all data from the `Visits` sheet as JSONP.
* **Schema:** Expects the 19-column `Visits` sheet.

### 4.3. `apps_script_advanced.txt` (Advanced Mode)
* **Placeholders:** `%%SECRET_KEY%%`, `%%ORGANISATION_NAME%%`, `%%GEMINI_API_KEY%%`.
* **Config Vars:** `DEFAULT_REPORT_TEMPLATE_ID`, `PDF_OUTPUT_FOLDER_ID`, `LONGITUDINAL_ID_HEADER`.
* **Schema:** Expects the 21-column `Visits` sheet and the `Checklists` sheet.
* **`doGet(e)`:** Handles two request types:
    * **Monitor (JSONP):** Same as simple mode.
    * **Worker (JSON):** If `action=getForms`, it performs a **keyless** read of the `Checklists` sheet for the given `companyName` and returns a JSON array of question objects (e.g., `{type: "header", text: "..."}`).
* **`_aggregateDataForMonth()`:** A private helper function that performs the core data aggregation for all reporting. It reads the `Visits` sheet, filters by month, and groups data by `Company Name - Location Name` and `Company Name (Combined)`. It returns a single object containing `masterReport`, `combinedReport`, `templateIdMap`, etc.
* **`generateMasterMonthlyReport()`:** Calls `_aggregateDataForMonth`, runs `runAICorrection` on the data, and creates a new spreadsheet **tab** for each site and combined total.
* **`generateAllPdfReports()`:** Calls `_aggregateDataForMonth`, runs `runAICorrection`, then loops through each combined company. It finds the correct `templateId` (from `templateIdMap` or default), copies the Google Doc, replaces all `{{...}}` tags, builds and inserts the tables using `buildTableInDoc`, and saves the final file as a PDF in the `PDF_OUTPUT_FOLDER_ID`.
* **`runAllLongitudinalReports()`:** Calls `_aggregateDataForMonth`, then loops through each company, opens their specific spreadsheet (using the ID from `longitudinalIdMap`), and appends the month's summary data as a new row.
* **`createLongitudinalWorkbook()`:** A utility function run manually by the admin to create a new spreadsheet for a company and save its ID back to the `Checklists` sheet.
* **Helper Functions:** `runAICorrection` (handles AI calls), `writeReportToTab` (builds spreadsheet tabs), `buildTableInDoc` (builds Google Doc tables).

---

## 5. Worker App (PWA)

### 5.1. Common Features (Both Apps)
* **`localStorage` State:** `settings`, `locations`, `activeVisit`.
* **`loadState()`:** Initializes the app. Must create the default `"Travelling"` location if it doesn't exist.
* **Safety Logic:**
    * `handleArriveLongPress`: Calls `executeStartVisit`.
    * `executeStartVisit`: Sends `ON SITE` data via `sendToGoogleSheet`.
    * `handleExtendLongPress`: Prompts for PIN, calls `withPinVerification`, sends `Notes` update.
    * `triggerPanicAlert`: Sends `EMERGENCY - PANIC BUTTON` data, attempts to get GPS, and sends that as well. **Must** handle `gpsData.longitude` correctly.
* **PIN Logic:** `withPinVerification` must check for both `pinCode` (normal) and `duressPin`.
* **Alerts:** `handleSafeLongPress` and `tick` function for check-ins, pre-alerts, and overdue alerts.

### 5.2. `worker_app_simple.txt` (Simple Mode)
* **Location Modal:** Only collects `Location Name` and `Location Address`.
* **`handleDepartLongPress`:** Calls `depart()`.
* **`depart()`:** Simply sends `DEPARTED` status and `Battery:` note.
* **`handleSafeLongPress`:** Calls `withPinVerification`. On success, calls `iamSafe()`.
* **`iamSafe()`:** Simply sends `SAFE - MANUALLY CLEARED` status.

### 5.3. `worker_app_advanced.txt` (Advanced Mode)
* **`localStorage` State:** Adds `cachedForms: {}` and `pendingUploads: []`.
* **Location Modal:** Collects `Location Name`, `Location Address`, **`Company Name`**, and **`Template Name (Optional)`**.
* **`handleDepartLongPress`:** Calls `showReportModal(false)`.
* **`handleSafeLongPress`:** Calls `withPinVerification`. On normal PIN, it checks if `location.name === 'Travelling'`. If so, it calls `iamSafe(false)`. If not, it calls `showReportModal(true)`.
* **`showReportModal(isFromAlert)`:**
    * Skips modal entirely if `location.name === 'Travelling'`.
    * Gets `templateName` from location. If blank, sets `formToLoad = "(Standard)"`.
    * Loads form from `state.cachedForms[formToLoad]`.
    * If not found, loads `state.cachedForms["(Standard)"]`.
    * If *still* not found, loads hardcoded `STANDARD_CHECKLIST_FALLBACK`.
* **`buildReportForm(items)`:** Renders HTML based on `item.type` (`header`, `textarea`, `checkbox`).
* **`submitReport()`:** Gathers all checklist/notes into a JSON object and passes it to `depart()` or `iamSafe()` in the `reportData` parameter.
* **`sendToGoogleSheet()`:** Is modified to catch errors and add `DEPARTED` or `SAFE` reports to the `pendingUploads` queue if the network is offline.
* **`syncAllForms()`:** On app load, *always* calls `fetchForm("(Standard)")`. Also loops through all `locations` and fetches any unique `templateName`s not in the cache.
* **`fetchForm(companyName)`:** Makes a **keyless** `GET` request to the script URL (`?action=getForms...`).
* **`processUploadQueue()`:** Runs on app load, re-sends any reports in `pendingUploads`.

---

## 6. Monitor App (`monitor_app_template.txt`)
* **Placeholders:** `%%LOGO_URL%%`, `%%ORGANISATION_NAME%%`.
* **Login:** `setupPage` is shown first. `saveUrlBtn` tests the URL/Key with a JSONP ping. On success, saves to `localStorage` and calls `Maps('dashboard')`.
* **`fetchData()`:** Runs on a 15-second `setInterval`. Performs a JSONP request with the saved token.
* **`processData()`:** Filters for `activeStatuses` and sorts by `statusPriority`.
* **`renderWorkers()`:**
    * Renders one card per worker.
    * Sets card color based on `Alarm Status`.
    * Shows `(Batt: N/A)` if `Battery Level` is null.
    * **Reads `Battery Level` column** (not the `Notes` field) for battery status.
    * Shows "STALE GPS" warning if `Location Name` is "Travelling" and `GPS Timestamp` is > 30 mins old.
* **`checkForNewAlerts()`:** Compares new data to `lastKnownStatuses` to trigger `playAlarm()` or `playUpdateSound()` and show a `Notification`.

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
* `%%GEMINI_API_KEY%%` (Advanced backend script)
* `%%GOOGLE_SHEET_URL%%` (Both worker apps)
* `%%FIRST_ALERT_MINUTES%%` (Both worker apps)
* `%%CHECKIN_ENABLED%%` (Both worker apps)
* `%%CHECKIN_INTERVAL%%` (Both worker apps)
* `%%LOGO_URL%%` (Default: `https://i.postimg.cc/dVmVg3Pn/favicon-logo-for-LWSApp.png`) (All apps, manifest)

### Injected by User (in Google Doc Template):
* `{{CompanyName}}`
* `{{ReportMonth}}`
* `{{TotalVisits}}`
* `{{TotalHours}}`
* `{{ChecklistTable}}` (Tag to be replaced by checklist table)
* `{{NotesTable}}` (Tag to be replaced by notes table)

---

## 9. Documentation

The `templates` folder must contain the three source `.md` files for the final `Documentation.pdf`:
* `Deployment_Admin_Guide.md`
* `Installer Creation Guide.md`
* `Promotional Blurb.md`

The main `Documentation.pdf` must be manually compiled from these and hosted at the root of the setup tool repository.
```
