---
description: Handles first install and joining an existing brain. Detects connected platforms, searches for existing brain, runs interview if new.
---

# Skill: brain-builder

Handles first-time installation and joining an existing Company Brain. Run this when `config.json` does not exist.

## Auto-invoke

At the start of every session:

1. Search Notion for a page with "Brain — Context" in the title using the Notion MCP.
2. If found → fetch the page, load it silently into context, greet briefly:
   > "Company Brain loaded — [company name], last updated [date]. What are we working on?"
3. If not found → run this skill immediately. Do not greet the user first.

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

Use the Notion MCP for all reads and writes. Always read fresh from Notion — never cache locally beyond the current session.

---

## Step 1: Detect Connected Platform MCPs

Check whether the Notion MCP is connected in Cowork.

**If not connected:**

> "To use Company Brain, you need to connect Notion. Go to Claude Settings → Connections and connect Notion. Then come back and I'll set up your brain."

Stop here. Do not proceed until the Notion MCP is connected.

---

## Step 2: Search for an Existing Brain

Search Notion for a page with "Brain — Context" in the title.

**If exactly one result is found:**

> "Found it — Company Brain for [company name], last updated [date][, created by [author] if available]. Connect to this one?"

If the user confirms → fetch the page, load into context, greet the user → done. Skip the interview entirely.

**If multiple results are found:** show a numbered list with title and last-modified date for each:

> "I found [N] Company Brain pages. Which one is yours?"

User picks one → load → done.

**If nothing is found:**

> "No existing Company Brain found in your Notion. I'll create one now — this takes about 5 minutes."

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

Page titles are plain text. Icons are set via the `icon` property only — never include emoji in the title string.

```
[Company Name] Brain/             ← root page          icon: 🧠
├── Context                       ← loads every session icon: 📋
├── Identity & Strategy/                                icon: 🏢
│   ├── Who We Are & Thesis
│   ├── Strategic Bets
│   └── Rejected Directions
├── Projects/                                           icon: 🚀
│   └── [one subfolder per project with]:
│       ├── Overview & Status
│       └── Decisions
├── Infrastructure/                                     icon: ⚙️
│   └── [one page per system]
├── Decisions Log                 ← flat, newest first  icon: 📋
├── People & Conventions/                               icon: 👥
│   ├── Team
│   ├── Publishing Standards
│   ├── Content Voice
│   └── Outreach Rules
└── Open Questions                                      icon: ❓
```

### Creation Sequence

Must follow this order:

1. Create the root page — title: `[Company Name] Brain`, icon: `🧠`
2. Create section pages as children of root — use the icon and plain-text title from the table below
3. Create all leaf pages as children of their section — no icon required on leaf pages
4. For each project mentioned in the interview, create a project subfolder under **Projects** (icon: `🚀`) with **Overview & Status** and **Decisions** child pages
5. For each system mentioned in the interview, create one page under **Infrastructure** (icon: `⚙️`)
6. **Last** — create the **Context** page (icon: `📋`) and populate the Quick Index with the actual Notion URLs of every page just created

**Icon reference for section pages:**

| Title (plain text)     | Icon |
|------------------------|------|
| Context                | 📋   |
| Identity & Strategy    | 🏢   |
| Projects               | 🚀   |
| Infrastructure         | ⚙️   |
| Decisions Log          | 📋   |
| People & Conventions   | 👥   |
| Open Questions         | ❓   |

When calling the Notion MCP to create any page: pass the title as plain text and set the icon via the `icon` property. Never embed emoji in the title string.

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

## Step 5: Confirm

Confirm to the user:

> "Brain is ready. Claude will find it automatically at the start of every session by searching your Notion workspace. Share this page URL with teammates: [brain root page URL]. When they install the plugin, Claude will find the brain automatically."
