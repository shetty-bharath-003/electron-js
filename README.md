# Electron JS - Session Guide

## Part 1 — What is Electron JS?

Electron is an open-source framework that lets you build **cross-platform desktop applications** using web technologies you already know:

- HTML
- CSS
- JavaScript (Node.js)

**Real apps built with Electron:**

- Visual Studio Code
- Discord
- Slack
- Figma (desktop client)
- Postman

If you can build a website, you can build a desktop app.

---

### How Electron Works

Electron has two core processes that work together:

```
+---------------------------+
|        Electron App       |
|                           |
|  +-------------------+    |
|  |   Main Process    |    |  <-- Node.js (controls the app)
|  |   (main.js)       |    |
|  +-------------------+    |
|           |               |
|  +-------------------+    |
|  | Renderer Process  |    |  <-- Chromium (displays the UI)
|  |  (index.html)     |    |
|  +-------------------+    |
+---------------------------+
```

**Main Process**
- Entry point of the app
- Creates and manages windows
- Has full access to Node.js and OS APIs
- Think of it as the backend

**Renderer Process**
- Runs inside each BrowserWindow
- Displays your HTML/CSS/JS UI
- Think of it as the frontend

---

## Part 2 — Building Your First Electron App

### Prerequisites

Make sure you have Node.js installed.

```bash
node -v
npm -v
```

Download Node.js from: https://nodejs.org

---

### Step 1 — Create a New Project

```bash
mkdir my-electron-app
cd my-electron-app
npm init -y
```

---

### Step 2 — Install Electron

```bash
npm install electron --save-dev
```

---

### Step 3 — Create the Main Process File

Create a file called `main.js`:

```js
const { app, BrowserWindow } = require('electron');
const path = require('path');

function createWindow() {
  const win = new BrowserWindow({
    width: 900,
    height: 650,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js')
    }
  });

  win.loadFile('index.html');
}

app.whenReady().then(() => {
  createWindow();

  // Re-create window on macOS when dock icon is clicked
  app.on('activate', () => {
    if (BrowserWindow.getAllWindows().length === 0) createWindow();
  });
});

// Quit when all windows are closed (except on macOS)
app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') app.quit();
});
```

---

### Step 4 — Create the UI

Create `index.html`:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>My First Electron App</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      margin: 0;
      background: #1e1e2e;
      color: #cdd6f4;
    }
    h1 { font-size: 2rem; }
  </style>
</head>
<body>
  <h1>Hello from Electron</h1>
</body>
</html>
```

---

### Step 5 — Update package.json

Open `package.json` and make these two changes:

```json
{
  "main": "main.js",
  "scripts": {
    "start": "electron ."
  }
}
```

---

### Step 6 — Run the App

```bash
npm start
```

A native desktop window opens with your HTML page inside it. That is your first desktop app.

---

## Part 3 — IPC (Inter-Process Communication)

IPC is how the Main Process and Renderer Process talk to each other.

**Why do you need it?**

The Renderer Process cannot directly access Node.js or OS features for security reasons. IPC is the bridge.

```
Renderer (UI)  <---IPC--->  Main Process (Node.js / OS)
```

There are two IPC patterns:

| Pattern | Use when |
|---------|----------|
| `ipcRenderer.send` + `ipcMain.on` | Fire and forget (one-way) |
| `ipcRenderer.invoke` + `ipcMain.handle` | Request and wait for a response (two-way) |

---

### Setting Up IPC — Three Files

**1. preload.js** — the secure bridge between Renderer and Main

```js
const { contextBridge, ipcRenderer } = require('electron');

contextBridge.exposeInMainWorld('electronAPI', {
  sendMessage: (msg) => ipcRenderer.invoke('send-message', msg)
});
```

**2. index.html** — the UI that triggers the IPC call

```html
<!DOCTYPE html>
<html>
<head>
  <title>IPC Demo</title>
</head>
<body>
  <h2>IPC Demo</h2>
  <button id="btn">Send Message to Main</button>
  <p id="response"></p>

  <script>
    document.getElementById('btn').addEventListener('click', async () => {
      const reply = await window.electronAPI.sendMessage('Hello from Renderer');
      document.getElementById('response').innerText = reply;
    });
  </script>
</body>
</html>
```

**3. main.js** — handles the message and sends a reply

```js
const { app, BrowserWindow, ipcMain } = require('electron');
const path = require('path');

ipcMain.handle('send-message', async (event, message) => {
  console.log('Received:', message);
  return 'Main Process received: ' + message;
});

function createWindow() {
  const win = new BrowserWindow({
    width: 900,
    height: 650,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js')
    }
  });
  win.loadFile('index.html');
}

