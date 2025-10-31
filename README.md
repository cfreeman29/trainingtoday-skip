# Training Today Auto-Complete Bypass

A console snippet that bypasses Training Today content and automatically completes training modules. This tool exploits the SCORM 2004 runtime API to mark courses as completed without requiring you to watch all the content.

__⚠️ Use responsibly__ — This is for personal use only and may violate your organization's training policies.

---

## How to Use

### Basic Usage

1. __Start your training__ — Open the Training Today course in your browser and begin playing the content
2. __Open Developer Tools__ — Press `F12` or `Ctrl+Shift+I` (Windows/Linux) / `Cmd+Option+I` (Mac) to open the browser console
3. __Paste the snippet__ — Copy the contents of `console_snippet.js` and paste it into the console
4. __Press Enter__ — Execute the snippet
5. __Refresh the page__ — Press `F5` to refresh the Training Today page
6. __Check completion__ — Navigate back to the training menu/dashboard to see the course marked as completed

### Allow Pasting in Console

If your browser blocks pasting in the console for security reasons, you'll see a message like "Pasting is disabled in this console." To enable it:

__Chrome/Brave/Edge:__

- Type `allow pasting` in the console and press Enter to bypass the security restriction

__Firefox:__

- Go to `about:config` in the address bar
- Search for `devtools.selfxss.count`
- Set it to a value ≥ 5 (or manually type your code instead of pasting)

---

## How It Works

The snippet directly calls the __SCORM 2004 Runtime API__ that the course SCO (Shareable Content Object) normally uses to communicate learner progress to the LMS (Learning Management System).

### The SCORM API Call Sequence

```javascript
// 1. Get reference to the SCORM API
const API = window.API || window.parent.API || window.top.API;

// 2. Update learner state fields
API.SetValue("cmi.progress_measure", "1");           // 100% progress
API.SetValue("cmi.score.scaled", "1");               // Passing score
API.SetValue("cmi.success_status", "passed");        // Mark as passed
API.SetValue("cmi.completion_status", "completed");  // Mark as completed

// 3. Commit the changes to the LMS
API.Commit();

// 4. Terminate the session
API.Terminate();
```

### Key Fields Explained

| Field | Value | Purpose | |-------|-------|---------| | `cmi.progress_measure` | `1` (1.0 = 100%) | Sets learner progress to 100% | | `cmi.score.scaled` | `1` (1.0 = 100%) | Sets test score to passing (most LMS require 0.5–1.0) | | `cmi.success_status` | `"passed"` | Explicitly marks the SCO as passed | | `cmi.completion_status` | `"completed"` | Explicitly marks the SCO as completed |

---

## Why It Works

The SCORM 2004 API is the __official, client-accessible channel__ for reporting learning state to the LMS. When you call these functions:

1. __SetValue()__ updates the in-memory learning record with progress, score, success, and completion fields
2. __Commit()__ persists that state to the LMS server, telling it to save the learner's progress
3. __Terminate()__ ends the session, prompting the LMS to finalize and credit the SCO

By setting the right combination of fields:

- `cmi.progress_measure` satisfies heuristics that check for forward progress
- `cmi.score.scaled` satisfies heuristics that require a passing score
- `cmi.success_status = "passed"` supplies one canonical indicator for completion
- `cmi.completion_status = "completed"` supplies the second canonical indicator

Most LMS systems consider an SCO __successfully completed__ when both `success_status` and `completion_status` are properly set, making this approach effective for bypassing content requirements.

### Limitations

This __may not work__ if your LMS enforces:

- __Server-side validation__ — Backend checks that verify actual content consumption
- __Rollup rules__ — Parent course requirements that the child module meets
- __Signed launch tokens__ — Security tokens that validate the launch session
- __Minimum time heuristics__ — Requires spending a minimum duration in the course
- __Activity tracking__ — Logs that require interaction with course elements

---

## Tips for Using

### 1. __Automate with Browser Extension or Tampermonkey Script__

Instead of manually pasting into the console each time, create a __Tampermonkey script__ to run automatically:

