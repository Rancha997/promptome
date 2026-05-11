---
description: Handles first install and joining an existing brain. Detects connected platforms, searches for existing brain, runs interview if new.
---

# Skill: brain-builder

Handles first-time installation and joining an existing Company Brain. Run this when `config.json` does not exist.

## Auto-invoke

At the start of every session:

1. Check if `config.json` exists in the plugin folder.
2. If **NO** → run this skill immediately. Do not greet the user first.
3. If **YES** → read `config.json`, fetch the context file from the stored platform location using the appropriate MCP, load it silently into context, then greet the user briefly:
   > "Company Brain loaded — [company name], last updated [date]. What are we working on?"

Never summarize the brain contents unprompted. Just confirm it loaded and move on.

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

## Platform Access

- Platform type and brain location are stored in `config.json`.
- Use the matching MCP for all reads and writes:
  - Notion → Notion MCP
  - Google Drive → Google Drive MCP
  - Confluence → Confluence MCP
- Always read fresh from the platform — never cache locally beyond the current session.
- `config.json` is the only local file. All brain content lives on the platform.

---

## Step 1: Detect Connected Platform MCPs

Check which of the following platform MCPs are currently connected in Cowork:

- Notion MCP
- Google Drive MCP
- Confluence MCP

**If none are connected:**

> "To use Company Brain, you need to connect at least one doc platform. Go to Claude Settings → Connections and connect Notion, Google Drive, or Confluence. Then come back and I'll set up your brain."

Stop here. Do not proceed until at least one platform MCP is connected.

**If exactly one is connected:** use it. Ask no questions about platform.

**If two or more are connected:** ask once:

> "I can see you have [list each connected platform] connected. Which one does your team use for the Company Brain?"

Wait for the user's answer. Use that platform for all steps that follow.

---

## Step 2: Search for an Existing Brain

Search the chosen platform for an existing Company Brain document:

- **Notion** — search for a page titled "Company Brain — Context" or with tag/property `company-brain`
- **Google Drive** — search for a file named "Company Brain — Context"
- **Confluence** — search for a page with label `company-brain`

**If exactly one result is found:**

> "Found it — Company Brain for [company name], last updated [date][, created by [author] if available]. Connect to this one?"

If the user confirms → save to `config.json` → load the brain → greet the user → done. Skip the interview entirely.

**If multiple results are found:** show a numbered list with title and last-modified date for each:

> "I found [N] Company Brain documents. Which one is yours?"

User picks one → save to `config.json` → load → done.

**If nothing is found:**

> "No existing Company Brain found in your [platform]. I'll create one now — this takes about 5 minutes."

Proceed to Step 3.

---

## Step 3: New Brain Interview

Ask questions one at a time. Wait for each answer before asking the next. Ask a follow-up if an answer is vague or incomplete. Do not rush.

1. "What's your company or project called, and what does it do in one sentence?"
2. "Who is it for — who's the target user or customer?"
3. "What's actually live and working right now? What's in progress? What's paused or abandoned?"
4. "Walk me through your tech stack — what are you running and how does it connect?"
5. "What are the most important decisions you've made so far — product, technical, or strategic? What did you explicitly decide NOT to do?"
6. "How are you thinking about revenue and growth right now?"
7. "Who's on the team and how do you work together?"
8. "What are you still figuring out — what's unsettled or actively debated?"

---

## Step 4: Build the Brain

After the interview, create the following Notion folder structure. Follow the creation sequence exactly.

### Folder Structure

```
🧠 [Company Name] Brain/          ← root page
├── 📋 Context                    ← agent-optimized, loads every session
├── 🏢 Identity & Strategy/
│   ├── Who We Are & Thesis
│   ├── Strategic Bets
│   └── Rejected Directions
├── 🚀 Projects/
│   └── [one subfolder per project with]:
│       ├── Overview & Status
│       └── Decisions
├── ⚙️ Infrastructure/
│   └── [one page per system]
├── 📋 Decisions Log              ← flat, all decisions newest first
├── 👥 People & Conventions/
│   ├── Team
│   ├── Publishing Standards
│   ├── Content Voice
│   └── Outreach Rules
└── ❓ Open Questions
```

### Creation Sequence

Must follow this order:

1. Create the root page **[Company Name] Brain**
2. Create all section folders as child pages of root
3. Create all leaf pages as children of their section
4. For each project mentioned in the interview, create a project subfolder under **Projects/** with **Overview & Status** and **Decisions** child pages
5. For each system mentioned in the interview, create one page under **Infrastructure/**
6. **Last** — create the **Context** page and populate the Quick Index with the actual Notion URLs of every page just created

The Context page is created last so it can contain real URLs for all other pages. Use the Notion MCP to get the URL of each created page and write it into the Quick Index.

### Context Page Format

Populate the Context page with this exact structure:

```markdown
# [Company Name] Brain — Context
Last updated: YYYY-MM-DD

## Quick Index
[one line per section + direct Notion URL to each page]
- Identity & Strategy: [url]
- Projects: [url]
  - [Project Name]: [url] (one line per project)
- Infrastructure: [url]
  - [System Name]: [url] (one line per system)
- Decisions Log: [url]
- People & Conventions: [url]
- Open Questions: [url]

## Identity (compressed)
[5-7 bullets — what we build, who for, operating principles, what we are NOT]

## Active Projects (compressed)
[one line per project: name | status | next milestone]

## Critical / Urgent
[anything time-sensitive: deadlines, broken systems, migration alerts]

## Last 5 Decisions
[most recent 5 decisions inline — date | decision | why]
```

Be dense and specific — this file is for Claude to read, not humans. Use bullet points, not prose. Leave no section empty; use "—" if genuinely nothing to add.

---

## Step 5: Save Config and Confirm

Write `config.json` to the plugin folder:

```json
{
  "platform": "notion",
  "brain_id": "[root page id]",
  "brain_url": "[root page url]",
  "context_id": "[context page id]",
  "context_url": "[context page url]",
  "company_name": "[company name]",
  "installed_by": "[user name from interview]",
  "installed_at": "YYYY-MM-DD",
  "teammates": []
}
```

`config.json` stores both `brain_id` (root) and `context_id` (the lean page Claude loads every session). Claude fetches `context_url` on session start, not the root.

Confirm to the user:

> "Company Brain is ready. Claude now has full context about [company name] in every session. Share the brain URL with your team: [brain_url]. When a teammate installs this plugin, I'll find the brain automatically."
