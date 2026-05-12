---
description: Scans the web for opportunities scored against your personal lens. Deduplicates, appends to signals.jsonl, and rebuilds the Signal dashboard. Invoke with /scan.
---

# Signal Scanner

You are running the daily Signal scan.

## File paths

All files live in `~/Documents/Signal/`:
- `lens.md` — your personal scoring lens (written by lens builder)
- `sources.md` — list of sources and queries to check
- `signals.jsonl` — append-only scored signals, one JSON object per line
- `run-status.md` — overwritten each run, system health summary

## Steps

### 1. Read context

- Read `~/Documents/Signal/lens.md` — this is the scoring filter. Everything is scored against it.
- Read `~/Documents/Signal/sources.md` — the list of sources, queries, and tool routing.
- Read `~/Documents/Signal/signals.jsonl` — extract the set of existing IDs (sha1 of url, first 12 chars). Use this set to skip duplicates before spending tokens on scoring.

If any of these files don't exist, stop and tell the user:
"Signal isn't set up yet. Run /setup-signal to configure your lens and sources."

### 2. Fetch each source

Follow the tool routing in sources.md exactly. Do not improvise tool choice.

Default routing rules:
- **Reddit:** Use web_search with site:reddit.com — search for posts from the last 24 hours
- **Hacker News:** Use web_fetch on `https://hn.algolia.com/api/v1/search?tags=story&numericFilters=created_at_i>{epoch_24h_ago}` — compute epoch threshold at run time (current time minus 86400 seconds)
- **Specific URLs:** Use web_fetch directly
- **Everything else:** Use web_search

If a source fails, log it in run-status.md and continue. Never abort the whole run for one failed source.

### 3. Build candidate list

For each fetched item:
- Extract: title, url, source, published date if available
- Compute id: sha1 of url, first 12 chars
- **Skip immediately if id is already in the existing IDs set from step 1.** Do not score duplicates.

### 4. Score every remaining candidate

Score against lens.md using this exact schema:

```json
{
  "id": "<sha1 of url, first 12 chars>",
  "scanned_at": "<ISO 8601 UTC>",
  "run_date": "<YYYY-MM-DD>",
  "title": "...",
  "url": "...",
  "source": "...",
  "score": 0-100,
  "category": "technique|gap|pattern|actionable|insight",
  "connects_to": "specific project, goal, or context from lens.md",
  "reason": "1-2 sentences. concrete. no fluff. why this matters to this specific user.",
  "action_suggested": "read|save|build|reply|ignore",
  "decay": "hours|days|weeks"
}
```

### 5. Discard below 40

Do not write, list, or mention anything scoring below 40. Discard silently.

### 6. Append to signals.jsonl

For every item scoring 40+, append one JSON line to `~/Documents/Signal/signals.jsonl`.

- One item per line
- No array brackets
- No commas between lines
- Append only — never rewrite or reorder existing content

### 7. Write run-status.md

Overwrite `~/Documents/Signal/run-status.md` with:

```
# Run: YYYY-MM-DD HH:MM

Signals: {N} new ({K} duplicates skipped)
Sources: {ok} ok, {failed} failed

## Source status
{one line per source: name + ✓ or ✗ + brief note if failed}

## Notes
{anything unusual — scoring edge cases, sources returning nothing, sources that failed, profile gaps noticed}
```

Keep under 1KB. System health glance, not a journal.

### 8. Rebuild the Signal dashboard

Read the full `~/Documents/Signal/signals.jsonl` — parse every line as a JSON object.
Read `~/Documents/Signal/run-status.md` — extract run timestamp and counts.
Build the Signal Live Artifact with all signals injected as:

```javascript
const SIGNALS = [ /* full parsed array */ ];
const LAST_RUN = "/* run timestamp */";
```

Use the full artifact HTML/CSS/JS template from the artifact spec below.

This rebuilds on every scan run — Claude reads the file and injects the data.
The artifact renders from the injected const — no file reads, no fetch calls.

