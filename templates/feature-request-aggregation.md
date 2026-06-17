# Feature Request Aggregation Template

**Business Context:**
Build data-driven product roadmaps by aggregating feature requests from actual customer conversations. Identify most-requested features, understand urgency, and prioritize based on real demand rather than gut feel. Directly feeds into product planning and roadmap prioritization.

**Use when:** Analyzing 5+ customer conversations, feedback sessions, or support interactions to identify feature requests and rank by demand.

## Prompt

```
GOAL:
I want you to analyze all the transcripts/documents in this directory. Extract feature requests, suggestions, or "I wish..." statements from customers. Track frequency to identify the most commonly requested features.

BEFORE YOU START:
Create three files:

1. context.md - Store this entire goal statement and the extraction criteria
2. todos.md - Create a numbered checklist of every file in this directory
3. insights.md - This will store all feature requests, organized by frequency and product area

AS YOU WORK:
- Process one file at a time
- After processing each file, update insights.md with any feature requests (include exact quote, source file, and context)
- Immediately after updating insights.md, check off that file in todos.md
- CRITICAL: Make sure todos.md is up to date BEFORE your memory resets

AFTER YOUR MEMORY RESETS:
- First, read context.md to remember your goal
- Then, read todos.md to see which files you've already completed
- Continue processing from where you left off

EXTRACTION RULES:
- Extract requests, suggestions, wishes, or improvement ideas
- Include exact quote and surrounding context
- Note if mentioned by multiple customers - track frequency as you go
- Categorize by product area if clear from context
- Distinguish "must-have" from "nice-to-have" based on customer language intensity
- Track whether customer said they'd pay for it, or mentioned competitors having it

OUTPUT FORMAT IN insights.md:

Frequency-based merge recommended for feature requests.

Example (frequency-based):
## High Frequency (3+ mentions)
- "Bulk export to CSV" (ticket_023.txt, ticket_091.txt, transcript_15.txt - all mentioned needing CSV export)

Example (category-based):
## Authentication
- Two-factor authentication (transcript_08.txt, transcript_22.txt, transcript_31.txt)

## Medium Frequency (2 mentions)
- "Dark mode interface" (ticket_045.txt, transcript_19.txt - both called it "nice to have")

## Single Mentions - High Priority
- "API access for automation" (transcript_12.txt - customer said they'd upgrade plan for this)

Work through all files in this directory until complete. Do not stop until every file in todos.md is checked off.
```
