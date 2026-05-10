---
description: Rebuilds the agent context file from all source docs. Run after heavy updates.
---

# Skill: brain-sync

Full rebuild of the Company Brain context file from all source docs. Invoked by `/brain-sync`. Use this after a period of heavy updates or when the context file feels out of sync.

---

## Steps

1. **Read `config.json`** to get the platform type and brain location.

2. **Fetch all Company Brain source docs** from the platform:
   - "Company Brain — Stack & Architecture"
   - "Company Brain — Decision Log"
   - "Company Brain — Projects"
   - Any other docs linked from the context file

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