```javascript
// ==UserScript==
// @name         Training Today Auto-Complete
// @namespace    http://tampermonkey.net/
// @version      1.0
// @description  Auto-complete Training Today courses
// @match        https://trainingtoday.com/*
// @grant        none
// ==/UserScript==

(function() {
  // Wait for SCORM API to be available
  setTimeout(() => {
    const API = window.API || window.parent.API || window.top.API;
    if (API) {
      API.SetValue("cmi.progress_measure", "1");
      API.SetValue("cmi.score.scaled", "1");
      API.SetValue("cmi.success_status", "passed");
      API.SetValue("cmi.completion_status", "completed");
      API.Commit();
      API.Terminate();
      console.log("Training auto-completed!");
    }
  }, 2000); // Wait 2 seconds for API to initialize
})();
```

Install Tampermonkey from:

- [Chrome/Edge](https://chromestore.google.com/detail/tampermonkey/dhdgffkkebhmkfjojejmpbldmpobgkta)
- [Firefox](https://addons.mozilla.org/en-US/firefox/addon/tampermonkey/)

### 2. __Create a Bookmarklet__

Save this as a bookmark to quickly execute the snippet:

```javascript
javascript:(() => {const API = window.API || window.parent.API || window.top.API; if(API){API.SetValue("cmi.progress_measure", "1");API.SetValue("cmi.score.scaled", "1");API.SetValue("cmi.success_status", "passed");API.SetValue("cmi.completion_status", "completed");API.Commit();API.Terminate();}})();
```

Steps:

1. Right-click your bookmark bar → __Bookmark Manager__
2. Click __⋮__ (three dots) → __Add new bookmark__
3. __Name:__ `Training Today Auto-Complete`
4. __URL:__ Paste the bookmarklet code above
5. Click __Save__
6. When needed, click the bookmark to auto-complete

### 3. __Check If the Script Succeeded__

Add error handling to verify the snippet worked:

```javascript
const API = window.API || window.parent.API || window.top.API;
if (!API) {
  console.error("❌ SCORM API not found. Course may not be SCORM-compliant.");
} else {
  try {
    API.SetValue("cmi.progress_measure", "1");
    API.SetValue("cmi.score.scaled", "1");
    API.SetValue("cmi.success_status", "passed");
    API.SetValue("cmi.completion_status", "completed");
    API.Commit();
    API.Terminate();
    console.log("✅ Training marked as complete!");
  } catch (e) {
    console.error("❌ Error:", e);
  }
}
```

### 4. __Timing Issues__

If the API isn't immediately available:

```javascript
// Retry every 500ms for up to 10 seconds
let attempts = 0;
const interval = setInterval(() => {
  const API = window.API || window.parent.API || window.top.API;
  if (API) {
    clearInterval(interval);
    API.SetValue("cmi.progress_measure", "1");
    API.SetValue("cmi.score.scaled", "1");
    API.SetValue("cmi.success_status", "passed");
    API.SetValue("cmi.completion_status", "completed");
    API.Commit();
    API.Terminate();
    console.log("✅ Training marked as complete!");
  }
  if (++attempts > 20) {
    clearInterval(interval);
    console.error("❌ SCORM API never became available");
  }
}, 500);
```

### 5. __Troubleshooting__

| Problem | Solution | |---------|----------| | API not found | Course may not use SCORM. Check the Network tab for API launch parameters. | | Changes don't persist | LMS may have server-side validation. Contact IT for alternatives. | | Session terminated too quickly | Wait a few seconds after refresh before checking completion. | | Can't paste in console | Type `allow pasting` in console or switch browsers. |

---

## What's in This Repo

- __console_snippet.js__ — The core script to paste into your browser console

- __web_source_ref/__ — Reference files extracted from Training Today to understand the SCORM implementation:

  - `index_lms.html` — LMS launch page
  - `scormdriver.js` — SCORM API driver implementation

- __README.md__ — This file

---

## Disclaimer

This tool is provided for educational purposes. Using it to bypass required training may:

- Violate your organization's acceptable use policy
- Result in disciplinary action
- Compromise compliance certifications

Use at your own risk and only in scenarios where you have authorization.

---

## References

- [SCORM 2004 Standard](https://scorm.com/scorm-explained/technical-scorm/)
- [SCORM API Reference](https://scorm.com/wp-content/assets/courseware_packages_uploads/SCORM_2004_3RD_EDITION_BASE.pdf)
- [MDN: Working with Browser DevTools](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/Tools_and_setup/What_are_browser_developer_tools)
