# Deployment Administrator Guide: On-the-Go AppSuite

Welcome! This guide explains how to use the online Setup Tool to create your customized On-the-Go AppSuite (OTG AppSuite) and deploy them using GitHub Pages.

**Goal:** To generate and host your own private, secure, and branded versions of the Worker App and Monitoring Dashboard. This guide will walk you through the entire process from a blank spreadsheet to live, functioning apps.

**Estimated Time:**

* Reading: 15-20 minutes

* Set-up: 20-60 minutes. This is a one-time process. While it may seem technical, following these steps precisely will result in a stable and secure system.

## **Table of Contents**

* **Part 1:** Use the Online Setup Tool

* **Part 2:** Host Your Generated Apps using GitHub Pages

* **Part 3:** Distribute to Your Team

* **Part 4:** (Optional) How to Get a Gemini API Key

* **Part 5:** (Optional) Spreadsheet Administrator Guide

* **Part 6:** Troubleshooting Guide

## **Part 1: Use the Online Setup Tool**

Your system administrator has provided a link to the OTG AppSuite Setup Tool website. This tool is a simple wizard that gathers your settings, injects them into the application templates, and bundles them as a `.zip` file for you.

1. **Open** the Setup Tool Link in your web browser.

2. Follow the steps presented in the tool:

   ### **Step 1: Welcome**

   * Read the introduction. The "Start Setup" button will fetch all the necessary code templates from the server. If this step fails, ensure you are online and that the `setup.html` file is being hosted on a web server (like GitHub Pages), not run from a local file (`file:///...`).

   * Click **"Start Setup"**.

   ### **Step 2: Sheet Setup**

   * **Create Sheet:** Click the **`sheets.new`** link to create a new, blank Google Sheet.

     * **Recommendation:** We strongly advise using a new, dedicated Google Account for this system (e.g., `mycompany.safety@gmail.com`). This ensures that if an employee leaves, the app and its data remain with the company, not tied to a personal account. It centralizes ownership and security.

     * Give your Google Sheet a clear name (e.g., "OTG AppSuite Data").

   * **`Visits` Sheet:** Rename the first tab (default "Sheet1") to **`Visits`**. This name must be exact, as the backend script looks for it. This sheet will become your primary database, logging every action, alert, and report.

   * Click cell **A1** in the `Visits` sheet.

   * Go back to the setup tool, click **"Copy Headers"**, and paste. The headers will fill row 1. This step is critical, as the backend script uses these exact header names to find where to write the data. **Do not change their spelling or order.**

   * **(Optional) `Checklists` Sheet:** This step is **only required if** you plan to use the Advanced Reporting features (which you will select in Step 3).

     * Click the **`+`** icon at the bottom of the sheet to add a new tab.

     * Rename this new tab to **`Checklists`** (must be exact).

     * Set cell **A1** to `Company Name`, **B1** to `Question 1`, **C1** to `Question 2`, and so on.

     * **Add `(Standard)` Row:** In cell **A2**, you **must** type **`(Standard)`** (including the parentheses). This is the "magic key" the system uses as the default fallback form. If a worker uses a location that doesn't have a specific `Template Name` assigned, the app will load this `(Standard)` form from its cache.

     * In cells `B2`, `C2`, etc., add the questions for your default, standard report form.

   ### **Step** 3: Configure & **Deploy Script**

   * **Enter `Organisation Name`:** (e.g., "ACME Corp"). This will brand your apps. The Monitor App will say "ACME Corp Monitor," and the Worker App will have this name on its home screen.

   * **Enter `Secret Key`:** Create and enter a strong private password. This is the *only* thing that protects your Monitor Dashboard from being viewed by the public. Treat it like a master password. **Do not lose this key.**

   * **Choose App Type:** This is the most important choice, as it is permanent for this deployment.

     * **Simple (Default):** Leaves the box unchecked for a safety-only app. The "Depart" button will simply end the visit. The Google Sheet will only have the simple (19-column) headers for safety logging. This is the best choice for organizations that *only* need safety timers and alerts.

     * **Advanced (Checkbox):** Check this box to enable all visit reporting features. This will:

       * Change the "Copy Headers" button in Step 2 to the full 21-column version (including `Company Name` and `Visit Report Data`).

       * Enable the "Depart & File Report" workflow in the Worker App.

       * Enable the "Gemini API Key" field below.

   * **(Optional) `Gemini API Key`:** If you enabled Advanced Reporting, paste your API key here.

     * **(See Part 4 of this guide for detailed instructions on how to get your key).**

   * **Copy & Paste Script:** Click **"Copy Script"**. This copies the correct script (`simple` or `advanced`) with your keys injected. Go to your Google Sheet (`Extensions > Apps Script`), paste the code into `Code.gs`, and click **Save**.

   * **Deploy:** Click **`Deploy` > `New deployment`**.

     * **Type:** `Web app`

     * **Execute as:** `Me`

     * **Who has access:** `Anyone`

   * Click **`Deploy`**.

   * **Authorize:** Allow the script access to your Google Account. This is a necessary step. You are giving *your own script* permission to: 1) Read and write to this Google Sheet (to log visits) and 2) Send emails on your behalf (to send safety alerts). If you enable Advanced PDF reports, it will also ask for permission to access your Google Drive (to save the PDFs) and Google Docs (to use your template).

   * **Copy URL:** After deployment, **copy the `Web app URL`**.

   * **Set Trigger:** In Apps Script, click **"Triggers"** (alarm clock icon), click **`Add Trigger`**, set it to run **`checkOverdueWorkers`** on a **`Time-driven`** trigger, **`Minutes timer`**, **`Every 5 minutes`**. Click **`Save`**.

   * **IMPORTANT - How to Update Your Script:** If you ever need to edit the `Code.gs` file later (e.g., to add an API key), you **must re-deploy** it. Do *not* click "New deployment" again. Instead, click **`Deploy` > `Manage deployments`**, click the **pencil (Edit)** icon, select **"New version"** from the "Version" dropdown, and click **`Deploy`**. This is the most common point of failure.

   ### **Step** 4: **Configure Apps**

   * **Web App URL:** Paste the **`Web app URL`** you just copied. This is the "brain" that your Worker and Monitor apps will talk to.

   * **Configure:** Set the First Alert Time, Check-in options, and optional Logo URL.

     * **Check-ins:** Be mindful that this feature only works when the app is open and in the foreground on the worker's phone. (See Worker App User Guide, Section 7).

   * **Test:** Click **"Test Connection & Proceed"**. This test makes a live call to your newly deployed script and uses your Secret Key to try and log in. If it succeeds, you know everything is working.

   ### **Step 5: Download**

   * Click the **"Generate & Download App Package (.zip)"** button.

   * This will save the `OTG_AppSuite_Package.zip` file, which contains your two app folders (`WorkerApp` and `MonitorApp`) and a backup of the script you deployed (`01-PASTE_THIS_INTO_APPS_SCRIPT.txt`).

   ### **Step 6: Finish**

   * Download the full documentation for your records. You are now ready to host the apps.

