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

## Step 6 — Signal Dashboard Artifact

The Signal dashboard is a Live Artifact that Claude rebuilds with fresh data
on each scan. Claude reads signals.jsonl and injects the data as JSON into
the artifact — the artifact does not read files itself.

### How to build it

1. Read `~/Documents/Signal/signals.jsonl` — parse every line as a JSON object
2. Read `~/Documents/Signal/run-status.md` — extract last run timestamp and counts
3. Build a Live Artifact (HTML + embedded JavaScript) with the signal data
   injected as a const JSON array at the top of the script
4. The artifact renders from that const — no file reads, no fetch calls

### When to rebuild

Rebuild the artifact on every scan run after writing to signals.jsonl.
This is the correct pattern — Claude reads the file, Claude injects the data,
the artifact renders it. The "live" part is that Claude always reads fresh
data before building.

### Artifact spec

```html
<script>
const SIGNALS = [/* Claude injects the full parsed signals array here */];
const LAST_RUN = "/* Claude injects run timestamp here */";
const RUN_STATS = {/* new signals count, sources checked */};
</script>
```

**Header**

- Title: "Signal"
- Last run timestamp from `LAST_RUN`
- Total signal count

**Filter Bar**

- Run: dropdown of distinct `run_date` values from `SIGNALS`, newest first, format "May 10 · 8:02am". Default: most recent. "All runs" at top. When a specific run is selected, only show signals where `run_date` matches.
- Category: All | technique | gap | pattern | actionable | insight
- Score: 80+ | 60+ | All

**Signal Cards** — sorted by score descending, filtered by active filters

- Score badge — 80+ = teal `#2A6B6B`, 60–79 = amber `#B45309`, 40–59 = grey `#6B7280`
- Title as a clickable link (opens in new tab)
- Category tag
- `connects_to` (italic, smaller)
- `reason`
- Action badge + Decay badge
- Source name + date

**Empty State**

- No signals in data: "No signals yet. Type /scan to run your first scan."
- Filters active but no matches: "No signals match the current filters."

**No refresh button** — Claude rebuilds the artifact on every scan, so there is no stale data problem. A refresh button adds complexity without benefit in this model.

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
