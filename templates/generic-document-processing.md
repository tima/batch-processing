# Generic Document Processing Template

**Business Context:**
Flexible template for any bulk document processing task. Extract specific information, identify patterns, or transform data from 5+ similar documents. Adapt this template by filling in the bracketed sections with your specific extraction criteria.

**Use when:** Processing 5+ documents of the same type (meeting notes, research papers, interview transcripts, reports, etc.) to extract specific information.

## Prompt

```
GOAL:
I want you to analyze all the [DOCUMENT TYPE] in this directory. Extract [SPECIFIC INFORMATION YOU WANT]. Focus on [WHAT MATTERS MOST] and ignore [WHAT TO SKIP].

BEFORE YOU START:
Create three files:

1. context.md - Store this entire goal statement and the extraction criteria
2. todos.md - Create a numbered checklist of every file in this directory
3. insights.md - This will store all extracted information, organized by [YOUR CATEGORIES]

AS YOU WORK:
- Process one document at a time
- After processing each document, update insights.md with extracted information (include source file name and brief context)
- Immediately after updating insights.md, check off that document in todos.md
- CRITICAL: Make sure todos.md is up to date BEFORE your memory resets

AFTER YOUR MEMORY RESETS:
- First, read context.md to remember your goal
- Then, read todos.md to see which documents you've already completed
- Continue processing from where you left off

EXTRACTION RULES:
[List your specific criteria - examples:]
- Extract only [SPECIFIC TYPE OF INFORMATION]
- Include [REQUIRED DETAILS] with each finding
- Categorize by [YOUR CATEGORIES]
- Track frequency if same item appears multiple times
- Ignore [WHAT TO SKIP]

OUTPUT FORMAT IN insights.md:

Choose merge strategy based on your needs:
- Category-based: Group by [YOUR CATEGORIES]
- Chronological: Preserve document order
- Frequency-based: Most mentioned items first

Example:
## [CATEGORY 1]
- [Finding with details] (source_file.txt - brief context)
- [Another finding] (another_file.txt - brief context)

## [CATEGORY 2]
- [Finding] (file.txt - context)

Work through all documents in this directory until complete. Do not stop until every file in todos.md is checked off.
```