## **Part 2: Host Your Generated Apps using GitHub Pages**

Now you'll upload the apps from the `.zip` file to your own GitHub repository to make them live. This is a free way to host public websites.

### **2.1 Prepare Your Files**

* **Unzip:** Find the **`OTG_AppSuite_Package.zip`** file you downloaded and unzip it.

* **Locate Folders:** Inside, find the **`WorkerApp`** folder and the **`MonitorApp`** folder.

### **2.2 Create a GitHub Account (If you don't have one)**

* Go to <https://github.com/>.

* Click **"Sign up"** and create a free account. Verify your email.

### **2.3 Create a New Repository**

* Log in to GitHub. Click **"+"** > **"New repository"**.

* **Repository name:** Choose a name, e.g., **`my-otg-apps`**.

* Select **"Public"**. This is required for the free GitHub Pages hosting to work.

* Click **"Create repository"**.

### **2.4 Upload App Folders**

* On your new repository page, click **"Add file"** > **"Upload files"**.

* **Important:** Drag the **`WorkerApp`** folder AND the **`MonitorApp`** folder from your unzipped package onto the GitHub upload area.

* Wait for GitHub to process both folders.

* Scroll down and click **"Commit changes"**.

### **2.5 Enable GitHub Pages**

* Go to your repository's **"Settings"** tab.

* Click **"Pages"** in the left sidebar.

* Under "Build and deployment", set "Source" to **"Deploy from a branch"**.

* Under "Branch", select **`main`**, folder **`/root`**, and click **"Save"**.

* **Wait:** GitHub will provide a public URL (e.g., `https://your-username.github.io/my-otg-apps/`). It may take a minute or two to become active. (If you get a 404 error, just wait 5-10 minutes and try again).

### **2.6 Get Your App URLs**

Your apps are now live! These are the two links you will share with your team.

