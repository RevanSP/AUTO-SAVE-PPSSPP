# 🕹️ PPSSPP Save Auto Git Push

Automatically back up your PPSSPP save files (`.bin`, `.png`, `.sfo`) to a remote Git repository whenever they change. This script monitors your PPSSPP save data folder and automatically commits and pushes save file changes to your Git repository with intelligent batching and retry mechanisms.

---

## ✨ Features

* 🔄 **Automatic Detection:** Watches for changes and additions to `.bin`, `.png`, and `.sfo` files in your PPSSPP save data folder
* 📝 **Smart Filtering:** Only processes save files within game folders (ULUS****, ULES****, ULJM****, etc.)
* ⏱️ **Intelligent Batching:** Groups multiple rapid changes into single commits using a 5-second debounce delay
* 🔄 **Retry Mechanism:** Automatically retries failed pushes up to 3 times with 2-second delays
* 🔒 **Git Lock Handling:** Automatically removes Git lock files and performs lightweight cleanup
* 🌐 **Internet Check:** Verifies internet connection before starting
* 📁 **File Validation:** Ensures files exist and are not locked before processing
* 🛑 **Graceful Shutdown:** On exit (`Ctrl+C`), attempts to push any pending changes
* 🚀 **Force Commit:** Uses `--allow-empty` to ensure commits are created even when Git thinks there are no changes

---

## 📦 Installation

1. **Prerequisites**
   - Bun installed on your system
   - Git configured with proper credentials for your repository
   - PPSSPP save data folder already set up

2. **Install Dependencies**
   ```bash
   bun add chokidar
   ```

