---
description: Runs on first install. Interviews the user to build their personal opportunity lens, configure sources, and set up the daily scheduled scan.
---

# Signal Lens Builder

You are setting up Signal for the first time.

## What you are building

Two files that the scanner reads on every run:
- `~/Documents/Signal/lens.md` — the personal scoring lens
- `~/Documents/Signal/sources.md` — the sources and queries to monitor

## Interview

Ask these questions one at a time. Wait for each answer before asking the next.
Ask follow-up questions if answers are vague — specific answers produce better signals.

1. "What are you working on right now? Describe your active projects briefly."
2. "What kinds of opportunities matter most to you — new tools, techniques, market gaps, competitors, job signals, funding news, community discussions?"
3. "Which specific domains or topics should the scanner focus on? Be specific — e.g. 'Claude MCP ecosystem', 'n8n community', 'indie SaaS founders', 'AI infrastructure'."
4. "What sources do you already follow? And where do you wish you had more coverage?"
5. "What should the scanner ignore completely? Topics, content types, or noise you always skip."
6. "What does a perfect signal look like to you? Give me one concrete example of something you'd be glad you caught."

## Writing lens.md

After the interview, create `~/Documents/Signal/` if it doesn't exist, then write `~/Documents/Signal/lens.md` with these sections:

```markdown
# Signal Lens — [User's name or context]
Last updated: YYYY-MM-DD

## Identity & Context
[Who this person is and what they're building — 3-5 bullets]

## Active Projects
[List of active projects — one line each, what they are and why they matter for scoring]

## Opportunity Categories
[Ranked list of what matters most — specific, not generic]

## Sources to Monitor
[List of sources from the interview]

## What to Ignore
[Explicit ignore list — topics, content types, noise]

## Scoring Guidance
[Specific guidance on what scores high vs low for this person]

## Example Perfect Signal
[The concrete example the user gave — this is the benchmark]
```

## Writing sources.md

Write `~/Documents/Signal/sources.md` with concrete sources derived from the interview:

```markdown
# Signal Sources
Last updated: YYYY-MM-DD

## Daily sources
[Each source with: name, what to search/fetch, tool to use]

## Weekly sources
[Slower-moving sources checked less frequently]

## Tool routing
- Reddit sources: web_search with site:reddit.com
- HN: web_fetch https://hn.algolia.com/api/v1/search?tags=story&numericFilters=created_at_i>{epoch}
- Specific URLs: web_fetch directly
- Everything else: web_search
```

## After writing both files

Confirm:
"Your lens is configured. Here's what Signal will look for: [2-3 sentence summary of the lens]"

Then immediately:

1. Run `/schedule` to create the daily 8am task with full scanner instructions
2. Run `/scan` once to fetch sources, score the first batch, and build the dashboard
3. Confirm: "Signal is running. Your first scan is complete — here's your dashboard."

## Creating the scheduled task

When `/schedule` runs, provide:

**Name:** Signal daily scan
**Schedule:** Daily at 8:00 AM
**Instructions:**
```
Run the Signal daily scanner.

1. Read ~/Documents/Signal/lens.md for scoring criteria
2. Read ~/Documents/Signal/sources.md for sources to check
3. Read ~/Documents/Signal/signals.jsonl — extract existing IDs for deduplication
4. Fetch each source using the tool routing in sources.md
5. Score each candidate 0-100 against lens.md — discard below 40
6. Append qualifying signals to ~/Documents/Signal/signals.jsonl (one JSON line each, append only)
7. Write ~/Documents/Signal/run-status.md with scan summary
8. Read full signals.jsonl, parse all lines, rebuild Signal dashboard Live Artifact with all signals injected as const SIGNALS = [...] in the HTML
```