#### JSON serialization rules

When injecting SIGNALS into the artifact:
- Parse each line of signals.jsonl as JSON
- Serialize the full array with `JSON.stringify()` — do not manually construct the array
- The injected value must be valid JSON — all special characters (em dashes, arrows, quotes, unicode) must be properly escaped
- If a field contains HTML special characters (`< > & " '`), they must be escaped in the artifact HTML context but remain valid in the JSON

The const injection must look exactly like:
```javascript
const SIGNALS = [{"id":"abc","title":"...","reason":"..."},...];
```

Not like:
```javascript
const SIGNALS = [{id: "abc", title: '...'}];  // ← wrong, not valid JSON
```

## Scoring rules

- Score 80+ only for things the user would genuinely regret missing. Most items should score 40-60. Inflated scores destroy the system.
- `connects_to` must name something specific from lens.md. "Connects to AI" fails. If you can't name a specific connection, score below 40.
- Reasons stay concrete. No "this is interesting because AI is evolving fast." Say what specifically and how it connects.
- Skip categories the lens says to ignore. Don't list them. Don't apologize.
- Time-sensitive items decay. A job posting from yesterday scores higher than one from last week.
- JSONL is the source of truth. Never let it disagree with run-status.
- Fail loud not silent. If a source fails, log the specific tool and error in run-status. Silent fallbacks degrade signal quality without the user noticing.
- No prose digest output. The artifact handles rendering. Do not generate "Top signals" blocks, "Worth a look" lists, or summary paragraphs. That is the artifact's job.

## Artifact spec