app.whenReady().then(createWindow);
```

**Run it, click the button, and the reply appears on screen.**

---

## Part 4 — Native OS Integration

Electron gives you access to native OS features through built-in modules.

---

### File System — Read and Write Files

```js
// In main.js
const { ipcMain } = require('electron');
const fs = require('fs');
const path = require('path');

ipcMain.handle('read-file', async () => {
  const filePath = path.join(__dirname, 'data.txt');
  const content = fs.readFileSync(filePath, 'utf-8');
  return content;
});

ipcMain.handle('write-file', async (event, content) => {
  const filePath = path.join(__dirname, 'data.txt');
  fs.writeFileSync(filePath, content, 'utf-8');
  return 'File saved.';
});
```

---

### Native File Open / Save Dialogs

```js
// In main.js
const { ipcMain, dialog } = require('electron');
const fs = require('fs');

ipcMain.handle('open-file-dialog', async () => {
  const result = await dialog.showOpenDialog({
    properties: ['openFile'],
    filters: [{ name: 'Text Files', extensions: ['txt'] }]
  });

  if (!result.canceled) {
    const content = fs.readFileSync(result.filePaths[0], 'utf-8');
    return content;
  }
  return null;
});
```

Expose it in `preload.js`:

```js
contextBridge.exposeInMainWorld('electronAPI', {
  openFile: () => ipcRenderer.invoke('open-file-dialog')
});
```

Use it in your HTML:

```html
<button id="open">Open File</button>
<pre id="content"></pre>

<script>
  document.getElementById('open').addEventListener('click', async () => {
    const text = await window.electronAPI.openFile();
    if (text) document.getElementById('content').innerText = text;
  });
</script>
```

---

### System Tray

```js
// In main.js
const { app, Tray, Menu, nativeImage } = require('electron');
const path = require('path');

let tray;

app.whenReady().then(() => {
  const icon = nativeImage.createFromPath(path.join(__dirname, 'icon.png'));
  tray = new Tray(icon);

  const contextMenu = Menu.buildFromTemplate([
    { label: 'Show App', click: () => { /* show window */ } },
    { label: 'Quit', click: () => app.quit() }
  ]);

  tray.setToolTip('My Electron App');
  tray.setContextMenu(contextMenu);
});
```

---

### Notifications

```js
// In main.js
const { Notification } = require('electron');

function sendNotification() {
  new Notification({
    title: 'Electron App',
    body: 'This is a native desktop notification.'
  }).show();
}
```

---

## Part 5 — Packaging and Distribution

Once your app is ready, you need to package it so users can install it without Node.js.

The most popular tool is **Electron Forge**.

---

### Step 1 — Install Electron Forge

```bash
npm install --save-dev @electron-forge/cli
npx electron-forge import
```

This updates your `package.json` with the necessary build configuration.

---

### Step 2 — Package the App

```bash
npm run package
```

This creates a folder in `out/` with a standalone executable for your current OS.

---

### Step 3 — Create an Installer

```bash
npm run make
```

This creates a distributable installer:

| Platform | Output |
|----------|--------|
| Windows | `.exe` installer via Squirrel |
| macOS | `.dmg` file |
| Linux | `.deb` or `.rpm` package |

---

### Project Structure (Production Ready)

```
my-electron-app/
├── main.js           # Main Process
├── preload.js        # Secure IPC bridge
├── index.html        # UI entry point
├── renderer.js       # Frontend JavaScript
├── assets/
│   └── icon.png
├── package.json
└── out/              # Generated after packaging
```

---

## Quick Reference Card

| Concept | What it does |
|---------|--------------|
| `BrowserWindow` | Creates a native app window |
| `app.whenReady()` | Fires when Electron is ready to create windows |
| `ipcMain.handle` | Main process listens for a request and can return a value |
| `ipcRenderer.invoke` | Renderer sends a request and waits for a reply |
| `contextBridge` | Safely exposes Main Process APIs to the Renderer |
| `dialog` | Native open/save file dialogs |
| `Tray` | Adds your app to the system tray |
| `Notification` | Sends a native OS notification |
| `electron-forge` | Packages and builds your app for distribution |

---

## What to Explore Next

- Auto-updater with `electron-updater`
- SQLite or LevelDB for local data storage
- Multiple windows and window management
- Custom menus and keyboard shortcuts with `Menu` and `globalShortcut`
- Deep linking and protocol handlers
- Using React, Vue, or Svelte as the Renderer UI framework

---

## Resources

- Official Docs: https://www.electronjs.org/docs/latest
- Electron Forge: https://www.electronforge.io
- Electron Fiddle (sandbox): https://www.electronjs.org/fiddle
- Community: https://discord.gg/electron

---

*Session prepared for beginner to intermediate students. Duration: 1 hour.*
