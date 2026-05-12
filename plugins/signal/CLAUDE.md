# Signal — Claude Session Instructions

## Session Start Logic

At the start of every session:

1. Detect OS and set DATA_DIR:
   - Run `uname -s`
   - `Darwin` → `$HOME/Library/Application Support/Claude/signal`
   - starts with `MINGW`, `MSYS`, or `Windows` → `$APPDATA/Claude/signal`
   - `Linux` → `$HOME/.config/Claude/signal`

2. Check if `$DATA_DIR/lens.md` exists.
3. If it does **NOT** exist → the user hasn't run setup yet. Greet them briefly and immediately start the lens-builder interview. Do not wait for them to ask.
4. If it **DOES** exist → read the `data_dir:` value from the first line to confirm the canonical path, greet the user briefly, and confirm Signal is ready:
   > "Signal is ready. Type /scan to run a scan, or /lens to update your focus."

Never scan without a lens. Always check first.

---

## Available Commands

- `/setup-signal` — full onboarding: lens builder interview + daily schedule + first scan. Run once after install.
- `/scan` — run a scan now. Also runs automatically at 8am daily after setup.
- `/lens` — update your lens. Tell Claude what to change for a quick edit, or run the full interview for a strategic reset.
