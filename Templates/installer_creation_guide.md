# Optional Guide: Creating a Desktop Installer for the Monitor App

This is an **optional but highly recommended** step for your safety monitor.

This guide will walk you through the process of turning the **Monitoring Dashboard URL** (the website) into a standalone, clickable desktop application (like an `.exe` file on Windows or an `.app` file on Mac).

**Why do this?**

* **Professionalism:** It puts a dedicated "OTG Monitor" icon on the monitor's desktop. This feels like a permanent, professional application, not just another browser tab.

* **Prevents Accidental Closure:** It runs in its own, separate window. This is the biggest advantage. It prevents the monitor from accidentally closing the app along with their other browser tabs, which would stop all safety alerts.

* **Always On:** You can set this desktop app to start automatically any time the user logs in to their computer. This ensures the monitor is always running and ready to receive an alert.

We will use a free, trusted, and very popular tool called `Nativefier` to accomplish this.

## **Part 1: Install the Necessary Tools**

Before you can build the app, you need to install a free program called **Node.js**. This is a very common and safe background technology used by many modern applications. It also includes `npm` (Node Package Manager), a tool we need to install Nativefier.

1. **Go to the Node.js Website:**

   * Open your web browser and go to: <https://nodejs.org/>

2. **Download the Installer:**

   * You will see two buttons. Click the one for **"LTS"** (Long Term Support). This is the version recommended for most users as it is the most stable and reliable.

3. **Run the Installer:**

   * Find the file you just downloaded (usually in your "Downloads" folder) and double-click it.

   * Follow the on-screen prompts. You can safely accept all the default settings. There is no need to change anything; just click "Next," "Agree," "Install," etc., until it says it has finished.

## **Part 2: Open Your Command Line Tool**

This part can look intimidating, but it's just a text-based way to give your computer commands.

* **On Windows (Command Prompt):**

  * Press the **Windows Key** on your keyboard (or click the Start button).

  * Type the letters: `cmd`

  * You will see **"Command Prompt"** appear in the list. Click on it.

  * A black window will open. This is the command line.

* **On macOS (Terminal):**

  * Click the **Spotlight icon** (the magnifying glass in the top-right corner of your screen).

  * Type: `Terminal`

  * You will see the **"Terminal.app"** icon. Click on it.

  * A white or black window will open. This is the command line.

## **Part 3: Install Nativefier**

Now, you'll use `npm` (which you installed with Node.js) to download and install the Nativefier tool.

* In the command-line window you just opened, type (or copy and paste) the following command and press **Enter**:
  `npm install -g nativefier`

* **What is this command doing?** `npm install` tells the Node Package Manager to install something. `-g` means "globally," so you can run this tool from anywhere on your computer. `nativefier` is the name of the tool we want.

* You'll see text scrolling by as it downloads and installs. This may take a minute or two.

* Once it's finished, it will return you to a new line, ready for your next command. (You can safely ignore any "warnings" (`WARN`) it might show).

* You only need to do this once, ever. Nativefier is now permanently installed.

## **Part 4: Build Your Desktop App**

This is the final step where you create the app.

1. **Go to Your Desktop:**

   * It's easiest to save the new app directly to your Desktop. In your command-line window, type the following and press **Enter**:
     `cd Desktop`

   * `cd` stands for "Change Directory." You have just told the command line to move into your Desktop folder.

2. **Run the Nativefier Command:**

   * Get your **Monitoring App URL** from your Deployment Guide (e.g., `https://your-username.github.io/my-otg-apps/MonitorApp/`).

   * Carefully type (or copy and paste) the following command into your command-line window. **Replace the placeholder URL** with your actual Monitor App URL. Be sure to keep the quotation marks.

   `nativefier --name "OTG Monitor" --internal-urls ".*" "https://your-username.github.io/my-otg-apps/MonitorApp/"`

   * **What is this command doing?**

     * `nativefier`: Runs the tool.

     * `--name "OTG Monitor"`: Sets the name of your final application file.

     * `--internal-urls ".*"`: This is an important detail. It tells the app that *any* link it clicks (like the Google Maps link) should also open *inside* the app, rather than opening an unwanted new browser tab.

     * `"https://..."`: The website it will package into an app.

   * Press **Enter**. Nativefier will start working. It will download a lightweight version of a web browser and bundle your web app inside it. This will take a few minutes.

3. **Find Your App:**

   * Look on your computer's **Desktop**.

   * You will find a new folder (e.g., `OTG Monitor-win32-x64` on Windows or `OTG Monitor-darwin-x64` on Mac).

   * Inside this folder is your new application (`OTG Monitor.exe` or `OTG Monitor.app`). You can double-click this to run it!

## **Part 5: Share the App with Your Monitor**

To send this to your safety monitor, you must send them **the entire folder** that Nativefier created, not just the `.exe` file. The application needs all the other files in that folder to run correctly.

1. Find the folder on your Desktop (e.g., `OTG Monitor-win32-x64`).

2. **Right-click** on the folder.

3. Select **"Send to" > "Compressed (zipped) folder"** (on Windows) or **"Compress..."** (on Mac).

4. This will create a new `.zip` file (e.g., `OTG Monitor-win32-x64.zip`).

5. You can now send this single `.zip` file to your monitor via email or a file-sharing service.

**Instructions for Monitor:** Tell them to download the `.zip` file, unzip it (right-click > "Extract All..."), open the new folder, and find and run the `OTG Monitor` application inside. They can then create a shortcut to this file and place it on their desktop for easy access.

## **Part 6: (Recommended) Make the Monitor App Start Automatically**

To ensure the Monitoring App is always running when the monitor is at their computer, you can add it to their "Startup" or "Login Items."

**Benefit:** This is the most reliable way to use the Monitor. The app will launch automatically every time the computer starts, so the monitor can't forget to run it.

### **On Windows (using the Startup Folder):**

1.  First, find your `OTG Monitor.exe` application. It's inside the folder Nativefier created (e.g., `OTG Monitor-win32-x64`).
2.  **Move this folder** somewhere permanent, like your `C:\Program Files` directory, so it isn't accidentally deleted from the Desktop.
3.  Once it's moved, open the folder and **right-click** on the `OTG Monitor.exe` file.
4.  Select **"Create shortcut"**.
5.  On your keyboard, press the **Windows Key + R** at the same time. This will open a small "Run" window.
6.  In the "Run" window, type `shell:startup` and press **Enter**.
7.  A new folder window named "Startup" will open.
8.  **Drag and drop** (or cut and paste) the `OTG Monitor - Shortcut` file you created in step 4 into this "Startup" folder.

That's it! The next time the user logs in to Windows, the monitor app will start automatically.

### **On macOS (using Login Items):**

1.  Move the `OTG Monitor.app` file (and its folder) from your Desktop to your main **`Applications`** folder.
2.  Open **System Settings** (from the Apple menu in the top-left corner). On older macOS versions, this is called **System Preferences**.
3.  Click on **"General"** in the sidebar, then click **"Login Items"**.
4.  You will see a list called "Open at Login". Click the **plus icon (`+`)** below this list.
5.  A Finder window will open. Navigate to your **`Applications`** folder.
6.  Select the `OTG Monitor.app` file and click **"Open"** (or "Add").

The app is now added to the list and will launch automatically when you log in. You can also tick the **"Hide"** checkbox next to it in the list if you want it to start minimized.
