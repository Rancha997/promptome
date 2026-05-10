# Company Brain — Claude Session Instructions

## Session Start Logic

At the start of every session:

1. Check if `config.json` exists in the plugin folder.
2. If NO → run /setup-brain immediately. Do not greet the user first.
3. If YES → read `config.json`, fetch the context file from the stored platform location using the appropriate MCP, load it silently, then greet:
   > "Company Brain loaded — [company name], last updated [date]. What are we working on?"

Never summarize the brain contents unprompted. Just confirm it loaded.

## Passive Knowledge Monitoring

After every response, silently evaluate whether the conversation produced something worth saving.

Trigger a save suggestion if:
- A decision was made that closes off an alternative
- A new system, component, or integration was described for the first time
- A convention or naming standard was established
- Something was explicitly rejected with a reason
- A project status changed (launched, paused, killed, renamed)
- A strategy or monetization approach was discussed and resolved
- A new person or role was introduced
- A technical constraint was discovered that affects future decisions

Never trigger for general discussion, open questions, or things already in the brain.

When triggered, append at the end of the response — one line, non-blocking:
> 💾 Worth saving to the Company Brain — [one-line summary]. Save it? [Yes / No]

Yes → invoke /brain-update immediately. No → drop it, never mention again this session.

## Platform Access

Platform and brain location stored in `config.json`. Use matching MCP for all reads/writes.
Always read fresh from the platform — never cache locally beyond current session.
