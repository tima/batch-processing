# Support Ticket Analysis Template

**Business Context:**
Identify recurring technical issues, common failure patterns, and root causes from support tickets. Use findings to prioritize bug fixes, improve documentation, or redesign problematic workflows. Reduces support load by addressing systemic issues rather than treating symptoms.

**Use when:** Analyzing 5+ support tickets to find patterns, recurring issues, or common root causes across customer problems.

## Prompt

```
GOAL:
I want you to analyze all the support tickets in this directory. Extract recurring issues, error patterns, and root causes. Categorize by severity and product area to identify what needs fixing most urgently.

BEFORE YOU START:
Create three files:

1. context.md - Store this entire goal statement and the extraction criteria
2. todos.md - Create a numbered checklist of every ticket file in this directory
3. insights.md - This will store all issues, organized by severity and product area

AS YOU WORK:
- Process one ticket at a time
- After processing each ticket, update insights.md with any issues (include severity, description, root cause if known, source ticket)
- Immediately after updating insights.md, check off that ticket in todos.md
- CRITICAL: Make sure todos.md is up to date BEFORE your memory resets

AFTER YOUR MEMORY RESETS:
- First, read context.md to remember your goal
- Then, read todos.md to see which tickets you've already completed
- Continue processing from where you left off

EXTRACTION RULES:
- Identify recurring error messages or failure patterns
- Extract root cause if customer or support agent mentioned it
- Note workarounds used (indicates pain points worth fixing properly)
- Categorize severity: Critical (product blocking), High (workflow impact), Medium (friction/annoyance)
- Track product area affected (authentication, export, API, etc.)
- Note if multiple tickets mention the same issue - track frequency

OUTPUT FORMAT IN insights.md:

Category-based merge recommended (by severity, then product area).

Example:
## Critical - Authentication
- Login timeout after 2FA (ticket_034.txt, ticket_089.txt - root cause: session timeout = 30s but 2FA takes 45s, affects 2 customers)
- Password reset emails not arriving (ticket_056.txt, ticket_071.txt, ticket_103.txt - spam filter issue, workaround: whitelist domain)

## High - Export Feature
- CSV export fails for datasets over 10k rows (ticket_042.txt, ticket_091.txt - timeout issue, workaround: split into smaller exports)

## Medium - UI/UX
- Confusing navigation in settings panel (ticket_015.txt - customer couldn't find API keys section)

Work through all tickets in this directory until complete. Do not stop until every file in todos.md is checked off.
```
