# On-The-Go App Suite – Complete Technical & Functional Specification
**Version 2025 DRAFT – FULL SYSTEM**
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

# On-The-Go Lone Worker Safety System  
Complete, Super-Simple, Step-by-Step Guide for a Normal Office Manager  
(No coding skills needed – everything is copy-and-paste or double-click)

This guide is written for a busy office manager with 10 staff who just wants the system to work perfectly and safely.

### What you will have when you finish (in plain English)

1. A phone app your staff install on their own phones  
2. One Google Sheet that does all the clever stuff  
3. A big monitoring screen in the office that flashes and makes a loud alarm the moment anyone is in trouble  
4. Perfect monthly reports that write themselves

Total time: 45–60 minutes the first time (then 5 minutes for new staff).

### Part 1 – Get your two free keys (5 minutes)

#### 1A – Free SMS key (so real text messages arrive)
1. Open Chrome or Edge and go to: https://textbelt.com  
2. Scroll down a little → click the big blue button “Get a free Textbelt key”  
3. Type your work email address → press Enter  
4. Open your email → you will get a message that contains a key like `textbelt987654321`  
5. Copy that whole key (highlight it → right-click → Copy)  
6. Paste it into a Word document or Notepad and label it “SMS key”

Important: The free key only sends ONE text message per phone number ever.  
If you want proper alerts, come back to the same website later and click “Buy quota”. $5 gives you 500 texts (enough for years).

#### 1B – Free AI key (makes notes look perfectly typed)
1. Go to: https://aistudio.google.com/app/apikey  
2. Click the blue button “Create API key”  
3. A long code appears starting with AIzaSy…  
4. Copy the whole thing → paste it into the same Word document and label it “AI key”

### Part 2 – Make three empty folders in Google Drive and get their secret codes (3 minutes)

1. Go to Google Drive → https://drive.google.com  
2. On the left, click the big blue “+ New” button → “Folder”  
3. Name the first folder OTG Photos → click Create  
4. Make two more folders called:  
   - OTG PDF Reports  
   - OTG Templates (only if you have your own report layout – you can skip this)

5. Double-click the OTG Photos folder to open it  
6. Look at the web address at the top of the screen. It will say something like:  
   https://drive.google.com/drive/folders/1a2b3c4d5e6f7g8h9i0j  
7. Copy ONLY the long string of letters and numbers after “folders/”  
   → in the example above that is 1a2b3c4d5e6f7g8h9i0j  
8. Paste it into your Word document and write “Photos folder code” next to it

Do the same for the OTG PDF Reports folder.

### Part 3 – Run the magic Setup tool (5 minutes)

1. Find the file called Setup.html (it came in the zip you were given)  
2. Double-click it – it opens in your web browser  
3. Fill in every box exactly like this:

| Box you see on screen               | What to type or paste                                      |
|-------------------------------------|------------------------------------------------------------|
| Organisation Name                   | Your real name (e.g. Sunnyvale Community Care)             |
| Logo URL                            | Leave blank (or paste a picture link if you have one)      |
| Secret Key                          | Make up anything long, e.g. sunnyvale2025secret            |
| First alert delay (minutes)         | 15                                                         |
| Escalation interval (minutes)       | 5                                                          |
| Gemini AI Key                       | Paste the AIzaSy… key (or leave blank)                     |
| Textbelt key                        | Paste the textbelt987… key                                 |
| Photos folder ID                    | Paste the long code from OTG Photos folder                 |
| PDF Reports folder ID               | Paste the long code from OTG PDF Reports folder            |
| Default Report Template ID          | Leave blank for now                                        |

4. Click the big blue button “Generate All Files”  
5. A zip file downloads → double-click it → drag everything to your Desktop

You now have all the pieces.

### Part 4 – Create the Google Sheet and the “brain” (10 minutes)

