
# Electron JS

## Part 1 — What is Electron?

Electron is an open-source framework that allows you to build cross-platform desktop applications using:

* HTML
* CSS
* JavaScript (Node.js)

If you can build a website, you can build a desktop application.

Electron applications run on Windows, macOS, and Linux using a single codebase.

---


## Real Applications Built with Electron

* Visual Studio Code
* Discord
* Figma (desktop client)
* Postman
* Slack

Electron is widely used in production-grade software.

---

## How Electron Works

Electron applications consist of two main processes:

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

### Main Process

* Entry point of the application
* Creates and manages windows
* Has access to Node.js and operating system APIs
* Controls the lifecycle of the application

### Renderer Process

* Runs inside each BrowserWindow
* Displays HTML, CSS, and JavaScript
* Handles user interface and interactions

The Main process and Renderer process communicate using IPC.

---

# Part 2 — Creating a New Electron Project (Modern Method)

Instead of manually creating files, we use a project generator.

We will use Electron Forge.

---

## Prerequisites

Install Node.js from:

[https://nodejs.org](https://nodejs.org)

Verify installation:

```bash
node -v
npm -v
```

---

## Step 1 — Create a New Electron Project

```bash
npx create-electron-app my-electron-app
cd my-electron-app
```

This command:

* Creates the project folder
* Installs Electron
* Configures the build system
* Sets up project structure

---

## Step 2 — Run the Application

```bash
npm start
```

A desktop window opens.
This is your first Electron application.

---

## Default Project Structure

```
my-electron-app/
├── src/
│   ├── main.js
│   └── index.html
├── package.json
└── node_modules/
```

---

# Part 3 — Understanding the Main Process

Open:

```
src/main.js
```

You will see something similar to:

```js
const { app, BrowserWindow } = require('electron');
const path = require('path');

function createWindow() {
  const mainWindow = new BrowserWindow({
    width: 900,
    height: 650
  });

  mainWindow.loadFile('src/index.html');
}

app.whenReady().then(createWindow);
```

Key concepts:

* `app` controls the lifecycle of the application
* `BrowserWindow` creates a native desktop window
* `loadFile()` loads your HTML UI

---

# Part 4 — IPC (Inter-Process Communication)

The Renderer process cannot directly access Node.js APIs for security reasons.

To communicate with the Main process, we use IPC.

```
Renderer (UI)  <---IPC--->  Main Process
```

---

## IPC Patterns

| Pattern                                 | Purpose               |
| --------------------------------------- | --------------------- |
| `ipcRenderer.send` + `ipcMain.on`       | One-way communication |
| `ipcRenderer.invoke` + `ipcMain.handle` | Request and response  |

We will use the request/response pattern.

---

## Step 1 — Setup Preload Bridge

Open:

```
src/preload.js
```

Add:

```js
const { contextBridge, ipcRenderer } = require('electron');

contextBridge.exposeInMainWorld('api', {
  sendMessage: (msg) => ipcRenderer.invoke('send-message', msg)
});
```

---

## Step 2 — Handle IPC in Main Process

In `main.js`, add:

```js
const { ipcMain } = require('electron');

ipcMain.handle('send-message', (event, message) => {
  return 'Main Process received: ' + message;
});
```

---

## Step 3 — Update the UI

Open:

```
src/index.html
```

Replace body with:

```html
<h2>IPC Demo</h2>
<button id="btn">Send Message to Main</button>
<p id="response"></p>

<script>
  document.getElementById('btn').addEventListener('click', async () => {
    const reply = await window.api.sendMessage('Hello from Renderer');
    document.getElementById('response').innerText = reply;
  });
</script>
```

Run:

```bash
npm start
```

Click the button to see IPC in action.

---

# Part 5 — Packaging and Distribution

Once your application is ready, you can package it for distribution.

Electron Forge is already configured in the project.

---

## Package the Application

```bash
npm run package
```

Creates a build inside the `out/` folder.

---

## Create an Installer

```bash
npm run make
```

Output formats:

| Platform | Output           |
| -------- | ---------------- |
| Windows  | `.exe` installer |
| macOS    | `.dmg`           |
| Linux    | `.deb` or `.rpm` |

---

# Production Project Structure Example

```
my-electron-app/
├── src/
│   ├── main.js
│   ├── preload.js
│   ├── index.html
│   └── renderer.js
├── assets/
│   └── icon.png
├── package.json
└── out/
```

---

# Quick Reference

| Concept              | Description                          |
| -------------------- | ------------------------------------ |
| `BrowserWindow`      | Creates a native application window  |
| `app.whenReady()`    | Runs when Electron is initialized    |
| `ipcMain.handle`     | Handles requests from the Renderer   |
| `ipcRenderer.invoke` | Sends request and waits for response |
| `contextBridge`      | Securely exposes APIs to the UI      |
| Electron Forge       | Builds and packages applications     |

---

# Topics to Explore Next

* Multiple windows
* Keyboard shortcuts
* Local storage integration
* Using React, Vue, or Svelte in the Renderer

---

# Resources

* Official Docs: [https://www.electronjs.org/docs/latest](https://www.electronjs.org/docs/latest)
* Electron Forge: [https://www.electronforge.io](https://www.electronforge.io)
* Electron Fiddle: [https://www.electronjs.org/fiddle](https://www.electronjs.org/fiddle)


