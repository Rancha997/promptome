# Company Brain — Claude Session Instructions

## Session Start Logic

At the start of every session:

1. Check if `config.json` exists in the plugin folder.
2. If **NO** → run the `brain-builder` skill immediately. Do not greet the user first.
3. If **YES** → read `config.json`, fetch the context file from the stored platform location using the appropriate MCP, load it silently into context, then greet the user briefly:
   > "Company Brain loaded — [company name], last updated [date]. What are we working on?"

Never summarize the brain contents unprompted. Just confirm it loaded and move on.

---

## Passive Knowledge Monitoring

After every response, silently evaluate whether the conversation just produced something worth saving to the brain.

**Trigger a save suggestion if any of these occurred:**
- A decision was made that closes off an alternative
- A new system, component, or integration was described for the first time
- A convention or naming standard was established
- Something was explicitly rejected with a reason
- A project status changed (launched, paused, killed, renamed)
- A strategy or monetization approach was discussed and resolved
- A new person or role was introduced
- A technical constraint was discovered that affects future decisions

**Never trigger for:**
- General discussion without a clear decision or outcome
- Open questions with no answer
- Information already present in the brain

**When triggered**, append this at the very end of the response — subtle, one line, non-blocking:

> 💾 Worth saving to the Company Brain — [one-line summary of what to save]. Save it? [Yes / No]

- If user says **Yes** → invoke the `brain-update` skill immediately.
- If user says **No** → drop it. Never mention it again in this session.

---

## Platform Access

- Platform type and brain location are stored in `config.json`.
- Use the matching MCP for all reads and writes:
  - Notion → Notion MCP
  - Google Drive → Google Drive MCP
  - Confluence → Confluence MCP
- Always read fresh from the platform — never cache locally beyond the current session.
- `config.json` is the only local file. All brain content lives on the platform.
