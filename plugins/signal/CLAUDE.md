# Signal — Claude Session Instructions

## Session Start Logic

At the start of every session:

1. Check if `~/Documents/Signal/lens.md` exists.
2. If it does **NOT** exist → the user hasn't run setup yet. Greet them briefly and immediately start the lens-builder interview. Do not wait for them to ask.
3. If it **DOES** exist → greet the user briefly and confirm Signal is ready:
   > "Signal is ready. Type /scan to run a scan, or /lens to update your focus."

Never scan without a lens. Always check first.

---

## Available Commands

- `/setup-signal` — full onboarding: lens builder interview + daily schedule + first scan. Run once after install.
- `/scan` — run a scan now. Also runs automatically at 8am daily after setup.
- `/lens` — update your lens. Tell Claude what to change for a quick edit, or run the full interview for a strategic reset.