1. On your Desktop, find Master_Spreadsheet_Template.xlsx → double-click it  
2. Google Sheets opens → click “Make a copy” → name it “Sunnyvale Lone Worker System 2025” → click OK

3. On your Desktop, find the file OTG_Backend_Script.gs → right-click → Open with → Notepad  
4. In Notepad, press Ctrl+A (select all) then Ctrl+C (copy)  
5. Open a new browser tab and go to https://script.google.com  
6. Click the big “+ New project” button  
7. Delete the word “myFunction” that is already there  
8. Click anywhere in the white area and press Ctrl+V (paste)  
9. At the top, click the floppy-disk save icon → name it “OTG Brain”

10. Click the blue button “Deploy” → “New deployment”  
11. Click “Select type” → click “Web app”  
12. Fill in:  
    - Execute as: Me (your name)  
    - Who has access: Anyone  
13. Click “Deploy” → click “Authorize access” → choose your Google account  
14. A long web address appears ending in /exec → click the copy button

Paste that address into your Word document and label it “Web App URL” – you need it twice more.

### Part 5 – Mobile app for your staff (5 minutes per phone)

1. On your computer, open a browser and go to https://github.com  
2. Click the green “Sign up” button in the top right  
3. Type your email, make a password, choose a username (e.g. sunnyvalecare2025) → click Continue  
4. Skip the questions and click “Create account”  
5. Check your email and click the link GitHub sent you

6. Back on GitHub, click your picture in the top right → Settings  
7. On the left, click “Repositories” → click the green “New” button  
8. Name it sunnyvale-apps → check “Add a README file” → click “Create repository”

9. Now on the new page, click the green “Code” button → “Create new file”  
10. Name the file index.html → paste the entire content of your generated worker_app_advanced.html (or simple, if you chose simple mode) into the box  
11. Scroll down → click “Commit new file”  

12. Repeat step 9–11 for sw.js, manifest.json, and any icon.png if you added a logo

13. Click the gear icon next to “About” → check “Use your GitHub Pages website” → click Save  
14. Wait 30 seconds → the URL appears under GitHub Pages → e.g. https://sunnyvalecare2025.github.io/sunnyvale-apps

