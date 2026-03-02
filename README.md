# FYAT Routine Mapper

A lightweight, browser-based tool for BRACU students to visualize FYAT member availability across the weekly class schedule. Upload your group's course registrations and instantly see which time slots are free — no backend, no login, no install.

---

## What It Does

You enter your own registered courses. Your FYAT members submit their courses through a Google Form. You download the responses as a CSV, upload it here, and the tool generates a color-coded weekly grid showing how many members are free in each slot — with a click-to-inspect modal for individual availability.

---

## Features

- **Step-by-step accordion UI** — guided workflow from form creation to map generation
- **Auto-generated Google Form** — one Apps Script snippet creates the collection form in your Drive
- **Live schedule data** — course timetable JSON is fetched from Supabase Storage, no file management needed
- **Heatmap grid** — cells are color-interpolated from busy (red) to free (green) based on availability ratio
- **Click any cell** — modal shows each member's individual status (Busy / Free) for that slot
- **Your own slots highlighted** — your enrolled courses are marked separately in blue
- **Light / Dark theme toggle** — defaults to light, preference saved to localStorage
- **Single HTML file** — zero dependencies, works offline after first JSON fetch

---

## Project Structure

```
fyat-routine-mapper/
├── fyat_app.html        # The entire app — HTML, CSS, and JS in one file 
└── README.md
```

---

## Setup

### 1. Host the Schedule JSON

The app fetches `connect.json` (the full BRACU course timetable) from a public URL. The easiest option is Supabase Storage.

1. Create a free project at [supabase.com](https://supabase.com)
2. Go to **Storage → New Bucket**, name it anything, set it to **Public**
3. Upload `connect.json` to the bucket
4. Copy the public URL — it looks like:
   ```
   https://xxxxxxxxxxxx.supabase.co/storage/v1/object/public/your-bucket/connect.json
   ```
5. Open `fyat_app.html` and replace the `JSON_URL` constant near the top of the `<script>` block:
   ```js
   const JSON_URL = 'https://xxxxxxxxxxxx.supabase.co/storage/v1/object/public/your-bucket/connect.json';
   ```

Alternatively you can use a GitHub raw URL if the repo is public, or any static file host.

---

### 2. Create the Google Form

1. Go to [script.google.com](https://script.google.com) and create a new project
2. Paste the Apps Script code shown in **Step 01** of the app
3. Click **Run ▶** and grant Drive permissions
4. A form titled **FYAT Session Availability** will appear in your Google Drive
5. Share the form link with your FYAT members

---

### 3. Export Responses

Once all members have submitted:

1. Open the form → **Responses** tab → click the **Sheets icon**
2. In Google Sheets: **File → Download → Comma Separated Values (.csv)**

The CSV column order should be: `Timestamp, Email, Name, Student ID, Courses`

---

### 4. Generate the Map

1. Open `fyat_app.html` in any browser
2. Complete the four steps in the accordion:
   - Step 01 — Create & share the form *(one-time)*
   - Step 02 — Export responses as CSV
   - Step 03 — Enter your name, your courses, upload the CSV
   - Step 04 — View the generated availability map

---

## CSV Format

The parser is flexible — it uses regex to extract course codes and sections, so minor formatting differences are handled automatically.

Expected format for the courses field:
```
CSE110-02, MAT110-05, ENG101-11, PHY111-23
```

The full row structure expected from Google Forms export:
```
Timestamp, Email, Name, Student ID, Registered Courses
```

---

## Tech Stack

| Layer | Choice |
|---|---|
| UI | Vanilla HTML / CSS / JS — no framework |
| Fonts | Syne (display) + JetBrains Mono (mono) via Google Fonts |
| Schedule data | Supabase Storage (public JSON) |
| Form collection | Google Forms + Apps Script |
| CSV parsing | Custom regex-based parser |
| Hosting | Any static host — GitHub Pages, Netlify, local file |

---

## Customization

**Change whose slot is highlighted in blue**
In Step 03, enter your name and courses. The blue cells are generated from whatever you input — it is not hardcoded.

**Update the schedule data**
Re-upload a new `connect.json` to your Supabase bucket. The app fetches it fresh on each session (it caches within the same tab to avoid redundant requests).

**Change the default theme**
The app defaults to light mode. To default to dark, find this block at the bottom of the HTML file and change the condition:
```js
// Default light — change condition to default dark:
if (localStorage.getItem('theme') !== 'light') {
  document.body.classList.add('dark');
  document.getElementById('themeIcon').textContent = '☀';
}
```

**Time slots**
The weekly slots are defined in the `SLOTS` array in the script block. Edit the start/end times and display labels there if your university schedule changes.

---

## Deployment

Since the entire app is a single HTML file, deployment is trivial:

**GitHub Pages**
```
Settings → Pages → Source: main branch → / (root)
```
Access at `https://yourusername.github.io/fyat-routine-mapper/fyat_app.html`

**Netlify / Vercel**
Drop the file into a new project — it deploys instantly with no build step.

**Local**
Just open `fyat_app.html` directly in a browser. Note that the JSON fetch requires either a local server or CORS-permissive hosting — use `python -m http.server` for quick local testing.

---

## Notes

- The `connect.json` file is not committed to this repo as it contains BRACU's internal timetable data. You must supply your own copy.
- The app does not send any data anywhere. CSV parsing and all computation happens entirely in the browser.
- The Google Form collects emails by default (`setCollectEmail(true)`). Remove that line from the Apps Script if you don't need it.

---

## License

Licensed under the Apache License, Version 2.0.