3. **Place the Script**
   Save the script file (e.g., `script.js`) in a convenient location (doesn't need to be in the save data folder)

---

## ⚙️ Configuration

1. **Set the Watch Folder**
   Update the path in the script to match your PPSSPP save data folder:
   ```js
   const WATCH_FOLDER = "C:\\Users\\LENOVO\\Documents\\PPSSPP\\PSP\\SAVEDATA";
   ```

2. **Initialize Git Repository**
   If not already done, initialize a Git repo in your save data folder:
   ```bash
   cd "C:\Users\LENOVO\Documents\PPSSPP\PSP\SAVEDATA"
   git init
   git remote add origin https://github.com/yourusername/your-repo.git
   git branch -M main
   git add .
   git commit -m "Initial PPSSPP save upload"
   git push -u origin main
   ```

3. **Configure Git Identity**
   Set your Git username and email if not already configured:
   ```bash
   git config --global user.name "Your Name"
   git config --global user.email "your.email@example.com"
   ```

---

## ▶️ Running the Script

Start the watcher using Bun:
```bash
bun script.js
```

### Example Output:

**On Startup:**
```
🌐 Internet connected. Proceeding to start the script...
📁 Initializing file watcher for PPSSPP saves...
✅ Monitoring PPSSPP saves: C:\Users\LENOVO\Documents\PPSSPP\PSP\SAVEDATA
Starting PPSSPP save auto-push service.
```

**When Files Change:**
```
📝 Queuing PPSSPP save file: ULUS12345/SAVE.BIN
⏰ Debounce period ended, processing queued files...
🔄 Processing 1 file(s): ULUS12345/SAVE.BIN
🧹 Performing lightweight Git cleanup...
✅ Git updated from remote
➕ Adding: ULUS12345/SAVE.BIN
📝 Force committing with message: "Update PPSSPP saves: ULUS12345/SAVE.BIN - 2025-07-13T10:30:00.000Z"
🚀 Pushing to origin/main...
✅ Push complete for: ULUS12345/SAVE.BIN
```

To stop the script, press `Ctrl+C`. Any pending changes will be processed before shutdown.

---

## 🔍 How It Works

1. **Internet Connectivity Check**
   The script first verifies internet connection by attempting to reach Google. If offline, it exits immediately.

2. **File System Monitoring**
   Uses [`chokidar`](https://www.npmjs.com/package/chokidar) with polling mode to reliably detect changes to PPSSPP save files. The watcher:
   - Monitors the entire SAVEDATA folder recursively
   - Only processes files within game folders (ULUS****, ULES****, ULJM****, etc.)
   - Filters for valid save file extensions (`.bin`, `.png`, `.sfo`)
   - Ignores hidden files, Git files, and temporary files
   - Uses a 5-second stability threshold to ensure files are fully written
   - Polls every 3 seconds for changes

3. **Change Queue System**
   When changes are detected:
   - Files are added to a queue (`changeQueue`) with their relative paths
   - A 5-second debounce timer starts/resets
   - When the timer expires, all queued files are processed together

4. **Git Operations**
   For each batch of files:
   - Performs lightweight Git cleanup (removes lock files, pulls latest changes)
   - Validates each file exists and waits for file locks to release
   - Adds files to Git staging area with full folder structure
   - Creates a commit with timestamp and file list
   - Pushes to `origin/main`

5. **Error Handling & Retries**
   - Failed operations are retried up to 3 times with 2-second delays
   - Git lock files are automatically removed
   - Missing files are skipped with warnings
   - Detailed error logging for troubleshooting

6. **Graceful Shutdown**
   On `Ctrl+C`, the script processes any remaining queued files before exiting.

---

## 🎮 PPSSPP Save File Structure

PPSSPP saves are organized in a specific folder structure:
```
SAVEDATA/
├── ULUS12345/          # Game ID folder
│   ├── SAVE.BIN        # Save data file
│   ├── ICON0.PNG       # Save icon
│   └── PARAM.SFO       # Save parameters
├── ULES01234/          # Another game
│   ├── SAVE001.BIN
│   ├── SAVE002.BIN
│   └── PARAM.SFO
```

The script automatically handles this structure and monitors all game folders.

---

## 🛠 Customization Options

### Timing Configuration
```js
const MAX_RETRIES = 3;           // Number of retry attempts
const RETRY_DELAY = 2000;        // Delay between retries (ms)
const DEBOUNCE_DELAY = 5000;     // Batching delay (ms)
```

### File Type Filtering
Change the valid extensions in `queueFile()`:
```js
const validExtensions = ['.bin', '.png', '.sfo'];
// Add other extensions if needed: '.dat', '.save', etc.
```

### Game Folder Pattern
Modify the regex pattern for game folders:
```js
const isInGameFolder = pathParts.length >= 2 && /^UL[A-Z]{2}\d{5}[A-Z0-9]*$/i.test(pathParts[0]);
// Current pattern matches: ULUS12345, ULES01234, ULJM05678, etc.
```

### Commit Message Format
Modify the commit message in `gitPush()`:
```js
const commitMessage = `Update PPSSPP saves: ${existingFiles.join(", ")} - ${localTimestamp}`;
```

### Watcher Options
Adjust the `chokidar` configuration:
```js
const watcher = chokidar.watch(WATCH_FOLDER, {
  persistent: true,
  usePolling: true,
  interval: 3000,                    // Polling interval
  ignoreInitial: true,
  ignored: ["**/.git/**", "**/.*", "**/tmp_*"],
  awaitWriteFinish: {
    stabilityThreshold: 5000,        // Wait time for file stability
    pollInterval: 500,
  },
});
```

---

## ❗ Troubleshooting

### Common Issues

**"No internet connection detected"**
- Ensure you have an active internet connection
- Check if firewall is blocking the script
- Try running as administrator

**"Git lock file removed" messages**
- This is normal - the script automatically handles Git lock files
- If persistent, ensure no other Git operations are running

**"File not found, skipping"**
- PPSSPP might be using temporary files during save operations
- The script waits for file stability before processing

**"Skipping file: not in game folder or invalid extension"**
- This is normal - the script only processes files within game folders (ULUS****, etc.)
- Ensure your save files are in the correct game folder structure

**Authentication errors**
- Configure Git credentials: `git config --global credential.helper store`
- Or use SSH keys for authentication
- Ensure your repository remote URL is correct

**Permission denied errors**
- Run the script as administrator
- Check folder permissions for the PPSSPP save data directory

### Debug Information
The script provides detailed logging for troubleshooting:
- File queue operations with relative paths
- Git command execution
- Error messages with context
- Retry attempts and status

---

## 📊 Performance Notes

- **Polling Mode:** Uses file system polling for reliability across different systems
- **Debounce Batching:** Groups rapid changes to avoid excessive commits
- **File Lock Detection:** Waits for files to be fully written before processing
- **Lightweight Operations:** Minimal Git operations to reduce overhead
- **Recursive Monitoring:** Watches entire folder structure for new games/saves

---

## 🎯 Supported Game Regions

The script automatically detects and processes saves from all PSP game regions:
- **ULUS** - North America
- **ULES** - Europe
- **ULJM** - Japan
- **ULKS** - Korea
- **ULAS** - Asia
- **ULJS** - Japan (alternate)

---

## 📝 License

This project is free and open-source. You're welcome to use, modify, and distribute it as needed.