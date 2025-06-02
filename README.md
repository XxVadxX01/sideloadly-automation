Sideloadly IPA Automation Guide
MacOS

---

## **Overview**

This guide explains how to automate sideloading an IPA app onto your iPhone using Sideloadly on macOS.

Automation includes:

- Installing required tools (Homebrew, GitHub CLI, cliclick)
- Downloading the latest IPA from GitHub releases
- Opening Sideloadly, selecting the IPA file automatically
- Clicking the **Start** button programmatically
- (Optional) Quitting Sideloadly after completion

---

## **Prerequisites**

- macOS device
- Sideloadly installed
- Internet connection
- GitHub account

---

## **Step 1: Install Homebrew & Required Tools**

Run this command in Terminal to install Homebrew (if missing):

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Install GitHub CLI and cliclick:

```
brew install gh cliclick
```

---

## **Step 2: AppleScript for Automation**

Save this AppleScript as open_sideloadly.scpt:

```
on run argv
	set ipaPath to item 1 of argv
	set ipaString to ipaPath as string

	tell application "Sideloadly"
		activate
	end tell

	delay 2

	tell application "System Events"
		tell process "Sideloadly"
			click button 1 of window 1
		end tell
	end tell

	tell application "System Events"
		repeat until exists window 1 of process "Finder"
			delay 0.5
		end repeat

		delay 0.5

		tell process "Finder"
			set frontmost to true
			tell window 1
				keystroke "G" using {command down, shift down}
				delay 0.5
			end tell

			keystroke ipaString
			delay 0.5
			keystroke return
			delay 0.5
			keystroke return
		end tell
	end tell

	delay 3

	tell application "Sideloadly" to activate
	delay 1

	do shell script "/usr/local/bin/cliclick m:858,501"
	delay 0.3
	do shell script "/usr/local/bin/cliclick c:."
end run
```

---

## **Step 3: Shell Script to Run AppleScript**

Create sideload_eevee.sh:

```
#!/bin/bash

IPA_PATH="/full/path/to/your/app.ipa"  # Update this path

osascript ~/Documents/SideloadAutomation/open_sideloadly.scpt "$IPA_PATH"
```

Make executable:

```
chmod +x sideload_eevee.sh
```

---

## **Step 4: Running the Automation**

Run the shell script in Terminal:

```
./sideload_eevee.sh
```

This will:

- Launch Sideloadly
- Select your IPA file automatically
- Click the Start button to sideload

---

## **Optional: Create a macOS Shortcut or Automator App**

- Use **Shortcuts** or **Automator** to run your shell script with one click.
- Make sure you grant Accessibility permissions for the app you run the scripts from.

---

## **Notes & Troubleshooting**

- Ensure **Accessibility permissions** are enabled:
    
    System Settings > Privacy & Security > Accessibility
    
- cliclick must be installed and accessible (adjust path if on M1 Macs)
- Adjust the click coordinates in AppleScript if UI layout changes
- The IPA path must be absolute and correct
- You may need to login to GitHub CLI if using GitHub API for IPA downloads

---

## **Optional Enhancements**

- Add auto-quit after sideload completes
- Add notifications or logging
- Automate GitHub IPA download inside the shell script
- Detect device connection and trigger sideload automatically

---

# **Summary**

This setup automates sideloading on Mac from start to finish, reducing manual steps and increasing reliability.

---

## **💡 Optional Enhancements - Advanced Extras & Code Additions**

---

### **🔄 Auto-Quit Sideloadly After “Done”**

Add this to the end of your AppleScript (after cliclick) to quit Sideloadly once sideloading completes:

```
-- Wait until "Done." appears in Sideloadly
set maxWait to 300
set waitTime to 0

repeat
	tell application "System Events"
		tell process "Sideloadly"
			try
				set statusText to value of static text 1 of window 1
				if statusText contains "Done." then exit repeat
			end try
		end tell
	end tell

	delay 3
	set waitTime to waitTime + 3
	if waitTime ≥ maxWait then
		display dialog "Timeout — Sideloadly may have failed." buttons {"OK"}
		return
	end if
end repeat

delay 2
tell application "Sideloadly" to quit
```

---

### **📦 Automatically Download IPA From GitHub (Shell Script)**