15. On each worker’s phone:  
   - Open Chrome (Android) or Safari (iPhone)  
   - Go to that URL (e.g. https://sunnyvalecare2025.github.io/sunnyvale-apps)  
   - Tap the menu button → “Add to Home Screen” (Android) or Share button → “Add to Home Screen” (iPhone)  
   - An app icon appears on their phone – tap it to open

Note: GitHub is free and safe for this – no one can see your keys because they are inside the app files, but the app doesn’t expose them.

(How the phone app actually works – copy this and send to your team)
Subject: Your new safety app – please read once (takes 1 minute)
Hi team,
The app is deliberately very simple. There are only three things you ever do:
1. Quick Notes (most of the time – no timer, no check-in)
For vehicle checks, incident reports, messages, etc.
→ Open the app
→ Tap the form you want (e.g. “Daily Vehicle Check”)
→ Fill it → tap Submit
Done. Nothing else.
2. When you are going somewhere and want the office to watch over you
(This is the proper lone-worker protection – only when you need it)

Open the app
Tap the client or location you are going to (e.g. “Mrs Margaret Jones” or “Travelling to London”)
→ It turns blue to show it is selected
Slide the orange timer to show how long you expect to be there
(e.g. 45 minutes, 2 hours – whatever is realistic)
Press and hold the big green START button for 2 seconds
→ The office screen now shows you as “ON SITE” or “TRAVELLING” and you are fully protected

3. When you leave the location (or finish travelling)

Open the app again
You will see a big red DEPART button
Tap Depart → fill in the short departure questions (if any) → tap Submit
→ You disappear from the office screen

That’s it. Nothing else.
The PIN – only used in an emergency

The first time you open the app, it asks you to set your own 4-digit safety PIN (you choose any numbers you’ll remember).
You only ever type this PIN if the office screen has already gone yellow or red and they phone you to check you’re safe.
When everything is OK, open the app → type your normal 4-digit PIN → the alarm stops.

Secret emergency code (tell everyone in person)
If you are in danger and someone is watching you, type 9999 instead of your normal PIN.
This silently triggers the highest-level alarm without the attacker knowing.
30-second install (do this now)

On your phone open Chrome (Android) or Safari (iPhone)
Go to: https://your-org-name.github.io/your-apps/
Tap the three dots menu → “Install app” (Android) or Share □ → “Add to Home Screen” (iPhone)
Tap Install / Add
An app icon appears – open it once to set your safety PIN

That’s everything.
No daily passwords. No complicated steps. Just Quick Notes or proper Start → Depart when you need protection.
Thank you – you are now properly looked after.
(Print this page and put it on the staff noticeboard)

### Part 6 – The big monitoring screen that flashes and beeps (10 minutes)

#### The simple way – just double-click every morning
1. On your Desktop, find OTG_Monitor_2025.html → double-click it  
2. Two boxes appear:  
   - Paste the long /exec Web App URL  
   - Paste the secret key you made up earlier  
3. Click “Start Monitoring”  
4. The screen goes green and starts checking every 15 seconds  
5. Leave this window open all day (or press F11 for full-screen)

#### The better way – make it a real program that starts automatically (for Windows)
1. Go to https://github.com/tauri-apps/tauri/releases  
2. Scroll down to the latest version → click the file called tauri-cli.msi (for Windows) → download and double-click to install  
3. Create a new folder on the Desktop called “OTG Monitor App”  
4. Put only one file in it: OTG_Monitor_2025.html  
5. Open the Command Prompt (search for cmd in the Start menu)  
6. Type cd Desktop\OTG Monitor App → press Enter  
7. Type npx create-tauri-app → press Enter  
8. Choose “Use existing project” → follow the questions (say yes to all)  
9. When it finishes, type npm run tauri build → press Enter  
10. Wait 1 minute → a folder appears with OTG Monitor.exe inside  
11. Double-click the .exe → enter the Web App URL and key → it remembers forever

**Make it start automatically**  
1. Right-click the .exe → Copy  
2. Press Windows key + R → type shell:startup → Enter  
3. Right-click in the folder → New → Shortcut → paste the .exe → Finish  

Now the monitoring screen opens every time the office computer starts.

For Mac, the steps are similar but use Terminal (ask me if you need Mac steps).

### Part 7 – Adding locations and forms to the app (the fun bit)

Open your Google Sheet → click the Checklists tab.

This is where you create every form your staff will see.

**Example – Normal client visit**
1. In cell A2 type Sunnyvale Care  
2. In B2 type [HEADING] Morning Visit  
3. In C2 type [TEXT] How is the client?  
4. In D2 type [PHOTO] Front door  

**Example – Quick forms not for visits**
1. In cell A3 type Quick Forms  
2. In B3 type [HEADING] Vehicle Check  
3. In C3 type [YESNO] Tyres OK?  

Your staff will see these as buttons in the app.

### Worker App Quick Guide (print and give to your staff)

How to use the phone app  
1. Open the app (the icon on your home screen) → type your 4-digit PIN  
2. Choose your company name  
3. Choose the location (or Quick Forms)  
4. Fill in the questions → press Check In  
5. When you leave → press Check Out  

Red PANIC button = loud alarm in the office  
Wrong PIN on purpose = silent duress alarm

### Monitor Screen Quick Guide (print and put on the wall)

What the colours mean  
- Flashing RED = panic button → call 999/111 now  
- Flashing PURPLE = duress (danger but not alone) → call police  
- Yellow = someone is late  
- Gray = normal

To stop the loud noise → click Acknowledge  
To completely clear someone (only after calling them) → click Manually Resolve Alert and type their name exactly

You did it!  
Your staff are safer than ever – and you did it all yourself.
