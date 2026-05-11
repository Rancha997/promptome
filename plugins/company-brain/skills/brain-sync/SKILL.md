---
description: Rebuilds the agent context file from all source docs. Run after heavy updates.
---

# Skill: brain-sync

Full rebuild of the Company Brain context file from all source docs. Invoked by `/brain-sync`. Use this after a period of heavy updates or when the context file feels out of sync.

---

## Steps

1. **Find the brain** — search Notion for the page with "Brain — Context" in the title to get the brain location.

2. **Fetch all section pages** from the brain using the Quick Index URLs in the Context page:
   - Identity & Strategy (and its child pages)
   - Projects (and all project subfolders)
   - Infrastructure (and all system pages)
   - Decisions Log
   - People & Conventions (and its child pages)
   - Open Questions

3. **Re-read the full content** of each source doc.

4. **Rebuild "Company Brain — Context" from scratch**:
   - Compress all source doc content into the dense agent-optimized format defined in `brain-builder`
   - Preserve the exact section structure (Identity, Stack & Infrastructure, Active Projects, Decisions Log, Strategy & Monetization, People & Conventions, Rejected Ideas, Open Questions)
   - Every entry should be a bullet point — no prose
   - Remove outdated or superseded entries if the source docs make their status clear
   - Update the Quick Index to reflect current contents

5. **Update `Last updated`** in the context file header to today's date.

6. **Confirm** to the user:
   > "Brain synced — context file rebuilt from [N] source docs. [X] decisions, [Y] projects, [Z] open questions."
