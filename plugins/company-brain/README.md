# Company Brain

Company Brain gives Claude persistent, shared knowledge about your company. Every conversation starts with full context — your stack, active projects, key decisions, team conventions, and open questions — loaded silently before you type a word. No more re-explaining your architecture. No more relitigating decisions you made three months ago.

The brain lives in your team's doc platform (Notion, Google Drive, or Confluence), not on any one machine. One person creates it, everyone else connects to the same document. Claude reads it fresh each session and writes back to it whenever something worth saving happens.

---

## Requirements

- Claude Desktop (Mac or Windows)
- Pro or Max plan
- Cowork enabled
- At least one of the following connected in Claude Settings → Connections:
  - Notion
  - Google Drive
  - Confluence

---

## Installation

1. Open Cowork in Claude Desktop
2. Go to **Plugins → Browse** or install from [promptome.io](https://promptome.io)
3. Find **Company Brain** and click **Install**
4. Start a new conversation — Claude will run setup automatically

---

## First Run

When you install the plugin and start a new conversation, Claude will:

1. Detect which platform MCPs you have connected
2. Search your platform for an existing Company Brain document
3. If found — connect to it immediately, no interview needed
4. If not found — run a ~5 minute interview to build the brain from scratch

The interview covers: what you build, who for, your stack, active projects, key decisions, strategy, team, and open questions. Claude asks one question at a time and follows up if answers are vague.

After the interview, Claude creates a structured context file and linked source docs in your platform, then saves a local `config.json` pointer so future sessions load instantly.

---

## Joining an Existing Brain

Just install the plugin. When you start a new conversation, Claude searches your platform automatically. If your team's brain is there, Claude connects to it and skips the interview entirely.

You don't need the brain URL in advance — Claude finds it. If there are multiple Company Brain documents (e.g., from different teams), Claude will show the list and ask which one is yours.

---

## Daily Use

At the start of every session, Claude loads the brain silently and gives you a one-line confirmation:

> "Company Brain loaded — Acme Corp, last updated 2026-05-01. What are we working on?"

Then Claude monitors passively. When it detects something worth saving — a decision, a new system, a rejected approach, a project status change — it surfaces a single non-blocking suggestion at the end of its response:

> 💾 Worth saving to the Company Brain — decided to use Postgres over MongoDB for the new service. Save it? [Yes / No]

Say **Yes** and Claude saves it immediately. Say **No** and it drops it, never asks again in that session.

---

## Commands

| Command | What it does |
|---|---|
| `/brain-update` | Manually save something from the current conversation |
| `/brain-sync` | Rebuild the context file from all source docs (use after heavy edits) |
| `/brain-status` | Report what's in the brain, when it was last updated, and what looks thin |

You can also just say "save this to the brain" in plain language — no slash command required.

---

## Team Setup

1. **One person installs** Company Brain and goes through the interview to create the brain
2. **Share the brain URL** from your platform (Notion page, Drive file, or Confluence page) with your team
3. **Teammates install the plugin** — Claude finds the shared brain automatically on first run

Every teammate reads from and writes to the same brain. Updates from one session are available to everyone in the next. Teammates are tracked in `config.json` as they connect.

---

## Pricing

Single purchase at [promptome.io](https://promptome.io). One license per person — the brain itself is shared for free across your whole team.