```html
<!DOCTYPE html>
<html>
<head>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Fraunces:ital,wght@1,700;1,900&family=DM+Sans:wght@300;400;500;600&family=DM+Mono:wght@400;500&display=swap');

  * { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    background: #F5F0E8;
    font-family: 'DM Sans', sans-serif;
    color: #1A1A1A;
    min-height: 100vh;
    padding: 0;
  }

  /* Header */
  .header {
    background: rgba(245,240,232,0.94);
    backdrop-filter: blur(18px);
    border-bottom: 1px solid #E5DDD0;
    padding: 16px 32px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    position: sticky;
    top: 0;
    z-index: 100;
  }

  .header-left { display: flex; align-items: baseline; gap: 12px; }

  .header-title {
    font-family: 'Fraunces', serif;
    font-style: italic;
    font-weight: 900;
    font-size: 22px;
    color: #1A1A1A;
    letter-spacing: -0.02em;
  }

  .header-meta {
    font-size: 11px;
    color: #999;
    font-weight: 400;
  }

  .header-stats {
    display: flex;
    gap: 20px;
    align-items: center;
  }

  .stat {
    text-align: right;
  }

  .stat-num {
    font-family: 'DM Mono', monospace;
    font-size: 18px;
    font-weight: 500;
    color: #2A6B6B;
    line-height: 1;
  }

  .stat-label {
    font-size: 9px;
    text-transform: uppercase;
    letter-spacing: 0.08em;
    color: #999;
    margin-top: 2px;
  }

  /* Filters */
  .filters {
    padding: 14px 32px;
    display: flex;
    gap: 10px;
    align-items: center;
    border-bottom: 1px solid #E5DDD0;
    flex-wrap: wrap;
    background: #F5F0E8;
  }

  .filter-label {
    font-size: 10px;
    font-weight: 700;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    color: #999;
    margin-right: 4px;
  }

  select {
    appearance: none;
    background: white;
    border: 1.5px solid #E5DDD0;
    border-radius: 100px;
    padding: 6px 28px 6px 14px;
    font-family: 'DM Sans', sans-serif;
    font-size: 12px;
    font-weight: 500;
    color: #1A1A1A;
    cursor: pointer;
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='10' height='6' viewBox='0 0 10 6'%3E%3Cpath d='M1 1l4 4 4-4' stroke='%23666' stroke-width='1.5' fill='none'/%3E%3C/svg%3E");
    background-repeat: no-repeat;
    background-position: right 10px center;
    transition: border-color 0.12s;
  }

  select:focus { outline: none; border-color: #2A6B6B; }

  .filter-divider {
    width: 1px;
    height: 20px;
    background: #E5DDD0;
    margin: 0 4px;
  }

  /* Grid */
  .grid {
    padding: 24px 32px;
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(340px, 1fr));
    gap: 14px;
  }

  /* Signal card */
  .card {
    background: white;
    border: 1.5px solid #E5DDD0;
    border-radius: 12px;
    padding: 18px;
    display: flex;
    flex-direction: column;
    gap: 10px;
    transition: box-shadow 0.14s, transform 0.12s;
    cursor: pointer;
  }

  .card:hover {
    box-shadow: 0 4px 20px rgba(0,0,0,0.07);
    transform: translateY(-1px);
  }

  .card-top {
    display: flex;
    align-items: flex-start;
    justify-content: space-between;
    gap: 12px;
  }

  .score-badge {
    font-family: 'DM Mono', monospace;
    font-size: 20px;
    font-weight: 500;
    line-height: 1;
    flex-shrink: 0;
    width: 44px;
    height: 44px;
    border-radius: 10px;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .score-high { background: #E8F0F0; color: #2A6B6B; }
  .score-mid  { background: #FDF6E3; color: #B8860B; }
  .score-low  { background: #F5F5F5; color: #888888; }

  .card-title {
    font-family: 'Fraunces', serif;
    font-style: italic;
    font-weight: 700;
    font-size: 15px;
    line-height: 1.3;
    color: #1A1A1A;
    letter-spacing: -0.01em;
    text-decoration: none;
    flex: 1;
  }

  .card-title:hover { color: #2A6B6B; }

  .card-reason {
    font-size: 12.5px;
    color: #444;
    line-height: 1.6;
    font-weight: 300;
  }

  .card-connects {
    font-size: 11px;
    color: #2A6B6B;
    font-weight: 500;
    font-family: 'DM Mono', monospace;
  }

  .card-footer {
    display: flex;
    align-items: center;
    gap: 6px;
    flex-wrap: wrap;
    margin-top: 2px;
  }

  .pill {
    padding: 2px 9px;
    border-radius: 100px;
    font-size: 10px;
    font-weight: 600;
    letter-spacing: 0.03em;
    background: #F0EBE3;
    color: #666;
    text-transform: uppercase;
  }

  .pill-action { background: #E8F0F0; color: #2A6B6B; }
  .pill-decay  { background: #FDF6E3; color: #B8860B; }

  .card-source {
    font-size: 10px;
    color: #999;
    margin-left: auto;
    font-family: 'DM Mono', monospace;
  }

  /* Empty state */
  .empty {
    grid-column: 1/-1;
    text-align: center;
    padding: 80px 40px;
  }

  .empty-title {
    font-family: 'Fraunces', serif;
    font-style: italic;
    font-size: 22px;
    font-weight: 700;
    color: #1A1A1A;
    margin-bottom: 8px;
  }

  .empty-sub {
    font-size: 13px;
    color: #999;
    font-weight: 300;
  }

  /* Count bar */
  .count-bar {
    padding: 8px 32px;
    font-size: 11px;
    color: #999;
    border-bottom: 1px solid #E5DDD0;
  }
</style>
</head>
<body>

<div class="header">
  <div class="header-left">
    <span class="header-title">Signal</span>
    <span class="header-meta" id="run-time"></span>
  </div>
  <div class="header-stats">
    <div class="stat">
      <div class="stat-num" id="total-count">0</div>
      <div class="stat-label">Total signals</div>
    </div>
    <div class="stat">
      <div class="stat-num" id="shown-count">0</div>
      <div class="stat-label">Showing</div>
    </div>
  </div>
</div>

<div class="filters">
  <span class="filter-label">Run</span>
  <select id="run-filter"></select>
  <div class="filter-divider"></div>
  <span class="filter-label">Category</span>
  <select id="cat-filter">
    <option value="all">All categories</option>
    <option value="technique">Technique</option>
    <option value="gap">Gap</option>
    <option value="pattern">Pattern</option>
    <option value="actionable">Actionable</option>
    <option value="insight">Insight</option>
  </select>
  <div class="filter-divider"></div>
  <span class="filter-label">Score</span>
  <select id="score-filter">
    <option value="all">All scores</option>
    <option value="80">80+ only</option>
    <option value="60">60+ only</option>
  </select>
</div>

<div class="count-bar" id="count-bar"></div>
<div class="grid" id="grid"></div>

<script>
const SIGNALS = [/* CLAUDE INJECTS SIGNALS ARRAY HERE */];
const LAST_RUN = "/* CLAUDE INJECTS TIMESTAMP HERE */";

function scoreClass(s) {
  return s >= 80 ? 'score-high' : s >= 60 ? 'score-mid' : 'score-low';
}

function fmt(dateStr) {
  const d = new Date(dateStr);
  return d.toLocaleDateString('en-US', { month: 'short', day: 'numeric' }) +
    ' · ' + d.toLocaleTimeString('en-US', { hour: 'numeric', minute: '2-digit' });
}

function render() {
  const run = document.getElementById('run-filter').value;
  const cat = document.getElementById('cat-filter').value;
  const scoreMin = parseInt(document.getElementById('score-filter').value) || 0;

  const filtered = SIGNALS.filter(s => {
    if (run !== 'all' && s.run_date !== run && s.scanned_at?.slice(0,10) !== run) return false;
    if (cat !== 'all' && s.category !== cat) return false;
    if (s.score < scoreMin) return false;
    return true;
  }).sort((a,b) => b.score - a.score);

  document.getElementById('shown-count').textContent = filtered.length;
  document.getElementById('count-bar').textContent =
    filtered.length === SIGNALS.length
      ? `${SIGNALS.length} signals`
      : `Showing ${filtered.length} of ${SIGNALS.length} signals`;

  const grid = document.getElementById('grid');
  if (!filtered.length) {
    grid.innerHTML = `<div class="empty">
      <div class="empty-title">No signals match these filters.</div>
      <div class="empty-sub">Try adjusting the run, category, or score filter.</div>
    </div>`;
    return;
  }

  grid.innerHTML = filtered.map(s => `
    <div class="card" onclick="window.open('${s.url}','_blank')">
      <div class="card-top">
        <div class="score-badge ${scoreClass(s.score)}">${s.score}</div>
        <a class="card-title" href="${s.url}" target="_blank" onclick="event.stopPropagation()">${s.title}</a>
      </div>
      ${s.connects_to ? `<div class="card-connects">→ ${s.connects_to}</div>` : ''}
      <div class="card-reason">${s.reason}</div>
      <div class="card-footer">
        ${s.action_suggested ? `<span class="pill pill-action">${s.action_suggested}</span>` : ''}
        ${s.decay ? `<span class="pill pill-decay">${s.decay}</span>` : ''}
        ${s.category ? `<span class="pill">${s.category}</span>` : ''}
        <span class="card-source">${s.source || ''}</span>
      </div>
    </div>
  `).join('');
}

// Populate run filter
const runs = [...new Set(SIGNALS.map(s => s.run_date || s.scanned_at?.slice(0,10)).filter(Boolean))].sort().reverse();
const runSel = document.getElementById('run-filter');
runSel.innerHTML = '<option value="all">All runs</option>' +
  runs.map(r => `<option value="${r}" ${r === runs[0] ? 'selected' : ''}>${fmt(r)}</option>`).join('');

document.getElementById('total-count').textContent = SIGNALS.length;
document.getElementById('run-time').textContent = LAST_RUN ? `Last scan: ${LAST_RUN}` : '';

['run-filter','cat-filter','score-filter'].forEach(id =>
  document.getElementById(id).addEventListener('change', render));

render();
</script>
</body>
</html>
```