* **Worker App URL:** `[Your GitHub Pages URL]` + `/WorkerApp/`

  * e.g., `https://your-username.github.io/my-otg-apps/WorkerApp/`

* **Monitor App URL:** `[Your GitHub Pages URL]` + `/MonitorApp/`

  * e.g., `https://your-username.github.io/my-otg-apps/MonitorApp/`

Save these two URLs.

## **Part** 3: **Distribute to Your Team**

### **For Your Safety Monitor:**

* Send them the **Monitoring App URL**.

* **IMPORTANT:** On their first visit, they must enter the **`Web app URL`** (from Part 1, Step 3) and the **`Secret Key`** (from Part 1, Step 3). This is a one-time setup that links their dashboard to your specific Google Sheet.

### **For Your Workers:**

* Send them the **Worker App URL**.

* **Instruct them to:**

  1. Open the link on their smartphone (Safari on iPhone, Chrome on Android).

  2. Use their browser's menu to **“Add to Home Screen”** or **"Install app"**. This is critical for the app to work reliably and offline.

  3. After installation, close the browser and open the app from their new home screen icon.

  4. Go to **Settings** (gear icon).

  5. Fill in **all fields** (Name, Phone, Contacts, PINs). The URL will be pre-filled.

  6. (If using Advanced Reporting) Go to the **Main Screen** and add their visit locations, making sure to assign the correct **`Company Name`** and **`Template Name`**.

  7. Tap **"Save Settings"**.

**Setup Complete!** Your system is operational.

## **Part 4: (Optional) How to Get a Gemini API Key**

This step is only required if you enabled "Advanced Reporting" and want to use the AI-powered spelling and grammar correction for your PDF reports.

This process involves three stages:

1. **Get the Key** from Google AI Studio.

2. **Enable the API** in the Google Cloud Console.

