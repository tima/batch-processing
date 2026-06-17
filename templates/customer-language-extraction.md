# Customer Language Extraction Template

**Business Context:**
Extract real customer language from sales/support conversations to use in ad copy, landing pages, and marketing content. Beats assumptions and theory - these are actual phrases customers use when describing pain points. Directly applicable to copywriting, messaging, and positioning.

**Use when:** Analyzing 5+ sales calls, prospect conversations, or support transcripts to find emotional language for marketing.

## Prompt

```
GOAL:
I want you to analyze all the meeting transcripts in this directory. Extract phrases, questions, and statements where customers or prospects describe problems, frustrations, stress, confusion, or pain points. Only include language that shows genuine emotion or urgency - ignore casual mentions or theoretical scenarios.

BEFORE YOU START:
Create three files:

1. context.md - Store this entire goal statement and the extraction criteria
2. todos.md - Create a numbered checklist of every transcript file in this directory
3. insights.md - This will store all extracted customer language, organized by emotional category (Frustration, Fear, Confusion, Stress, Pain Points)

AS YOU WORK:
- Process one transcript at a time
- After processing each transcript, update insights.md with any relevant quotes (include the file name and approximate timestamp if available)
- Immediately after updating insights.md, check off that transcript in todos.md
- CRITICAL: Make sure todos.md is up to date BEFORE your memory resets

AFTER YOUR MEMORY RESETS:
- First, read context.md to remember your goal
- Then, read todos.md to see which transcripts you've already completed
- Continue processing from where you left off

EXTRACTION RULES:
- Only extract direct quotes - do not paraphrase
- Only include statements tied to visible emotion (tone shifts, repeated mentions, explicit statements like "this is frustrating")
- Categorize each quote by the primary emotion: Frustration, Fear, Confusion, Stress, or Pain Points
- If a quote fits multiple categories, include it in both
- Ignore any discussion about competitors unless it reveals our product's gaps
- Ignore feature requests (we'll extract those separately)

OUTPUT FORMAT IN insights.md:

Choose merge strategy when starting parallel mode:
- Category-based (recommended): Organize by emotion (Frustration, Fear, Confusion, Stress, Pain Points)
- Chronological: Organize by transcript order
- Frequency-based: Organize by how often emotions mentioned

Example (category-based):
## Frustration
- "We've been manually doing this for six months and it's killing us" (transcript_042.txt - data entry workflow)
- "Every time I try to export, the system times out" (transcript_018.txt - export feature complaint)

## Confusion
- "I don't understand why it works this way" (transcript_031.txt - questioning workflow logic)

Example (frequency-based):
## High Frequency (5+ mentions)
- Export timeout frustration (transcript_018.txt, transcript_024.txt, transcript_031.txt, transcript_042.txt, transcript_055.txt)

Work through all transcripts in this directory until complete. Do not stop until every file in todos.md is checked off.
```
