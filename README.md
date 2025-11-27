# On-The-Go App Suite – Complete Technical & Functional Specification
**Version 2025 FINAL – BULLETPROOF – FULL SYSTEM**
(Including the Real-Time Monitoring Dashboard)

For use by any AI (Claude, GPT-4o, Gemini, Grok, etc.) or human to recreate the entire end-to-end lone-worker safety platform from scratch.

## 1. System Overview – The Complete 2025 Architecture
Mobile Apps (iOS/Android)
│
│ HTTPS POST/GET
▼
Google Apps Script Web App (/exec)
│
▼
Google Sheet ("Master Spreadsheet")
│
┌────┴────┐
▼          ▼
Visits     Checklists
sheet       sheet
│          │
▼          ▼
Drive      Drive
Photos     PDF Reports
│
▼
Time-driven triggers (every 5–15 min)
│
▼
Real-Time Monitoring Dashboard (OTG_Monitor_2025.html)
←──── JSONP polling every 15 seconds ──── Google Apps Script

## 2. The Four Deliverables

| # | Component                        | File Type                | Line Count | Purpose                                             |
|---|----------------------------------|--------------------------|------------|-----------------------------------------------------|
| 1 | Mobile App                  | Flutter or React Native  | —          | Field worker interface + panic/duress               |
| 2 Backend                        | Google Apps Script       | 861        | All logic, alerts, reporting, photo storage         |
| 3 **Live Monitoring Dashboard**  | Single HTML file         | ~1,050     | Real-time command & control centre (wall tablet OK) |
| 4 Master Spreadsheet             | Google Sheet             | —          | Central database + configuration                    |

## 3. Live Monitoring Dashboard – Full Specification

**File:** `OTG_Monitor_2025.html` – completely self-contained (no server needed)

### Key Features (must be replicated exactly)
- Real-time JSONP polling every 15 seconds (`?callback=…&token=SECRET_KEY`)
- Full-screen flashing red/purple takeover on panic/duress
- Tone.js unignorable siren (square wave)
- Desktop notifications with click-to-focus
- Battery level with colour + pulse when <15%
- GPS age indicator (yellow "stale" if >30 min old while travelling)
- Pre-alert amber border when overdue but not yet escalated
- Manual resolve with typed worker-name confirmation
- Acknowledge button (writes note without clearing status)
- Session event log (collapsible, max 50 entries)
- 2,500+ row performance warning banner
- LocalStorage setup screen with live connection test
- Works offline until first successful poll

### Visual Status Matrix (must match exactly)

| Alarm Status                  | Card BG       | Border / Flash      | Full-Screen | Sound         | Notification |
|-------------------------------|---------------|---------------------|-------------|---------------|--------------|
| DURESS_CODE_ACTIVATED         | Purple-800    | Purple flashing     | Yes (purple)| Siren         | Yes          |
| EMERGENCY - PANIC BUTTON      | Red-700      | Red flashing        | Yes (red)   | Siren         | Yes          |
| ESCALATION_SENT / EMAIL_3_SENT| Yellow-700    | Yellow              | No          | Double chime  | Yes          |
| EMAIL_1/2_SENT / MISSED_CHECKIN| Yellow-700   | Yellow              | No          | Single chime  | Yes          |
| ON SITE + overdue (pre-alert) | Gray-700      | Amber border        | No          | None          | No           |
| ON SITE (normal)              | Gray-700      | None                | No          | None          | No           |

### Required Placeholders (replace at deployment)
%%ORGANISATION_NAME%%
%%LOGO_URL%%

## 4. Backend – Apps Script (Option A – 861 lines)

Already provided in previous messages.  
Key points:
- Single `handleRequest(e)` pattern (no duplicate functions)
- All logic from old `doPost` merged inside
- Zero duplication, zero missing braces
- Full photo handling, escalation engine, AI proofreading, PDF reports, longitudinal workbooks

## 5. Google Sheet Structure (exact column headers – case sensitive)

**Visits sheet**
Worker Name, Worker Phone Number, Company Name, Location Name, Location Address,
Arrival Time, Actual Departure Time, Anticipated Departure Time,
Alarm Status, Emergency Contact Name, Emergency Contact Email, Emergency Contact Number,
Escalation Contact Name, Escalation Contact Email, Escalation Contact Number,
Last Known GPS, GPS Timestamp, Battery Level, Notes, Visit Report Data,
Photo 1, Photo 2 … Photo 10, Photo URLs

**Checklists sheet**
- Column A = Company Name
- Column B onward = questions using [TAG] system
- Optional columns: Report Template ID, Longitudinal Report ID

**Master Report sheet**
- Cell B1 = YYYY-MM (month to report on)

## 6. Configuration Placeholders (all components)

| Placeholder                        | Used In                    | Example                         |
|------------------------------------|----------------------------|---------------------------------|
| %%SECRET_KEY%%                     | Backend + Mobile           | a9f3k8x2p1                      |
| %%FIRST_ALERT_MINUTES%%            | Backend                    | 15                              |
| %%ESCALATION_MINUTES%%             | Backend                    | 5                               |
| %%GEMINI_API_KEY%%                 | Backend (AI proofreading)  | AIzaSy...                        |
| %%ORGANISATION_NAME%%              | Backend + Monitor HTML     | Rural Care Services             |
| %%LOGO_URL%%                       | Monitor HTML               | https://i.imgur.com/abc123.png  |
| YOUR_PHOTOS_FOLDER_ID_HERE         | Backend                    | 1AbCdefGHIjklMNO                |
| YOUR_PDF_OUTPUT_FOLDER_ID_HERE     | Backend                    | 1XyZ9876543210                  |
| YOUR_DEFAULT_DOC_TEMPLATE_ID_HERE  | Backend                    | 1DocTemplateABC123              |

## 7. Deployment Checklist – Full System in <30 Minutes

1. Create Google Sheet with exact columns above columns
2. Paste the 861-line Apps Script (Option A)
3. Replace all placeholders and Drive folder/template IDs
4. Deploy → New deployment → Web app → Execute as: Me → Access: Anyone
5. Copy the /exec URL
6. Open OTG_Monitor_2025.html → paste URL + secret key → Start Monitoring
7. (Optional) Wrap monitor HTML in Electron/Tauri for native desktop app
8. Set triggers:
   - `checkOverdueWorkers` → every 5 minutes
   - `archiveOldData` → weekly
9. Done – full military-grade lone-worker safety system is live

November 2025
