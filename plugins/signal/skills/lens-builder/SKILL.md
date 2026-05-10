---
description: Runs on first install. Interviews the user to build their personal opportunity lens and configure sources.
---

# Lens Builder Skill

This skill runs once on first install. It interviews the user to understand their work, goals, and interests, then writes their personal lens — the scoring filter used by every future scan.

## Auto-invoke

At the start of every session:

1. Check if `~/Documents/Signal/lens.md` exists.
2. If it does NOT exist → greet the user, explain the plugin, and immediately invoke this skill to start the onboarding interview.
3. If it DOES exist → greet the user briefly and confirm they're ready to scan. Show `/scan` and `/lens` as available commands.

Do not scan without a lens. Do not write any signals without reading the lens first.

## When to Invoke

Invoke this skill automatically when `~/Documents/Signal/lens.md` does not exist. The user should not need to ask for it.

## Interview Process

Interview the user one question at a time, conversationally. Do not dump all questions at once. Listen carefully — follow up if an answer is vague. The goal is to build a precise, actionable lens, not just collect surface-level keywords.

Ask these questions in order, waiting for the user's response before asking the next:

1. "What are you working on right now? Describe your active projects briefly."

2. "What kind of opportunities matter most to you — new tools, market gaps, techniques, competitors, job signals, funding news?"

3. "What domains or topics should the scanner focus on? Be specific — e.g. 'Claude MCP ecosystem', 'n8n community', 'indie SaaS', 'AI infrastructure'."

4. "What sources do you already follow? And where do you wish you had more coverage?"

5. "What should the scanner ignore completely? Topics, types of content, or noise you always skip."

6. "What does a perfect signal look like to you — give me one example of the kind of thing you'd be glad you caught."

After each answer, briefly acknowledge what you heard and confirm your understanding if anything is ambiguous. Only proceed to the next question when you have a usable answer.

## Writing the Output

Create `~/Documents/Signal/` if it does not exist before writing any files.

Once you have answers to all six questions, do the following:

### Step 1: Write ~/Documents/Signal/lens.md

Write a structured personal lens document. Use this exact section structure:

```markdown
# Personal Opportunity Lens

## Identity & Assets
[Who the user is, what they've built, their domain expertise, what they're known for. Infer from their answers.]

## Active Projects
[Bullet list of what they're working on right now. Use their exact words where possible.]

## Opportunity Categories (Ranked)
[Numbered list from most to least important, based on what they said. Each item: category name + one-line description of what counts.]

1. [Category] — [what counts as this type of opportunity for this user]
2. ...

## What to Ignore
[Bullet list of topics, content types, or noise patterns the user explicitly wants skipped. Be specific.]

## Scoring Guidance
[3–5 sentences that describe how to weight signals for this user. What pushes a score up? What caps it low? What makes something a 90+?]

## Example Perfect Signal
[One concrete example of what a 90+ score looks like for this user, in their own terms. This is the calibration anchor.]
```

### Step 2: Write ~/Documents/Signal/sources.md

Write a sources file with specific, targeted queries for each topic area the user mentioned. Format:

```markdown
# Scanner Sources

Last updated: YYYY-MM-DD

## [Domain/Topic Name]

**Query:** [exact search query]
**Routing:** web search
**Frequency:** daily
**Notes:** [any filtering or context — e.g. "prioritize last 24h", "exclude job board results"]

---
[repeat for each source]
```

Derive sources from what the user said — their existing follows, desired coverage gaps, and specific domains. Aim for 8–15 sources. More is not better — precision over volume.

### Step 3: Confirm and summarize

After writing both files:
1. Confirm both files were written successfully.
2. Show the user a brief summary: their top 3 opportunity categories and how many sources are configured.
3. Proceed immediately to the **After the Interview** steps below — do not stop here.

## After the Interview

After writing `~/Documents/Signal/lens.md` and `~/Documents/Signal/sources.md`:
1. Run `/schedule` to create the daily 8am scan task.
2. Run `/scan` immediately to fetch sources, score signals, and build the Live Artifact dashboard.
3. Confirm to the user: "Signal is set up. Your lens is configured, your 8am daily scan is scheduled, and here's your first dashboard."

Do all three in the same session. Do not ask the user to do anything between steps.

## Important

- Write to `~/Documents/Signal/lens.md` and `~/Documents/Signal/sources.md`. Create `~/Documents/Signal/` if it does not exist.
- Do not start scanning during this skill. Lens-building and scanning are separate steps handled by the "After the Interview" section above.
- If the user gives vague or generic answers (e.g. "AI stuff"), ask a focused follow-up before moving on.
- The lens is a living document — the user can update it later with `/lens` or inline in chat.