3. **Enable** Billing in the Google Cloud Console (this is required by Google, but you won't be charged for this small usage).

### **4.1** Get the API **Key**

1. Go to the Google AI Studio website: [**https://aistudio.google.com/**](https://aistudio.google.com/)

2. Sign in with the **same Google Account** you used for your Google Sheet.

3. On the left-hand menu, click on **"API keys"**.

4. Click the **"Create API key"** button. You may be asked to create a new "Google Cloud project." This is normal. Give it a name (e.g., "My-OTG-App-Project") and click "Create".

5. A new API key (a long string of letters and numbers) will be generated for you. **Copy this key.**

6. Paste this key into the **`Gemini API Key`** field in the **Setup Tool (Step 3)** *before* you copy the script.

   * **If** you already **deployed your script:** You can paste this key directly into your `Code.gs` file (on line 8) and then **re-deploy** your script (see the "How to Update Your Script" note in Step 3).

### **4.2 Enable the API**

Creating the key *does not* automatically turn it on.

1. Go back to the **"API keys"** page in [Google AI Studio](https://aistudio.google.com/).

2. You will see your new key listed. Click on the **Google Cloud project name** written next to it.

3. This will open the Google Cloud Console, which is a much more complex dashboard.

4. In the search bar at the top, type **`Generative`** Language **`API`** and select it from the results.

5. A new page will load. If you see a blue **"ENABLE"** button, click it. (If it says "API ENABLED" or "MANAGE", you are all set).

### **4.3 Enable Billing (Required)**

Google requires a billing account to be linked to the project to use the API, even for free-tier usage.

1. In the Google Cloud Console, click the "hamburger" menu (☰) in the top-left corner.

2. Select **"Billing"**.

3. Look at the project you just created. If it says "This project has no billing account," you must link one.

4. Click **"Link a billing account"** and follow the prompts to create a new billing profile (this usually requires a credit card).

5. You will **not** be charged. The `gemini-2.5-flash-preview-09-2025` model used by the script has a very large free tier, and your monthly report generation will not come close to exceeding it. This is purely for identity verification and to prevent abuse of the service.

## **Part 5: (Optional) Spreadsheet Administrator Guide**

This guide is for the person managing the Google Sheet database *after* deployment. This role involves setting up report templates, running monthly reports, and basic data maintenance.

### **5.1 How to Set Up Monthly Reports (Spreadsheet Tabs)**

This function generates a separate, clean spreadsheet *tab* for each company's monthly data.

1. **Manually Create Tabs:** In your Google Sheet, create two new tabs:

   * One named `Master Report`

   * One named `Reports` (This one is for an older, single-company report function).

2. **Go** to the **`Master Report` tab.**

3. In cell **B1**, type the month you want to report on (e.g., `2025-10`).

4. **Create a Button (One-time setup):**

   * Go to **`Insert` > `Drawing`**.

   * Create a button shape and type "Generate Spreadsheet Reports" on it.

   * Click **"Save and Close"**.

   * Drag the button where you want it. Click it once, click the **three-dot menu** in its corner, and select **"Assign script"**.

   * In the box, type: `generateMasterMonthlyReport`

   * Click **OK**.

5. **Run the Report:** Click the button. The script will run and create new tabs (e.g., `Report - Smith & Co.`) with all the data for that month.

### **5.2** How to Set **Up PDF Reports (Google Doc Mail Merge)**

This function generates professional, multi-page **PDFs** for each company and saves them to a Google Drive folder.

1. **Create your Google Doc Templates:**

   * Create one or more Google Docs to act as your templates.

   * Design them with your logo, text, and placeholder "tags" like `{{CompanyName}}`, `{{TotalVisits}}`, `{{TotalHours}}`, `{{ChecklistTable}}`, and `{{NotesTable}}`.

   * For each template, open it and copy its **ID** from the URL (the long string between `/d/` and `/edit`).

2. **Create your Output Folder:**

   * In Google Drive, create a folder (e.g., "Monthly Reports - Generated").

   * Open the folder and copy its **ID** from the URL (the string after `/folders/`).

3. **Configure the Script:**

   * Go to **`Extensions` > `Apps Script`**.

   * At the top of the `Code.gs` file, paste your IDs into these lines:

     * `var DEFAULT_REPORT_TEMPLATE_ID = "YOUR_DEFAULT_TEMPLATE_ID_HERE";`

     * `var PDF_OUTPUT_FOLDER_ID = "YOUR_OUTPUT_FOLDER_ID_HERE";`

   * Save and **Re-deploy** (`Deploy` > `Manage deployments` > `Edit` > `New version`).

4. **Link Templates to Companies:**

   * Go to your `Checklists` sheet.

   * Find an empty column (e.g., `G1`) and type the header: `Report Template ID`

   * In that column, paste the Google Doc Template ID for each company. **You** must add **one for the `(Standard)` row.** This is the fallback template.

5. **Create** the **Button:**

   * Go to the `Master Report` tab.

   * Insert a new drawing (button) and label it "Generate All PDF Reports".

   * Assign it the script name: `generateAllPdfReports`

6. **Run the Report:** Click the button. The script will create all the PDFs in your Google Drive folder.

### **5.3 Setting up Longitudinal (Year-over-Year) Reports**

This feature appends monthly totals to a separate, dedicated spreadsheet for each company, allowing you to build graphs that track trends over time.

**A)** One-Time **Setup per Company:**

1. Go to your `Checklists` sheet.

2. Find an empty column (e.g., `H1`) and type the header: `Longitudinal Report Sheet ID`

3. Go to **`Extensions` > `Apps Script`**.

4. From the function dropdown at the top, select `createLongitudinalWorkbook`.

5. Click **`Run`**.

6. A prompt will appear. Enter the *exact* `Company Name` (e.g., `Smith & Co.`) and click OK.

7. The script will create a new Google Sheet file ("Longitudinal Report - Smith &Co."), create all the correct headers, and **automatically paste the new Sheet's ID** into the `Longitudinal Report Sheet ID` column for you.

8. Repeat this process for every company you want to track.

**B)** Running **the Monthly Append:**

1. Go to the `Master Report` tab.

2. Insert a new drawing (button) and label it "Append Longitudinal Data".

3. Assign it the script name: `runAllLongitudinalReports`

4. Click **OK**.

5. Now, when you have a month in cell `B1` and click this button, the script will open every linked workbook and append that month's summary as a new row.

### **5.4 Database Maintenance (Trimming the `Visits` Sheet)**

Your `Visits` sheet will grow over time. To keep the sheet fast and responsive, you should periodically archive old data.

**Warning:** This action is permanent. Always make a backup first (`File` > Make` a copy`).

1. Go to the **`Visits`** sheet.

2. Click the filter icon on **Column A (`Date`)** and sort it A→Z (oldest to newest).

3. Select the rows you want to archive. For example, click on the row number for "2" and drag down to select all rows from last year. **CRITICAL: Do NOT delete Row 1 (the headers).**

4. **Right-click** on the highlighted row numbers.

5. Select **`Delete rows [2 - XX]`**.

## **Part 6: Troubleshooting Guide (For Admins)**

Here are solutions to the most common problems.

* **Problem:** The "Start Setup" button on the setup tool website doesn't work.

  * **Cause:** The tool can't load its template files.

  * **Solution:** Make sure you are running `setup.html` from the **live** GitHub **Pages URL** (e.g., `https://...github.io/...`), not by double-clicking the file from your computer (e.g., `file:///...`).

* **Problem:** My Worker App won't install / The "Travelling" location is missing / The "Settings" button doesn't work.

  * **Cause (Advanced App):** A JavaScript error on load, usually because the `(Standard)` checklist is missing from your `Checklists` sheet. When the app starts, it immediately tries to fetch the `(Standard)` checklist. If it can't find that row, the app's startup script fails, and none of the buttons or features will work.

  * **Solution (Advanced App):**

    1. Go to your `Checklists` sheet.

    2. Make sure you have created a row with the `Company Name` **`(Standard)`** (with parentheses) and added at least one question to it.

    3. On the worker's phone, go to **Settings** and click **"Clear** Cached Forms & **Data"**.

    4. Restart the app.

  * **Cause (Simple App):** The `setup.html` tool may have failed to inject the `%%GOOGLE_SHEET_URL%%` during creation.

  * **Solution (Simple App):** Re-run the setup tool, ensuring you complete Step 4 and get a "Success" message before downloading the .zip package.

* **Problem:** My Custom Form shows `"#Header"` or `"%Note"` with a checkbox.

  * **Cause:** This means the backend script (`Code.gs`) is running an old version of the code *before* we added this feature, or you have a typo in your `Checklists` sheet.

  * **Solution:**

    1. First, check your `Checklists` sheet. Make sure you add a **space** after the `#` or `%` (e.g., `"#` Section` 1"`, `"% Notes"`).

    2. If that is correct, re-deploy your script. Go to `Extensions > Apps Script`, click `Deploy > Manage deployments`, edit your deployment, select **"New version"**, and click **`Deploy`**.

    3. After fixing, clear the cache on the Worker App (see above).

* **Problem:** The Monitor App shows "Cannot Connect" or a 404 error.

  * **Cause:** The Google Sheet URL or Secret Key is wrong.

  * **Solution:**

    1. Re-copy the **`Web`** app **`URL`** (not the Deployment ID) from `Manage deployments` in your Apps Script.

    2. Go to the Monitor App, click the **Reset/Gear icon** in the header.

    3. Re-paste the correct URL and re-type your Secret Key.

* **Problem:** My `generate...Report` script fails or reports are blank.

  * **Cause:** Mismatch in names or date format.

  * **Solution 1:** Make sure the month in the `Master Report` tab (cell `B1`) is in `YYYY-MM` format (e.g., `2025-10`).

  * **Solution 2:** Make sure the `Company Name` in the `Visits` sheet (entered by workers) *exactly* matches the `Company Name` in your `Checklists` sheet (it is case-sensitive).

* **Problem:** My AI Note Correction isn't working (notes are uncorrected).

  * **Cause:** The API Key, Billing, or API is not set up correctly.

  * **Solution:**

    1. Go to **`Extensions` > `Apps Script`** and open your **`Code.gs`** file.

    2. Make sure you have pasted your **Gemini API Key** correctly into the `GEMINI_API_KEY` variable at the top.

    3. **In Google AI Studio**, find your API key and click the link to its **"Google Cloud project"**.

    4. In the Cloud Project, use the search bar to find **"Generative Language API"** and make sure it is **ENABLED**.

    5. In the Cloud Project, go to the **"Billing"** section and make sure the project is **linked to an active billing account**.

    6. After all this, you **must re-deploy** your Apps Script (see Step 3 of this guide).

* **Problem:** My `generateAllPdfReports` button gives a "No item with the given ID" error.

  * **Cause:** A Google Doc ID or Folder ID is wrong, or the script doesn't have permission.

  * **Solution:**

    1. **Re-Authorize:** Run the `generateAllPdfReports` function *manually* from the script editor once to grant it Drive/Docs permissions (see Step 3).

    2. **Check IDs:** Double-check the `PDF_OUTPUT_FOLDER_ID` and `DEFAULT_REPORT_TEMPLATE_ID` in your script (lines 13-16).

    3. **Check `Checklists`:** Make sure your `Report Template ID` column header is spelled *exactly* right and that the IDs in that column are for **Google Docs**, not Google Folders or Spreadsheets.