Add this to your shell script (sideload_eevee.sh) before calling AppleScript:

```
REPO="whoeevee/EeveeSpotify"
DOWNLOAD_DIR="$HOME/Downloads/Sideload"
mkdir -p "$DOWNLOAD_DIR"

gh release download -R "$REPO" --pattern "*.ipa" -D "$DOWNLOAD_DIR"
IPA_PATH=$(ls -t "$DOWNLOAD_DIR"/*.ipa | head -n 1)
```

Now the script will auto-download the latest IPA before sideloading.

---

### **🔔 macOS Notification After Install**

Add this to the shell script **after** the AppleScript finishes:

```
osascript -e 'display notification "Sideload complete!" with title "Sideloadly Automation"'
```

---

### **⚙️ Automatically Detect if iPhone is Plugged In (Bonus)**

To detect if an iPhone is plugged in before sideloading:

```
if system_profiler SPUSBDataType | grep -q "iPhone"; then
	echo "iPhone detected."
else
	echo "No iPhone found. Plug in your device and try again."
	exit 1
fi
```

You can run this check before launching Sideloadly.

---

### **🧼 Auto-clean old IPA files (optional)**

To delete previous IPA files before downloading:

```
rm -f "$DOWNLOAD_DIR"/*.ipa
```

Put this line right before gh release download.

---

## **🧠 Tip: Combine Everything**

You can combine all extras into a single, polished shell script that:

- Detects your device
- Cleans old IPAs
- Downloads new ones
- Sideloads via AppleScript
- Quits Sideloadly
- Shows a desktop notification

---

**Power Script.** This one script does **everything**:

✅ Detects if iPhone is connected

✅ Clears old IPA files

✅ Downloads the latest IPA from GitHub

✅ Sideloads it via AppleScript

✅ Waits until install is complete

✅ Quits Sideloadly

✅ Sends a macOS notification

---

### **🚀 Power Script: Fully Automated IPA Sideloading**

```
#!/bin/bash

# === CONFIG ===
REPO="whoeevee/EeveeSpotify"
DOWNLOAD_DIR="$HOME/Downloads/Sideload"
SCRIPT_PATH="$HOME/Documents/SideloadAutomation/open_sideloadly.scpt"

# === Check for iPhone ===
if system_profiler SPUSBDataType | grep -q "iPhone"; then
	echo "✅ iPhone detected."
else
	echo "❌ No iPhone found. Please plug in your device."
	exit 1
fi

# === Ensure Required Tools ===
echo "🔧 Checking Homebrew, gh, and cliclick..."

if ! command -v brew &> /dev/null; then
	echo "Installing Homebrew..."
	/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
fi

brew install gh cliclick

# === GitHub Login ===
if ! gh auth status &>/dev/null; then
	echo "🔑 Logging into GitHub CLI..."
	gh auth login --web
fi

# === Clean Previous IPA Files ===
echo "🧼 Cleaning old IPA files..."
mkdir -p "$DOWNLOAD_DIR"
rm -f "$DOWNLOAD_DIR"/*.ipa

# === Download Latest IPA ===
echo "⬇️ Downloading latest IPA from $REPO..."
gh release download -R "$REPO" --pattern "*.ipa" -D "$DOWNLOAD_DIR"

IPA_PATH=$(ls -t "$DOWNLOAD_DIR"/*.ipa | head -n1)

if [ ! -f "$IPA_PATH" ]; then
	echo "❌ No IPA file found after download."
	exit 1
fi

echo "📦 Latest IPA: $IPA_PATH"

# === Run AppleScript to Sideload ===
echo "🚀 Launching Sideloadly for installation..."

osascript "$SCRIPT_PATH" "$IPA_PATH"

# === Notify After Sideload ===
osascript -e 'display notification "Sideloading complete!" with title "Sideloadly Automation"'
```

---

### **📥 How to Use**

1. Copy this script into a file:
    
    nano ~/sideload_power.sh
    
2. Make it executable:
    
    chmod +x ~/sideload_power.sh
    
3. Run it anytime:
    
    ~/sideload_power.sh
    

---

This script is fully automatic. Just double-click your Automator app or run it via Terminal and you’re done.
