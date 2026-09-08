# Goal Tracker

A single-file, zero-dependency app for tracking your yearly employee goals — impact, success measures, milestones, and dated progress updates.

## Usage

1. Open `index.html` in **Chrome or Edge** (double-click it, or drag it into the browser).
2. Click **Choose data folder…** and pick this folder — the one containing `index.html`.
   A `goals.json` is loaded from there, or created if it doesn't exist yet.
3. That's it — every change autosaves to `goals.json`. On your next visit, click **Reconnect** once.

Your data is plain JSON you can inspect, back up, or edit by hand. Use **File → Export a copy…** for backups and **Import JSON…** to restore one. Reorder goals with the ▲▼ arrows on each card.

> `goals.json` is git-ignored so personal goals don't end up in the repo — remove that line from `.gitignore` if you *want* to version them.
