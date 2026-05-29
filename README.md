# Screen Time — Chrome Extension

> A privacy-first browser time tracker inspired by Android's Digital Wellbeing.  
> Know exactly where your attention goes — by category, by site, by week.

---

## Table of Contents

1. [What It Does](#what-it-does)
2. [Features](#features)
3. [Categories & Websites](#categories--websites)
4. [The Dashboard](#the-dashboard)
5. [Data Storage & Privacy](#data-storage--privacy)
6. [How Time Tracking Works](#how-time-tracking-works)
7. [Daily Reset & Weekly Archival](#daily-reset--weekly-archival)
8. [Installation](#installation)
9. [File Structure](#file-structure)

---

## What It Does

Screen Time runs silently in the background and records how long your browser's active tab is open on any given website. At the end of each day you have a full breakdown of your browsing — how many minutes went to social media, how long you spent on YouTube, how much time on AI tools — presented in a clean dashboard that mirrors the feel of Google's Digital Wellbeing on Android.

Everything is stored **locally on your machine**. No account. No server. No analytics. No data ever leaves your browser.

---

## Features

### Real-Time Tracking
- Tracks the **active, focused tab** only — time spent on a tab you're not looking at is never counted
- Automatically **pauses** when you switch to another app or minimize the browser (window focus detection)
- Automatically **pauses when the screen is locked** — uses the `chrome.idle` API which detects the OS-level lock state, not just app focus. Resumes only after you unlock and Chrome is in the foreground again
- Automatically **resumes** the moment you come back to the browser
- Saves accumulated time to local storage every **60 seconds** via a Chrome alarm, so a crash or browser close loses at most ~1 minute of data
- Each periodic save is **capped at 90 seconds** — if Chrome's service worker was suspended and woke up with a stale timestamp, it cannot inflate hours of phantom time in a single tick

### Popup Summary (Click the Toolbar Icon)
- Shows **today's total screen time** at a glance
- Mini donut chart with color-coded category segments
- Top 6 categories listed with individual times and proportional bars
- One-tap **"View Full Report"** button to open the full dashboard

### Full Dashboard
- Opens as a **dedicated browser tab** (`dashboard.html`)
- Large **ring/donut chart** — each arc segment represents one category, color-coded and proportional to time spent
- **Legend chips** below the ring showing category name, time, and percentage
- Full **category breakdown** with expandable rows — click any category to reveal every individual site tracked within it, each with its own time and a relative bar
- **Weekly bar chart** for the current week — one bar per day (Mon–Sun), today highlighted in blue, future days shown as empty
- **Historical weekly summaries** for the last 12 weeks, showing total time and per-category breakdown

### Smart Categorization
- **9 categories** covering 130+ domains out of the box
- Any site not matched falls into **Other** — it is still tracked individually, just not labeled
- Category matching supports **subdomains** automatically (e.g. `mail.google.com` matches `gmail.com`'s entry; any subdomain of a listed domain is captured)
- Categories are defined in a single shared file (`categories.js`) — easy to edit

### Daily Reset
- At **midnight (00:00) every day**, today's data is archived into the weekly store and the daily counter resets to zero
- Uses a Chrome alarm scheduled precisely to the next midnight, re-scheduled each day
- If the browser was closed at midnight and reopened the next day, the reset happens automatically on startup

### Weekly Archival & Summaries
- During the current week, **per-day, per-site data** is kept in full — you can see exactly what you visited each day
- When the week ends (Monday 00:00), the week's data is **compressed to category totals only** — individual site names are discarded for past weeks
- Up to **12 weeks** of summaries are retained; older entries are automatically dropped
- This design means your detailed browsing history is never stored long-term — only aggregates survive past 7 days

---

## Categories & Websites

| Category | Color | Notable Sites Tracked |
|---|---|---|
| **Social Media** | Pink `#E91E63` | Facebook, Instagram, X/Twitter, LinkedIn, Reddit, Snapchat, TikTok, Pinterest, Threads, Bluesky, Quora, Tumblr, Mastodon |
| **Video** | Red `#F44336` | YouTube, Netflix, Twitch, Prime Video, Hotstar, JioCinema, Disney+, Crunchyroll, Hulu, Vimeo, Zee5, SonyLIV, MX Player |
| **AI Tools** | Violet `#7C4DFF` | ChatGPT, Claude, Gemini, Perplexity, Microsoft Copilot, GitHub Copilot, Grok, Character.AI, Poe, Mistral, Hugging Face, Phind, Pi, Cohere, Groq, Replicate, You.com |
| **News** | Orange `#FF9800` | BBC, CNN, NYTimes, The Guardian, Reuters, TechCrunch, Medium, The Hindu, NDTV, Hindustan Times, Times of India, Bloomberg, Forbes, The Verge, Wired, Ars Technica, HN |
| **Shopping** | Purple `#9C27B0` | Amazon (.com + .in), Flipkart, eBay, Etsy, Myntra, Meesho, Nykaa, AJIO, Snapdeal, Walmart, Target |
| **Productivity** | Green `#4CAF50` | Google Docs/Sheets/Slides/Drive, **Google Keep**, **Google Calendar**, Notion, Trello, Asana, GitHub, GitLab, Figma, Canva, Airtable, Monday, Linear, ClickUp, Atlassian, Basecamp, Stack Overflow |
| **Search** | Blue `#2196F3` | Google, Bing, DuckDuckGo, Yahoo, Ecosia, Yandex, Startpage |
| **Communication** | Teal `#00BCD4` | Gmail, Outlook, Slack, Discord, WhatsApp Web, Telegram Web, Zoom, Google Meet, Microsoft Teams, Skype, Signal |
| **Education** | Deep Orange `#FF5722` | Coursera, Udemy, Khan Academy, edX, Udacity, Pluralsight, Skillshare, Duolingo, Wikipedia, Britannica, Google Scholar, JSTOR, ResearchGate, arXiv |
| **Other** | Gray `#78909C` | Any site not matched above — still tracked individually |

> **To add a site to a category:** open `categories.js` and add the domain string to the relevant `domains` array. Reload the extension. Done.

---

## The Dashboard

The dashboard is built entirely with vanilla HTML, CSS, and JavaScript — no frameworks, no external dependencies.

### Ring Chart
The large donut at the top is drawn with SVG `<circle>` elements using the `stroke-dasharray` / `stroke-dashoffset` technique. Each category gets one circle element; the arc length is proportional to its share of total time. A small gap between segments keeps them visually distinct. The chart starts at 12 o'clock (achieved by a `-90deg` rotation on the parent group).

### Category Rows
Each category row shows:
- A colored left-border bar indicating the category
- Category name and its percentage of the day's total
- Total time in human-readable format (`1h 30m`, `45m`, `30s`)
- A proportional progress bar (relative to the largest category, not total time)
- A chevron — clicking the row expands it to show every individual domain with its own time and bar

### Weekly Bar Chart
The current week shows 7 columns (Mon–Sun). Bar height is proportional to that day's total screen time relative to the busiest day of the week. Today is highlighted in blue; future days are shown as empty gray stubs. Below the current week, past weeks appear as compact rows with category chips.

---

## Data Storage & Privacy

### Where Data Lives

All data is stored using the **`chrome.storage.local` API**, which writes to a sandboxed database inside Chrome's profile directory on your local machine:

| OS | Approximate path |
|---|---|
| Windows | `C:\Users\<you>\AppData\Local\Google\Chrome\User Data\Default\Local Extension Settings\<extension-id>\` |
| macOS | `~/Library/Application Support/Google/Chrome/Default/Local Extension Settings/<extension-id>/` |
| Linux | `~/.config/google-chrome/Default/Local Extension Settings/<extension-id>/` |

This is a LevelDB database managed entirely by Chrome. It is:
- **Never synced** to Google's servers (that would require `chrome.storage.sync`, which is not used here)
- **Never sent anywhere** — the extension makes zero network requests
- **Not accessible** to any website or other extension
- **Deleted automatically** if you uninstall the extension

### What Exactly Is Stored

```
chrome.storage.local
│
├── today
│   ├── date: "2026-05-27"
│   └── sites: { "youtube.com": 3420, "github.com": 1800, ... }  ← seconds
│
├── weekData
│   ├── "2026-05-25": { "youtube.com": 5400, ... }
│   ├── "2026-05-26": { "reddit.com": 1200, ... }
│   └── ...  ← one entry per completed day this week
│
├── currentWeekStart
│   └── "2026-05-25"  ← ISO date of this week's Monday
│
└── weeklyHistory  ← array of up to 12 past week summaries
    ├── { weekStart: "2026-05-18", categories: { "Video": 18000, "Social Media": 9000, ... } }
    └── ...
```

A separate `chrome.storage.session` key (`t`) holds the in-progress tracking state (current URL + timestamp). This is volatile — it resets when the browser closes, by design.

### Data Lifecycle Summary

| Data | Granularity | Retention |
|---|---|---|
| Today's browsing | Per-site, per-second | Until midnight, then archived |
| This week's days | Per-site, per-second | Until Sunday, then compressed |
| Past weekly summaries | Category totals only | Up to 12 weeks, then dropped |

After a week ends, **individual site names from that week are permanently discarded**. Only the total time per category survives. This means your long-term history never reveals which specific sites you visited — only broad behavioural patterns (e.g. "you spent 4 hours on Video last week").

### What Is Never Collected

- Page titles or URLs beyond the domain name
- Scroll position, clicks, keystrokes, or any page content
- Your IP address or location
- Any personally identifiable information
- Data from incognito windows (Chrome extensions cannot access incognito tabs unless explicitly granted permission — this extension does not request that permission)

### No Network Requests

The extension has no `fetch`, `XMLHttpRequest`, or WebSocket calls anywhere in its code. It cannot communicate with any external server. You can verify this yourself by opening DevTools on the background service worker (`chrome://extensions` → click "Service Worker" link) and checking the Network tab — it will be empty.

---

## How Time Tracking Works

1. **Tab activation** — when you switch tabs, the previous tab's elapsed time is computed and saved, and tracking begins fresh on the new tab
2. **Tab navigation** — when a tab finishes loading a new URL while active, same flush-and-restart logic applies
3. **Window focus** — when Chrome loses focus (you Alt+Tab to another app), tracking pauses immediately via `chrome.windows.onFocusChanged`. It resumes the moment Chrome is focused again
4. **Screen lock** — handled separately via `chrome.idle.onStateChanged`. When the OS reports `"locked"`, tracking pauses and the start timestamp is cleared. `chrome.windows.onFocusChanged` alone is not enough here — on Windows, locking the screen (Win+L) is a system-level overlay that does not cause Chrome to emit a focus-change event, so without the idle API the timer would keep running through a locked screen
5. **Periodic flush** — every 60 seconds a Chrome alarm fires and saves the current session's elapsed time to storage, then resets the start timestamp. This is the heartbeat that prevents data loss. Each flush is capped at 90 seconds: if Chrome's MV3 service worker was suspended and woke up with a timestamp from hours ago, it saves at most 90 seconds rather than inflating the count with fake time
6. **Service worker restarts** — Chrome's MV3 service workers can be suspended to save memory. The current URL and tracking start time are stored in `chrome.storage.session` (which persists within a browser session) so tracking resumes correctly after a restart

### What Does Not Count
- Time on `chrome://` pages (new tab, settings, extensions)
- Time on `chrome-extension://` pages (including this dashboard)
- Time in incognito windows
- Time when the browser window is not focused
- Time when the screen is locked (detected via `chrome.idle` locked state)

### Split Screen Behaviour
Running two Chrome windows side by side (Windows Snap) does **not** mean both tabs are tracked simultaneously. The Chrome extension API enforces a hard rule: one active tab per window, one focused window at a time. Whichever window you last clicked in is the focused one — its active tab is tracked. The other window is fully paused, even if it is visible on screen. This is a browser-level constraint, not a limitation of this extension.

---

## Daily Reset & Weekly Archival

### Daily Reset (midnight)
```
At 00:00:
  1. Flush any in-progress session time for the current tab
  2. Archive today's { date, sites } object into weekData
  3. Set today = { date: new day, sites: {} }
  4. Schedule the next midnight alarm
```

### Weekly Archival (when week boundary is crossed)
```
When a new day's week-start ≠ stored currentWeekStart:
  1. Summarize weekData: for each domain across all days,
     look up its category and sum seconds into category totals
  2. Push { weekStart, categories } to weeklyHistory
  3. Trim weeklyHistory to last 12 entries
  4. Reset weekData to empty
  5. Update currentWeekStart
```

This compression step is what ensures individual site names never accumulate in long-term storage.

---

## Known Bugs Fixed

### Timezone mismatch causing multi-day accumulation (fixed)
**Symptom:** After using the extension for a few days, the daily counter showed impossible values (e.g. 34+ hours) because time from multiple days was lumped into one "today."

**Root cause:** `getTodayDate()` and `getWeekStart()` both used JavaScript's `.toISOString()`, which returns the date in UTC. For users in UTC+ timezones (e.g. India, IST = UTC+5:30), local midnight arrives while UTC is still on the previous day. So when the midnight alarm fired and compared `today.date` against `getTodayDate()`, both returned the same UTC date string — the reset condition was never true, and the day never rolled over.

The same offset caused `getWeekStart()` to return Sunday instead of Monday for UTC+ users, shifting every bar in the weekly chart one column to the right.

**Fix:** Both functions now build the date string from local date parts (`d.getFullYear()`, `d.getMonth()`, `d.getDate()`) instead of parsing `.toISOString()`. `getWeekStart()` also uses `new Date(year, month, day)` (local date constructor) rather than a string, which avoids UTC midnight shifting the date when parsed.

### Service worker stale timestamp inflating time (fixed)
**Symptom:** Occasionally, a large chunk of time (several hours) was added to a site in a single tick, with no corresponding real browsing.

**Root cause:** Chrome's MV3 service worker is aggressively suspended by the browser to save memory. The in-progress tracking state (current URL + start timestamp) is stored in `chrome.storage.session`. When the service worker woke up for the periodic tick alarm, it found a start timestamp from potentially hours ago and computed the full elapsed time as legitimate browsing time.

**Fix:** Each call to `flushTime()` now caps elapsed time at 90 seconds. Since the tick fires every 60 seconds, any gap larger than 90 seconds is evidence of a suspension — that gap is discarded rather than counted.

---

## Installation

### Prerequisites
- Google Chrome (any recent version)
- Node.js (only needed once, to generate the icon PNG files)

### Steps

```bash
# 1. Clone or download the extension folder
# (already at C:\Users\<you>\screen-time-extension)

# 2. Generate the icons (one-time setup)
cd screen-time-extension
node make_icons.js
# Creates icons/icon16.png, icons/icon48.png, icons/icon128.png
```

Then in Chrome:
1. Navigate to `chrome://extensions`
2. Enable **Developer mode** (toggle, top-right)
3. Click **Load unpacked**
4. Select the `screen-time-extension` folder
5. The extension icon appears in the toolbar — pin it for easy access

### Updating categories
Open `categories.js`, edit the `domains` array for any category (or add a new category object), save the file, then click the reload icon on `chrome://extensions`. Changes take effect immediately.

---

## File Structure

```
screen-time-extension/
│
├── manifest.json          Chrome extension manifest (MV3)
├── background.js          Service worker — tracking engine, alarms, storage
├── categories.js          Shared category definitions + helper functions
│
├── popup.html             Toolbar popup markup
├── popup.css              Popup styles
├── popup.js               Popup data loading + rendering
│
├── dashboard.html         Full report page markup
├── dashboard.css          Dashboard styles (Digital Wellbeing aesthetic)
├── dashboard.js           Dashboard data loading + SVG ring + weekly charts
│
├── make_icons.js          One-time Node.js script to generate PNG icons
│
└── icons/
    ├── icon16.png         Toolbar icon (16×16)
    ├── icon48.png         Extensions page icon (48×48)
    ├── icon128.png        Chrome Web Store icon (128×128)
    └── generate_icons.html  Alternative browser-based icon generator
```

---

*Built with zero external dependencies. Pure Chrome extension APIs, vanilla JS, SVG, and CSS.*
