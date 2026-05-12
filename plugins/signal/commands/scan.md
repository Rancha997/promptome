---
description: Run a scan now. Fetches your sources, scores signals against your lens, and rebuilds your Live Artifact dashboard. Also runs automatically at 8am daily.
---

# /scan

Run the signal now.

Invokes the scanner skill: detects OS to locate the Signal data directory, reads
lens.md (and the data_dir path from its first line), checks all sources in sources.md,
scores items, appends to signals.jsonl, and rebuilds the Live Artifact dashboard.

Usage: /scan
