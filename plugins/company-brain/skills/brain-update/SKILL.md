---
description: Saves a specific piece of knowledge to the Company Brain. Invoked manually or via passive save suggestions.
---

# Skill: brain-update

Saves a specific piece of knowledge to the Company Brain. Invoked when the user says "save this to the brain", clicks Yes on a passive save suggestion, or runs `/brain-update`.

---

## Steps

1. **Find the brain** — search Notion for the page with "Brain — Context" in the title to get the brain location.

2. **Extract what to save** from the current conversation:
   - Identify the specific decision, convention, rejection, status change, or new fact
   - Distill it to its essential form — one crisp bullet for the context file, fuller detail for source docs if warranted

3. **Determine the correct section** of the context file:
   - New system, tool, or integration → **Stack & Infrastructure**
   - Project status change → **Active Projects**
   - Decision made and why → **Decisions Log**
   - Revenue or growth change → **Strategy & Monetization**
   - New person, role, or working convention → **People & Conventions**
   - Something explicitly ruled out → **Rejected Ideas**
   - Unresolved open debate → **Open Questions**

4. **Update the context file** on the platform:
   - Add the new entry to the correct section
   - For Decisions Log: prepend (most recent first) with format `[Date] | [Decision] | [Why] | [What was rejected]`
   - Update the `Last updated` date in the file header
   - Do not duplicate — check if the information is already present before writing

5. **If substantial** (a decision with full reasoning, a new system description, a project milestone):
   - Also append the full detail to the appropriate source doc (Stack & Architecture, Decision Log, or Projects)

6. **Confirm** to the user:
   > "Saved to Company Brain — [one line of what was saved] → [section name]"

---

## Rules

- Context file entries: one bullet maximum per item. Dense, not wordy.
- Full reasoning and detail belong in source docs only.
- Always update `Last updated` in the context file header.
- Never write the same fact twice — read the relevant section before writing.
