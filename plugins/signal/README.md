# Signal

A Cowork plugin that monitors the web daily for opportunities relevant to your work and surfaces them in a live dashboard — scored against your personal lens.

## What It Does

Signal watches your chosen sources every day, scores what it finds against your strategic priorities, and shows you only what matters. Instead of a firehose of links, you get a filtered, ranked list of signals — each scored 0–100, each connected to something specific you're working on. A score of 80 means "you'd regret missing this." Below 40 is discarded automatically. The result is a daily briefing you can triage in under a minute.

## Requirements

- Claude Desktop (Mac or Windows)
- Claude Pro or Max plan
- Cowork enabled in your Claude settings

No external services, API keys, or MCP servers required. Web search is native to Cowork.

## Installation

1. Download the `signal` plugin folder.
2. Open Claude Desktop and go to **Settings → Cowork → Plugins**.
3. Click **Install Plugin** and select the `signal` folder.
4. The plugin will appear in your plugin list as "Signal".

## First Run — Lens Builder

The first time you open a Cowork session with Signal active, Claude will automatically start the **lens-builder interview**. This is a short, conversational interview (6 questions) that establishes your personal lens:

- What you're working on right now
- What kinds of opportunities matter most to you
- Which domains and topics to watch
- Which sources you already follow (and where you want more coverage)
- What to ignore completely
- What a perfect signal looks like for you

After the interview, Claude writes your lens to `~/Documents/Signal/lens.md` and your source list to `~/Documents/Signal/sources.md`, schedules the daily 8am scan, and runs your first scan — all in one session. You can update your lens anytime with `/lens`.

## Daily Use

### Automatic (Recommended)

Run `/setup` once. It runs the lens-builder interview, schedules a daily scan at 8:00 AM, and runs your first scan — all in one session. After that, Claude will run `/scan` automatically each morning and the dashboard will be ready when you open it.

### Manual

Type `/scan` at any time to run a scan immediately. Claude will:

1. Read your lens and sources
2. Search the web for each source
3. Score and filter results
4. Append new signals to `~/Documents/Signal/signals.jsonl`
5. Rebuild the Live Artifact dashboard

## Commands

| Command  | What It Does                                                                                                   |
| -------- | -------------------------------------------------------------------------------------------------------------- |
| `/setup` | Full onboarding in one session — lens interview, schedules daily scan, runs first scan                         |
| `/scan`  | Run the scanner now — fetches sources, scores signals, rebuilds the dashboard                                  |
| `/lens`  | Full lens review — re-interviews you and rewrites `~/Documents/Signal/lens.md` and `~/Documents/Signal/sources.md` |

### Updating Your Lens

For **small adjustments**, just tell Claude in chat:

> "Focus less on n8n and more on Cowork plugins"
> "Add Hacker News Show HN posts to my sources"
> "Stop tracking fundraising news"

Claude will update `~/Documents/Signal/lens.md` or `~/Documents/Signal/sources.md` inline without a full review.

For **major strategic shifts** — new job, new project, new focus area — run `/lens` to do a full re-interview and rewrite.

## Where Data Lives

All data is stored in `~/Documents/Signal/` on your Mac. This folder is outside the plugin and never touched by git.

| File                               | Contents                                     |
| ---------------------------------- | -------------------------------------------- |
| `~/Documents/Signal/lens.md`       | Your personal lens — the scoring filter      |
| `~/Documents/Signal/sources.md`    | Sources and search queries checked each scan |
| `~/Documents/Signal/signals.jsonl` | Append-only log of all scored signals        |
| `~/Documents/Signal/run-status.md` | Status of the last scan run                  |

You can open and edit any of these files directly. The scanner reads them fresh on every run.

## Pricing

Signal is a single-purchase plugin available at [promptome.io](https://promptome.io). One payment, permanent access, all future updates included.
