# Company Brain — Claude Session Instructions

## Session Start Logic

At the start of every session:

1. Search Notion for a page with "Brain — Context" in the title using the Notion MCP
2. If found → fetch the page, load it silently into context, greet:
   > "[Company name] Brain loaded — last updated [date]. What are we working on?"
3. If not found → run /setup-brain immediately. Do not greet first.

Never cache the brain URL locally. Always search fresh each session.
The search takes 1-2 seconds — this is acceptable.

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

Use the Notion MCP for all reads and writes. Always read fresh from Notion — never cache locally beyond the current session.
