---
description: Scans the web for opportunities scored against your personal lens. Appends to signals.jsonl and builds a Live Artifact dashboard.
---

# Scanner Skill

This is the core scan skill. It fetches sources, scores signals against the user's lens, deduplicates, appends to the log, and builds the Live Artifact dashboard.

Invoke this skill when the user runs `/scan` or when the daily scheduled task fires. Do not invoke it without a `~/Documents/Signal/lens.md` file — check first.

---

## Step 1: Read Context

Read all three files before doing anything else:

1. **`~/Documents/Signal/lens.md`** — this is the scoring filter. Internalize the identity, active projects, opportunity categories, what to ignore, scoring guidance, and the example perfect signal. Every scored item must connect to something specific named in this file.

2. **`~/Documents/Signal/sources.md`** — the list of sources and queries to check this run.

3. **`~/Documents/Signal/signals.jsonl`** — read every line and extract the `id` field from each JSON object. Build a deduplication set of all existing IDs. Any item whose ID is already in this set must be skipped — do not re-score, do not re-write.

---

## Step 2: Fetch Sources

For each source entry in `~/Documents/Signal/sources.md`:

- Run a web search using the specified query
- Focus on items published in the last 24 hours where possible
- Collect titles, URLs, and brief summaries

If a source fails (no results, search error, timeout): note it in the run status as a failed source. Do not abort the entire run — skip that source and continue.

Gather all candidate items across all sources before scoring.

---

## Step 3: Score Each Item

Score each candidate item against `~/Documents/Signal/lens.md` using the following schema exactly:

```json
{
  "id": "<sha1 hex of the item URL>",
  "date": "<YYYY-MM-DD of publication, or today if unknown>",
  "title": "<item title>",
  "url": "<full URL>",
  "source": "<source name from sources.md>",
  "score": 0,
  "category": "<technique|gap|pattern|actionable|insight>",
  "connects_to": "<specific project or asset named in ~/Documents/Signal/lens.md>",
  "reason": "<1-2 sentences, concrete, no generic AI hype>",
  "action": "<read|save|build|reply|ignore>",
  "decay": "<hours|days|weeks>",
  "run_date": "<today YYYY-MM-DD>"
}
```

### Scoring Rules — Enforce Strictly

- **Score 0–100.** Reserve 80+ for things the user would genuinely regret missing. A score of 90 should feel rare.
- **`connects_to` must be specific.** Name an exact project, asset, or goal from `~/Documents/Signal/lens.md`. If you cannot name something specific, the score must be below 40.
- **Discard below 40.** Do not write items scoring under 40 to the JSONL. They are noise.
- **Be ruthless.** An inflated score that trains the user to ignore the dashboard destroys the system. When in doubt, score lower.
- **No duplicates.** Skip any item whose `id` is already in the deduplication set. Do not write it.
- **`reason` must be concrete.** "This technique directly applies to the user's MCP plugin work" is good. "This is an interesting AI development" is not — discard it.
- **`category` definitions:**
  - `technique` — a method, tool, or approach the user could apply
  - `gap` — a market or ecosystem gap the user could fill
  - `pattern` — a trend or shift worth tracking
  - `actionable` — something requiring near-term action (reply, apply, submit)
  - `insight` — a framing or perspective that changes how the user thinks about something

### Compute the ID

The `id` field is the SHA-1 hex digest of the item's URL string. Use Python's `hashlib.sha1(url.encode()).hexdigest()` or equivalent. This must be deterministic — the same URL always produces the same ID.

---

## Step 4: Append to signals.jsonl

After scoring all items:

- Append each item that scored 40 or above as a single JSON line to `~/Documents/Signal/signals.jsonl`
- One JSON object per line — no pretty-printing, no trailing commas, no wrapping
- **Never overwrite** the file — always append new lines at the end
- Preserve all existing content exactly

---

## Step 5: Write Run Status

Overwrite `~/Documents/Signal/run-status.md` with a brief run summary:

```markdown
# Last Scan Status

**Last run:** YYYY-MM-DD HH:MM
**Sources checked:** N
**Items evaluated:** N
**Items scored ≥40:** N
**Items above 80:** N
**Duplicates skipped:** N
**Failed sources:** [list source names, or "none"]
```

---

## Step 6: Build the Live Artifact Dashboard

After writing the JSONL, build a Live Artifact (React component) that reads from `~/Documents/Signal/signals.jsonl` and renders the dashboard. The artifact should be a self-contained HTML/React component.

### Dashboard Spec

**Header**

- Title: "Signal"
- Last run date/time (from ~/Documents/Signal/run-status.md)
- Total signals collected (all time)
- Count of items above 80 this week

**Filter Bar**

- Time filter: All | Today | This week
- Run filter: dropdown showing distinct run dates from signals.jsonl, newest first, format "May 10 · 8:02am". Default: most recent run. "All runs" option at the top.
- Category filter: All | technique | gap | pattern | actionable | insight
- Score filter: 80+ | 60+ | All

When a specific run is selected, only show signals where run_date matches that run. The run dropdown should show the date and time of each unique run, derived from the run_date field in signals.jsonl combined with the timestamp in run-status.md.

**Signal Cards** — sorted by score descending
Each card shows:

- Score badge — color: 80+ = teal `#2A6B6B`, 60–79 = amber `#B45309`, 40–59 = grey `#6B7280`
- Title as a clickable link (opens in new tab)
- Category tag
- `connects_to` field (italic, smaller)
- `reason` (1–2 sentences)
- Action badge (read | save | build | reply | ignore)
- Decay badge (hours | days | weeks)
- Source name + date

**Refresh Button**

- Label: "↻ Refresh"
- Position: top-right of header, always visible
- On click: re-reads ~/Documents/Signal/signals.jsonl and ~/Documents/Signal/run-status.md from disk and redraws the entire dashboard
- Show "Last refreshed: X seconds ago" next to the button, updating every 30 seconds
- On load: always auto-refresh once on open so the user never sees stale data

The auto-refresh on load is the key fix — every time the artifact opens it immediately re-reads the JSONL before rendering, so the user always sees the latest scan data without having to manually click refresh.

**Empty State**

- When no signals match the current filters: "No signals yet. Type /scan to run your first scan."
- When filters are active but no matches: "No signals match the current filters."

**Design Principles**

- Clean, minimal — dark card background, readable contrast
- Score badge is the most prominent element on each card
- Cards are scannable at a glance — the user should be able to triage in 30 seconds
- No unnecessary chrome or decorative elements

---

## Error Handling

- If `~/Documents/Signal/lens.md` does not exist: stop immediately, tell the user to run the lens-builder first (or let onboarding run automatically per CLAUDE.md).
- If `~/Documents/Signal/sources.md` does not exist: stop and notify the user.
- If `~/Documents/Signal/signals.jsonl` does not exist yet: treat it as empty (no existing IDs). Create it on first write.
- If web search returns nothing for a source: log as failed source, continue.
- Never silently fail — always tell the user what happened.
