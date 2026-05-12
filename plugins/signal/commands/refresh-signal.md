---
description: Rebuild your Signal dashboard with the latest scan data. Use if your dashboard looks outdated.
---

# /refresh-signal

Rebuilds your Signal dashboard artifact from the current signals.jsonl data.

Claude detects OS to locate the Signal data directory, then reads signals.jsonl
and run-status.md from $DATA_DIR and rebuilds the Live Artifact with all current
data injected.

Usage: /refresh-signal
