---
name: chrome-extension
description: Chrome extension development with Manifest V3. Trigger words - chrome extension, browser extension, manifest v3, content script, popup, background script, service worker extension
---

# Chrome Extension Development

Build browser extensions with Manifest V3 using HTML, CSS, and JavaScript.

## When to Use This Skill

- Building browser extension functionality
- Need to inject scripts into web pages
- Want popup interface from browser toolbar
- Need background processing in browser

## Project Structure

```
my-extension/
├── manifest.json           # Required config
├── background/
│   └── service-worker.js   # Background tasks
├── content/
│   └── content-script.js   # Inject into pages
├── popup/
│   ├── popup.html          # Toolbar popup
│   ├── popup.js
│   └── popup.css
├── options/
│   ├── options.html        # Settings page
│   └── options.js
└── assets/icons/
    ├── icon16.png
    ├── icon48.png
    └── icon128.png
```

## manifest.json

```json
{
  "manifest_version": 3,
  "name": "My Extension",
  "version": "1.0.0",
  "description": "What your extension does",

  "icons": {
    "16": "assets/icons/icon16.png",
    "48": "assets/icons/icon48.png",
    "128": "assets/icons/icon128.png"
  },

  "action": {
    "default_popup": "popup/popup.html",
    "default_icon": {
      "16": "assets/icons/icon16.png",
      "48": "assets/icons/icon48.png"
    }
  },

  "background": {
    "service_worker": "background/service-worker.js"
  },

  "content_scripts": [
    {
      "matches": ["https://*/*"],
      "js": ["content/content-script.js"],
      "run_at": "document_idle"
    }
  ],

  "permissions": ["storage", "activeTab"],
  "host_permissions": ["https://api.example.com/*"],
  "options_page": "options/options.html"
}
```

## Background Service Worker

```javascript
// background/service-worker.js

// On install
chrome.runtime.onInstalled.addListener(({ reason }) => {
  if (reason === 'install') {
    chrome.storage.sync.set({ enabled: true });
  }
});

// Listen for messages
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.action === 'getData') {
    fetchData()
      .then(data => sendResponse({ success: true, data }))
      .catch(err => sendResponse({ success: false, error: err.message }));
    return true; // Keep channel open for async
  }
});

// Schedule tasks
chrome.alarms.create('dailyTask', { periodInMinutes: 1440 });

chrome.alarms.onAlarm.addListener((alarm) => {
  if (alarm.name === 'dailyTask') {
    performDailyTask();
  }
});
```

**Important:** Service workers terminate when idle (~30 seconds). Use `chrome.storage` not variables, use `chrome.alarms` not `setTimeout`.

## Content Script

```javascript
// content/content-script.js

// Runs on matching pages
console.log('Content script loaded:', window.location.href);

// Modify page DOM
document.body.style.backgroundColor = '#f0f0f0';

// Listen for messages from popup/background
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.action === 'highlight') {
    highlightText();
    sendResponse({ success: true });
  }
});

// Send message to background
chrome.runtime.sendMessage({ action: 'pageVisited', url: location.href });

// Get settings
chrome.storage.sync.get(['enabled'], (result) => {
  if (result.enabled) {
    initFeature();
  }
});
```

## Popup UI

```html
<!-- popup/popup.html -->
<!DOCTYPE html>
<html>
<head>
  <link rel="stylesheet" href="popup.css">
</head>
<body>
  <h1>My Extension</h1>
  <label>
    <input type="checkbox" id="enableToggle">
    Enable
  </label>
  <button id="actionBtn">Run Action</button>
  <script src="popup.js"></script>
</body>
</html>
```

```javascript
// popup/popup.js
const enableToggle = document.getElementById('enableToggle');
const actionBtn = document.getElementById('actionBtn');

// Load settings
chrome.storage.sync.get(['enabled'], (result) => {
  enableToggle.checked = result.enabled !== false;
});

// Save settings
enableToggle.addEventListener('change', (e) => {
  chrome.storage.sync.set({ enabled: e.target.checked });
});

// Send message to current tab
actionBtn.addEventListener('click', async () => {
  const [tab] = await chrome.tabs.query({ active: true, currentWindow: true });
  chrome.tabs.sendMessage(tab.id, { action: 'highlight' });
});
```

## Storage API

```javascript
// Sync storage (syncs across devices, 100KB limit)
chrome.storage.sync.set({ key: 'value' });
chrome.storage.sync.get(['key'], (result) => console.log(result.key));

// Local storage (local only, 10MB limit)
chrome.storage.local.set({ largeData: {...} });

// Listen for changes
chrome.storage.onChanged.addListener((changes, areaName) => {
  for (let key in changes) {
    console.log(`${key} changed from ${changes[key].oldValue} to ${changes[key].newValue}`);
  }
});
```

## Context Menus

```javascript
// In service worker
chrome.runtime.onInstalled.addListener(() => {
  chrome.contextMenus.create({
    id: 'myMenuItem',
    title: 'Send to Extension',
    contexts: ['selection']
  });
});

chrome.contextMenus.onClicked.addListener((info, tab) => {
  if (info.menuItemId === 'myMenuItem') {
    console.log('Selected text:', info.selectionText);
  }
});
```

## Load Extension for Testing

1. Go to `chrome://extensions/`
2. Enable "Developer mode"
3. Click "Load unpacked"
4. Select your extension directory

## Tips

- Use `chrome.storage` not variables (service worker terminates)
- Use `chrome.alarms` not `setTimeout` for scheduling
- Content scripts share DOM but have isolated JavaScript
- Use `return true` in message listener for async responses

## How to Verify

### Quick Checks
- Extension loads without errors in chrome://extensions
- Popup opens when clicking toolbar icon
- Content script runs on target pages

### Common Issues
- "Service worker inactive": Code must be event-driven
- Content script not running: Check `matches` pattern
- Storage not persisting: Use chrome.storage, not localStorage
