# Batch Processing Skill Improvements Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add parallel resume, dry-run mode, adaptive agent calculation, validation sampling, configurable merge, and token efficiency improvements to batch-processing skill.

**Architecture:** Extends existing batch-processing skill with new workflows and capabilities while maintaining backward compatibility. All improvements are additive - existing sequential and parallel modes continue to work unchanged.

**Tech Stack:** Markdown documentation updates, no code changes required (skill is documentation-only).

## Global Constraints

- Maintain backward compatibility - existing workflows must work unchanged
- Follow existing skill structure and formatting conventions
- Keep token count minimal - compress where possible, reference where appropriate
- Use exact file paths and complete code/markdown examples in every step
- No placeholders or "TBD" content

---

### Task 1: Add Parallel Resume Workflow

**Files:**
- Modify: `SKILL.md` (Detection Pattern section ~line 50-92)
- Modify: `SKILL.md` (Parallel Processing Workflow section ~line 167-245)
- Modify: `SKILL.md` (Troubleshooting section ~line 550-620)

**Interfaces:**
- Consumes: Existing Detection Pattern and Parallel Processing Workflow sections
- Produces: Resume workflow that other tasks can reference, used by validation and troubleshooting

- [ ] **Step 1: Add resume detection to Detection Pattern section**

Find the Detection Pattern section (after "Parallel vs Sequential" section). After step 6 "Verify completion", add new step 7:

```markdown
7. **If resuming interrupted parallel batch:**
   - Check for existing todos.md with mix of `[ ]`, `[>]`, and `[x]` items
   - Count remaining: `remaining = count([ ]) + count([>])`
   - Calculate N = min(ceil(remaining / 10), 5)
   - Inform user: "Resuming parallel batch - [N] subagents for [remaining] remaining items"
   - Read existing context.md (preserve original extraction rules)
   - Clean up stale `[>]` items (if any subagent IDs no longer active)
   - Identify highest existing insights-N.md number
   - Create additional insights files if N > existing max
   - Spawn N subagents with same work-stealing instructions
   - Monitor and merge as normal
```

- [ ] **Step 2: Verify documentation renders correctly**

Read the modified section to ensure:
- Step 7 follows step 6 logically
- Indentation matches surrounding content
- Code blocks are properly formatted

- [ ] **Step 3: Add resume workflow to Parallel Processing Workflow section**

Find the "Parallel Processing Workflow" section heading. After the "Failure Recovery" subsection, add new subsection:

```markdown
**Resuming Interrupted Parallel Batch:**

If coordinator crashes or user stops processing mid-batch:

1. **Check state:**
   - todos.md exists with mix of states
   - context.md exists (original extraction rules)
   - Some insights-N.md files exist (partial work)

2. **Calculate remaining work:**
   ```bash
   # Count unclaimed and in-progress items
   remaining=$(grep -c '^\- \[ \]' todos.md)
   in_progress=$(grep -c '^\- \[>\]' todos.md)
   total_remaining=$((remaining + in_progress))
   ```

3. **Clean stale claims:**
   - Any `[>] item.txt (agent-N)` where agent N is not running
   - Change back to `[ ]` for retry

4. **Resume:**
   - Calculate N = min(ceil(total_remaining / 10), 5)
   - Identify max existing insights-N.md number
   - Create additional insights-M.md files if N > max (empty)
   - Spawn N subagents
   - Continue work-stealing from current todos.md state
   - Merge all insights-*.md at completion (including partial files from first run)
```

- [ ] **Step 4: Verify workflow addition**

Read the Parallel Processing Workflow section to ensure:
- New subsection follows existing subsections
- Code block formatting is correct
- Resume workflow is complete and actionable

- [ ] **Step 5: Add resume troubleshooting entry**

Find the "Parallel Mode Issues" subsection in Troubleshooting section. Add new entry at the end:

```markdown
**Resume After Coordinator Crash**

**Symptom:** Coordinator crashed mid-parallel batch, some items complete, some in-progress, some unclaimed.

**State:**
- todos.md has mix of `[ ]`, `[>]`, `[x]` items
- context.md exists
- Some insights-N.md files exist

**Recovery:**
1. Count remaining: `remaining = count([ ]) + count([>])`
2. Calculate N = min(ceil(remaining / 10), 5)
3. Clean stale `[>]` claims (change to `[ ]`)
4. Identify max existing insights-N.md number
5. Create additional insights files if needed (insights-M.md where M > max)
6. Tell AI: "Resume parallel batch processing - read context.md and todos.md, spawn [N] subagents"
7. AI will spawn subagents, work-steal remaining items, merge all insights-*.md
```

- [ ] **Step 6: Verify troubleshooting entry**

Read the Parallel Mode Issues subsection to ensure:
- New entry follows existing format
- Symptom/State/Recovery structure is consistent
- Recovery steps are actionable

- [ ] **Step 7: Commit Task 1**

```bash
git add SKILL.md
git commit -m "feat: add parallel resume workflow for interrupted batches"
```

---

### Task 2: Add Dry-Run Mode

**Files:**
- Modify: `SKILL.md` (Detection Pattern section ~line 50-92)
- Modify: `SKILL.md` (Workflow section ~line 128-165)
- Modify: `SKILL.md` (Quick Reference section ~line 93-127)

**Interfaces:**
- Consumes: Existing Detection Pattern workflow
- Produces: Dry-run workflow that validation (Task 4) can reference

- [ ] **Step 1: Add dry-run offer to Detection Pattern**

Find step 4 "If sequential" in Detection Pattern section. Replace with:

```markdown
4. **If sequential:**
   - Offer: "Preview output first (dry-run on first 3 items) or process entire batch?"
   - **If dry-run:**
     - Create context.md with their goal
     - Enumerate files to create todos.md
     - Mark only first 3 items as active: `- [ ] file.txt # DRY-RUN`
     - Create empty insights.md
     - Process first 3 items
     - Show insights.md to user
     - Ask: "Does output format look correct? Adjust extraction rules or continue?"
     - If adjust: update context.md, clear insights.md, reprocess 3 items
     - If continue: remove # DRY-RUN markers, process remaining items
   - **If full batch:**
     - Create context.md with their goal
     - Enumerate files to create todos.md (format: `- [ ] filename`)
     - Create empty insights.md
     - Begin sequential processing loop
     - **Work through all items until complete**
```

- [ ] **Step 2: Update step 5 for parallel dry-run**

Find step 5 "If parallel" in Detection Pattern section. Replace with:

```markdown
5. **If parallel:**
   - Offer: "Preview output first (dry-run on first 10 items) or process entire batch?"
   - **If dry-run:**
     - Create context.md with their goal
     - Enumerate files to create todos.md
     - Mark only first 10 items as active: `- [ ] file.txt # DRY-RUN`
     - Calculate N = min(ceil(10 / 10), 5) = 1 subagent for dry-run
     - Create empty insights-1.md
     - Spawn 1 subagent to process first 10 items
     - Merge to insights.md
     - Show insights.md to user
     - Ask: "Does output format look correct? Adjust extraction rules or continue?"
     - If adjust: update context.md, reset first 10 items to `[ ]`, clear insights-1.md, reprocess
     - If continue: remove # DRY-RUN markers, calculate full N, spawn remaining subagents, process all items
   - **If full batch:**
     - Calculate N = min(ceil([item_count] / 10), 5)
     - Inform user: "Using [N] subagents for [item_count] items"
     - Create context.md with their goal
     - Enumerate files to create todos.md (format: `- [ ] filename`)
     - Create empty insights-1.md through insights-N.md
     - Spawn N subagents with work-stealing instructions
     - Monitor progress, merge when complete
```

- [ ] **Step 3: Verify Detection Pattern updates**

Read the updated Detection Pattern section to ensure:
- Dry-run offer is clear
- Dry-run and full batch workflows are distinct
- # DRY-RUN marker usage is consistent

- [ ] **Step 4: Add dry-run workflow to Workflow section**

Find the "Initial Setup (Automatic)" subsection in Workflow section. After this subsection, add new subsection:

```markdown
### Dry-Run Preview Mode

For batches where you're unsure about extraction rules, dry-run mode processes a small sample first.

**Sequential dry-run (3 items):**
1. AI processes first 3 items from todos.md (marked `# DRY-RUN`)
2. Shows insights.md output
3. Asks: "Does this match your expectations? Adjust rules or continue?"
4. If adjust: updates context.md, clears insights.md, reprocesses same 3 items
5. If continue: removes # DRY-RUN markers, processes remaining items

**Parallel dry-run (10 items):**
1. AI spawns 1 subagent to process first 10 items (marked `# DRY-RUN`)
2. Merges to insights.md
3. Shows insights.md output
4. Asks: "Does this match your expectations? Adjust rules or continue?"
5. If adjust: updates context.md, resets first 10 to `[ ]`, reprocesses
6. If continue: removes # DRY-RUN markers, spawns full N subagents, processes all items

**When to use dry-run:**
- First time using batch processing
- Unclear extraction rules
- Complex categorization requirements
- High stakes (can't afford to reprocess entire batch)
```

- [ ] **Step 5: Verify workflow addition**

Read the Workflow section to ensure:
- Dry-run subsection follows Initial Setup logically
- Sequential and parallel dry-run workflows are clear
- "When to use" guidance is actionable

- [ ] **Step 6: Add dry-run to Quick Reference**

Find the Quick Reference section. After the "Parallel Mode Pattern" subsection, add:

```markdown
**Dry-Run Pattern:**
1. AI offers: "Preview output first or process entire batch?"
2. If preview: processes 3 items (sequential) or 10 items (parallel)
3. Shows insights.md
4. User adjusts rules or continues with full batch
```

- [ ] **Step 7: Verify Quick Reference update**

Read Quick Reference to ensure:
- Dry-run pattern follows existing patterns
- Format is consistent with other patterns
- Content is concise (1-4 lines per pattern)

- [ ] **Step 8: Commit Task 2**

```bash
git add SKILL.md
git commit -m "feat: add dry-run preview mode for sequential and parallel processing"
```

---

### Task 3: Add Adaptive Agent Calculation

**Files:**
- Modify: `SKILL.md` (Parallel vs Sequential section ~line 26-48)
- Modify: `SKILL.md` (Detection Pattern section ~line 50-92)
- Modify: `SKILL.md` (Parallel Processing Workflow section ~line 167-245)

**Interfaces:**
- Consumes: Existing parallel mode formula `N = min(ceil(item_count / 10), 5)`
- Produces: Adaptive formula that dry-run mode (Task 2) and resume (Task 1) use

- [ ] **Step 1: Update Parallel vs Sequential section with adaptive calculation**

Find the "Parallel Mode:" description in "Parallel vs Sequential" section. Replace the "Time:" line with:

```markdown
- Time: Adaptive based on first item - measures first item processing time, estimates total, calculates N
- Formula: `N = min(ceil(estimated_total_minutes / 20), 5)` where 20min is target completion window
- Example: First item takes 3 min → 50 items = 150 min total → N = min(ceil(150/20), 5) = 5 agents → 30 min completion
```

- [ ] **Step 2: Verify section update**

Read the Parallel vs Sequential section to ensure:
- Adaptive formula is clear
- Example calculation is correct
- Time explanation is complete

- [ ] **Step 3: Update Detection Pattern with adaptive calculation**

Find step 5 "If parallel" in Detection Pattern section. Replace the "Calculate N" line with:

```markdown
   - Measure first item: process first item, record time T (minutes)
   - Estimate total: `estimated_total = T * item_count`
   - Calculate N = min(ceil(estimated_total / 20), 5) where 20min is target window
   - If T < 0.5 min: N = 1 (items too fast for parallel overhead)
   - If T > 5 min: N = 5 (items complex enough to always max out)
   - Inform user: "First item took [T] min, estimated [estimated_total] min total, using [N] subagents for ~[estimated_total/N] min completion"
```

- [ ] **Step 4: Verify Detection Pattern update**

Read step 5 to ensure:
- Adaptive calculation is integrated correctly
- Special cases (< 0.5 min, > 5 min) are handled
- User messaging includes timing information

- [ ] **Step 5: Update Parallel Processing Workflow with adaptive formula**

Find the "Coordinator Role" subsection in Parallel Processing Workflow. Replace the "Spawns N subagents" line with:

```markdown
- Measures first item processing time T
- Calculates N adaptively: `N = min(ceil(T * item_count / 20), 5)`
- Special cases: T < 0.5 → N=1, T > 5 → N=5
- Spawns N subagents
```

- [ ] **Step 6: Verify workflow update**

Read the Coordinator Role subsection to ensure:
- Adaptive formula is consistent with Detection Pattern
- Special cases match
- Content is concise

- [ ] **Step 7: Update parallel resume (from Task 1) with adaptive calculation**

Find the "Resuming Interrupted Parallel Batch" subsection (added in Task 1). Replace the "Calculate N" step with:

```markdown
4. **Resume with adaptive calculation:**
   - If insights-*.md files exist, estimate T from previous work:
     - `avg_time_per_item = total_elapsed / items_completed`
     - `estimated_remaining = avg_time_per_item * total_remaining`
   - Else measure first remaining item for T
   - Calculate N = min(ceil(estimated_remaining / 20), 5)
   - If T < 0.5: N=1, if T > 5: N=5
   - Identify max existing insights-N.md number
   - Create additional insights-M.md files if N > max (empty)
   - Spawn N subagents
   - Continue work-stealing from current todos.md state
   - Merge all insights-*.md at completion
```

- [ ] **Step 8: Verify resume workflow update**

Read the Resuming Interrupted Parallel Batch subsection to ensure:
- Adaptive calculation is integrated
- Fallback to measuring first item if no history exists
- Resume workflow is complete

- [ ] **Step 9: Commit Task 3**

```bash
git add SKILL.md
git commit -m "feat: add adaptive subagent calculation based on first item timing"
```

---

### Task 4: Add Validation Sampling Methodology

**Files:**
- Modify: `SKILL.md` (Validation section ~line 516-549)
- Modify: `SKILL.md` (Workflow section ~line 128-165)

**Interfaces:**
- Consumes: Dry-run mode (Task 2) for preview validation
- Produces: Validation methodology that troubleshooting can reference

- [ ] **Step 1: Replace Validation section with sampling methodology**

Find the "Validation" section heading. Replace entire section content (keep heading) with:

```markdown
## Validation

**Completion checks (automatic):**
- All items in todos.md checked off
- insights.md has entries
- Each insight includes source filename

**Validation Sampling Strategy:**

For batches > 20 items, spot-checking 2-3 items is insufficient. Use stratified sampling:

**Small batches (5-20 items):**
- Verify: 3 items (15-60% coverage)
- Select: First, middle, last items
- Check: Extraction matches context.md rules, categorization correct, source attribution present

**Medium batches (21-50 items):**
- Verify: 10% random sample (min 5 items)
- Select: Random items across batch (use `shuf` or random.org)
- Check: Same as small batches + category distribution matches expectations

**Large batches (50+ items):**
- Verify: 5% random sample (min 10 items)
- Select: Stratified random (2 items per category minimum)
- Check: Same as medium + no systematic errors within categories

**Early Detection (recommended):**

After first 10 items processed in any batch:
1. **Pause and validate:** Check 3 random items from first 10
2. **Look for systematic errors:**
   - Wrong category assignments
   - Missed extractions (too narrow)
   - Over-extraction (too broad)
   - Source attribution missing
3. **If errors found:**
   - STOP processing
   - Update context.md with clearer rules
   - Reset all items to `[ ]` in todos.md
   - Clear insights.md (or insights-*.md for parallel)
   - Restart with corrected rules
4. **If validation passes:** Continue processing remaining items

**Why early detection matters:** 5% error rate on 100 items = 5 wasted items if caught early, 95 wasted items if caught at end.

**Reported at completion:**
- Items processed: X/Y
- Output location: insights.md
- Time elapsed (if available)
- Validation sample: "Checked N items ([percentage]%), [errors] errors found"

**Parallel Mode Additional Checks:**

**Before merge cleanup:**
- Verify insights-1.md through insights-N.md all exist
- Sample 1 finding from each insights-N.md (check quality per subagent)
- Verify each insights-N.md includes source filenames

**After merge:**
- Verify insights.md exists and has content from all subagents
- Count findings per category - should match sum of all insights-N.md
- Verify no category duplication (each category appears once)
- Verify intermediate files (insights-1.md through insights-N.md) are deleted

**If merge validation fails:**
- Intermediate files should still exist (not deleted prematurely)
- Re-run merge manually or with coordinator
```

- [ ] **Step 2: Verify Validation section replacement**

Read the new Validation section to ensure:
- Sampling methodology is clear and actionable
- Different batch sizes have appropriate sampling rates
- Early detection workflow is complete
- Parallel mode checks are preserved

- [ ] **Step 3: Add early validation to Workflow section**

Find the "Processing Loop" subsection in Workflow section. After the "IMPORTANT: Continue this loop" paragraph, add:

```markdown
**Early Validation Checkpoint (recommended):**

After processing first 10 items:
1. Pause loop
2. Validate 3 random items from insights.md
3. If errors found: stop, update context.md, reset todos.md, restart
4. If validation passes: continue processing

This catches systematic errors early before wasting work on entire batch.
```

- [ ] **Step 4: Verify Workflow addition**

Read the Processing Loop subsection to ensure:
- Early validation checkpoint follows the loop description logically
- Workflow is concise and actionable
- Integration with Processing Loop is clear

- [ ] **Step 5: Commit Task 4**

```bash
git add SKILL.md
git commit -m "feat: add validation sampling methodology with early detection"
```

---

### Task 5: Add Configurable Merge Strategy

**Files:**
- Modify: `SKILL.md` (Parallel Processing Workflow section ~line 167-245)
- Modify: `SKILL.md` (Detection Pattern section ~line 50-92)
- Modify: `SKILL.md` (Ready-to-Use Templates sections ~line 296-514)

**Interfaces:**
- Consumes: Existing category-based merge from parallel workflow
- Produces: Merge strategy selection that templates reference

- [ ] **Step 1: Add merge strategy to Detection Pattern**

Find step 5 "If parallel" in Detection Pattern section. After the "Create empty insights-1.md through insights-N.md" line, add:

```markdown
   - Ask merge strategy: "How should findings be organized in final insights.md?"
     - A) By category (Frustration, Confusion, etc.) - groups similar findings
     - B) Chronologically (by file order in todos.md) - preserves sequence
     - C) By frequency (most mentioned first) - highlights common patterns
   - Store choice in context.md for merge step
```

- [ ] **Step 2: Verify Detection Pattern update**

Read step 5 to ensure:
- Merge strategy question is clear
- Three options are distinct and understandable
- Storage in context.md is mentioned

- [ ] **Step 3: Update Parallel Processing Workflow merge logic**

Find the "Merge Step" subsection in Parallel Processing Workflow. Replace the existing merge description (steps 1-6) with:

```markdown
When all items are `[x]`:
1. Coordinator waits for all subagents to finish
2. Reads context.md for merge strategy choice
3. Reads insights-1.md through insights-N.md
4. **Applies chosen merge strategy:**

   **A) Category-based (default):**
   - Groups findings by category (Frustration, Confusion, etc.)
   - Writes insights.md with category headers
   - Findings organized under relevant categories from all subagents

   **B) Chronological:**
   - Groups findings by source file order (from todos.md)
   - Writes insights.md with file-based headers: "## Findings from file_01.txt"
   - Preserves processing sequence across all subagents

   **C) Frequency-based:**
   - Counts mentions of each finding across insights-*.md files
   - Ranks by frequency (most mentioned first)
   - Writes insights.md with frequency headers: "## High Frequency (5+ mentions)"
   - Includes source files for each finding

5. Deletes intermediate insights-*.md files
6. Reports completion: "Processed Y items, output in insights.md (organized by [strategy])"
```

- [ ] **Step 4: Verify merge logic update**

Read the Merge Step subsection to ensure:
- Three strategies are clearly defined
- Default (category-based) is marked
- Each strategy has distinct output format
- Deletion and reporting steps are preserved

- [ ] **Step 5: Update example merge output**

Find the "Example merge:" code block in Parallel Processing Workflow. Replace with:

```markdown
**Example merge (category-based):**

```markdown
## Frustration (from insights-1.md, insights-3.md, insights-5.md)
- "We've been doing this manually for months" (file_01.txt - workflow pain)
- "System times out every time" (file_15.txt - export issue)

## Confusion (from insights-2.md, insights-4.md)
- "Why does it work this way?" (file_07.txt - questioning logic)
```

**Example merge (chronological):**

```markdown
## Findings from file_01.txt
- "We've been doing this manually for months" (insights-1.md - Frustration)

## Findings from file_07.txt
- "Why does it work this way?" (insights-2.md - Confusion)

## Findings from file_15.txt
- "System times out every time" (insights-3.md - Frustration)
```

**Example merge (frequency-based):**

```markdown
## High Frequency (3+ mentions)
- "System times out" (file_15.txt, file_23.txt, file_41.txt - export issue mentioned by 3 customers)

## Medium Frequency (2 mentions)
- "Confusing navigation" (file_07.txt, file_19.txt - UI issue)
```
```

- [ ] **Step 6: Verify example updates**

Read the example merge outputs to ensure:
- All three strategies have example output
- Format differences are clear
- Examples show how findings from multiple insights-*.md files combine

- [ ] **Step 7: Update Template 1 (Customer Language) with merge strategy**

Find Template 1 "Customer Language Extraction" in Ready-to-Use Templates section. In the "OUTPUT FORMAT IN insights.md:" section, replace with:

```markdown
OUTPUT FORMAT IN insights.md:

Choose merge strategy when starting parallel mode:
- Category-based (recommended): Organize by emotion (Frustration, Fear, Confusion, Stress, Pain Points)
- Chronological: Organize by transcript order
- Frequency-based: Organize by how often emotions mentioned

Example (category-based):
## Frustration
- "We've been manually doing this for six months and it's killing us" (transcript_042.txt - data entry workflow)
- "Every time I try to export, the system times out" (transcript_018.txt - export feature complaint)

Example (frequency-based):
## High Frequency (5+ mentions)
- Export timeout frustration (transcript_018.txt, transcript_024.txt, transcript_031.txt, transcript_042.txt, transcript_055.txt)
```

- [ ] **Step 8: Update Template 2 (Feature Requests) with merge strategy**

Find Template 2 "Feature Request Aggregation". In the "OUTPUT FORMAT IN insights.md:" section, replace with:

```markdown
OUTPUT FORMAT IN insights.md:

Frequency-based merge recommended for feature requests.

Example (frequency-based):
## High Frequency (3+ mentions)
- "Bulk export to CSV" (ticket_023.txt, ticket_091.txt, transcript_15.txt - all mentioned needing CSV export)

Example (category-based):
## Authentication
- Two-factor authentication (transcript_08.txt, transcript_22.txt, transcript_31.txt)

Example (chronological):
## Transcript_08.txt
- Two-factor authentication request (security requirement)
```

- [ ] **Step 9: Verify template updates**

Read both updated templates to ensure:
- Merge strategy guidance is clear
- Recommended strategies make sense for use case
- Examples show different strategies

- [ ] **Step 10: Commit Task 5**

```bash
git add SKILL.md
git commit -m "feat: add configurable merge strategy (category, chronological, frequency)"
```

---

### Task 6: Move Templates to Separate Files

**Files:**
- Create: `templates/customer-language-extraction.md`
- Create: `templates/feature-request-aggregation.md`
- Create: `templates/support-ticket-analysis.md`
- Create: `templates/generic-document-processing.md`
- Create: `templates/README.md`
- Modify: `SKILL.md` (Ready-to-Use Templates section ~line 296-514)

**Interfaces:**
- Consumes: Existing inline templates in SKILL.md
- Produces: Separate template files that SKILL.md references

- [ ] **Step 1: Create templates directory**

```bash
mkdir -p templates
```

- [ ] **Step 2: Extract Template 1 to file**

Create `templates/customer-language-extraction.md`:

```markdown
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
```

- [ ] **Step 3: Extract Template 2 to file**

Create `templates/feature-request-aggregation.md`:

```markdown
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
```

- [ ] **Step 4: Extract Template 3 to file**

Create `templates/support-ticket-analysis.md`:

```markdown
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
```

- [ ] **Step 5: Extract Template 4 to file**

Create `templates/generic-document-processing.md`:

```markdown
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
```

- [ ] **Step 6: Create templates README**

Create `templates/README.md`:

```markdown
# Batch Processing Templates

Ready-to-use templates for common batch processing scenarios. Each template includes:
- Business context (why you'd use this)
- When to use (triggering conditions)
- Complete prompt (ready to paste and use)

## Available Templates

1. **[customer-language-extraction.md](customer-language-extraction.md)** - Extract emotional customer language for marketing copy
2. **[feature-request-aggregation.md](feature-request-aggregation.md)** - Aggregate and rank feature requests by demand
3. **[support-ticket-analysis.md](support-ticket-analysis.md)** - Identify recurring issues and root causes
4. **[generic-document-processing.md](generic-document-processing.md)** - Flexible template for any document type

## How to Use

1. Choose template matching your use case
2. Read the Business Context and When to Use sections
3. Copy the Prompt section
4. Paste into your AI session
5. The AI will offer dry-run mode (recommended for first-time use)
6. Choose sequential or parallel processing
7. Choose merge strategy (if parallel)
8. AI processes all items and reports completion

## Customizing Templates

The generic-document-processing.md template has bracketed placeholders like [DOCUMENT TYPE] and [YOUR CATEGORIES]. Replace these with your specific requirements before pasting the prompt.
```

- [ ] **Step 7: Verify template files created**

Run:
```bash
ls -la templates/
```

Expected output: 5 files (4 templates + README.md)

- [ ] **Step 8: Replace Ready-to-Use Templates section in SKILL.md**

Find the "## Ready-to-Use Templates" section heading in SKILL.md. Replace entire section content with:

```markdown
## Ready-to-Use Templates

Templates moved to `templates/` directory for token efficiency. Each template includes business context, usage guidance, and complete ready-to-paste prompt.

**Available templates:**
- `templates/customer-language-extraction.md` - Extract emotional language for marketing
- `templates/feature-request-aggregation.md` - Aggregate and rank feature requests
- `templates/support-ticket-analysis.md` - Identify recurring technical issues
- `templates/generic-document-processing.md` - Flexible template for any document type

See `templates/README.md` for detailed usage instructions.

**Template structure:**
- Business Context: Why you'd use this template
- When to Use: Triggering conditions
- Prompt: Complete ready-to-paste prompt with extraction rules and output format

**Using templates:**
1. Read template file matching your use case
2. Copy the Prompt section
3. Paste into AI session
4. AI offers dry-run mode (recommended first-time)
5. Choose sequential or parallel mode
6. Choose merge strategy (if parallel)
7. AI processes batch autonomously
```

- [ ] **Step 9: Verify SKILL.md replacement**

Read the Ready-to-Use Templates section to ensure:
- Template references are correct
- Usage instructions are clear
- Token savings is significant (was ~220 lines, now ~25 lines)

- [ ] **Step 10: Commit Task 6**

```bash
git add SKILL.md templates/
git commit -m "refactor: move templates to separate files for token efficiency"
```

---

### Task 7: Add Exponential Backoff for Progress Monitoring

**Files:**
- Modify: `SKILL.md` (Parallel Processing Workflow section ~line 167-245)

**Interfaces:**
- Consumes: Existing 30-second polling from parallel workflow
- Produces: Exponential backoff schedule that adaptive calculation (Task 3) uses

- [ ] **Step 1: Update Progress Monitoring subsection**

Find the "Progress Monitoring:" subsection in Parallel Processing Workflow. Replace with:

```markdown
**Progress Monitoring:**

Coordinator polls todos.md with exponential backoff (reduces token waste):
- Initial: 30s after spawn
- Then: 60s, 120s, 240s (doubles each time, max 240s = 4min)
- Milestone checks: 25%, 50%, 75% completion (always report regardless of backoff)
- Final: When all items `[x]`

Example for 50-item batch (~20min total):
- 30s: "5/50 complete (10%)" - first check
- 90s: "15/50 complete (30%)" - 25% milestone
- 210s: "25/50 complete (50%)" - 50% milestone  
- 450s: "38/50 complete (76%)" - 75% milestone
- 690s: "50/50 complete (100%)" - completion

Reports: "X/Y items complete (Z% - P in-progress)"

**Why backoff:** Early phase has high activity (many claims), late phase is predictable. Checking every 30s for 20min = 40 checks. Backoff = ~8 checks with milestone coverage.
```

- [ ] **Step 2: Verify subsection replacement**

Read the Progress Monitoring subsection to ensure:
- Exponential backoff schedule is clear
- Milestone checks are explicit (override backoff)
- Example calculation is correct
- Rationale for backoff is provided

- [ ] **Step 3: Update Coordinator Role with backoff**

Find the "Coordinator Role:" subsection in Parallel Processing Workflow. Replace the "Monitor progress every 30s" line with:

```markdown
- Monitors progress with exponential backoff (30s, 60s, 120s, 240s max)
- Checks at milestones (25%, 50%, 75%) regardless of backoff
- Reports aggregate completion: "X/Y complete (Z% - P in-progress)"
```

- [ ] **Step 4: Verify Coordinator Role update**

Read the Coordinator Role subsection to ensure:
- Backoff is mentioned
- Milestone override is clear
- Content is concise

- [ ] **Step 5: Update Quick Reference with backoff**

Find the "Parallel Mode Pattern:" in Quick Reference section. Replace step 3 with:

```markdown
3. Coordinator monitors with exponential backoff (30s → 60s → 120s → 240s) + milestone checks (25%, 50%, 75%)
```

- [ ] **Step 6: Verify Quick Reference update**

Read the Parallel Mode Pattern to ensure:
- Backoff schedule is present
- Milestone checks are mentioned
- Format is consistent with other patterns

- [ ] **Step 7: Commit Task 7**

```bash
git add SKILL.md
git commit -m "feat: add exponential backoff for progress monitoring (token efficiency)"
```

---

### Task 8: Add Incremental Results Visibility

**Files:**
- Modify: `SKILL.md` (Parallel Processing Workflow section ~line 167-245)
- Modify: `SKILL.md` (Workflow section ~line 128-165)

**Interfaces:**
- Consumes: Existing insights-N.md isolation from parallel mode
- Produces: Incremental visibility that validation (Task 4) can reference

- [ ] **Step 1: Update Subagent Role with visibility guidance**

Find the "Subagent Role:" subsection in Parallel Processing Workflow. After the "Reports completion when done" line, add:

```markdown
- insights-N.md visible to user immediately (no wait for merge)
- User can monitor quality by reading any insights-N.md at any time
- Allows early error detection without stopping processing
```

- [ ] **Step 2: Verify Subagent Role update**

Read the Subagent Role subsection to ensure:
- Visibility guidance is clear
- User can read insights-N.md files during processing
- Connection to early error detection is mentioned

- [ ] **Step 3: Add incremental visibility workflow**

Find the "Parallel Processing Workflow" section heading. After the "Failure Recovery:" subsection, add new subsection:

```markdown
**Incremental Results Visibility:**

Unlike sequential mode (where insights.md updates after each item), parallel mode creates isolated insights-N.md files per subagent. These files are immediately readable - no need to wait for merge.

**Monitoring during processing:**
1. List insights files: `ls insights-*.md`
2. Read any subagent's progress: `cat insights-3.md`
3. Check quality of findings
4. If errors found in insights-N.md:
   - Identify which subagent (N)
   - Check todos.md for items claimed by agent-N
   - Stop that subagent if possible
   - Update context.md with clearer rules
   - Reset agent-N's items to `[ ]` for retry

**Benefits:**
- Early quality detection (don't wait for all 50 items to finish)
- Per-subagent quality check (identify problematic extraction patterns)
- User can track progress by checking file sizes: `ls -lh insights-*.md`
```

- [ ] **Step 4: Verify workflow addition**

Read the new Incremental Results Visibility subsection to ensure:
- Workflow is actionable
- Commands are correct (ls, cat)
- Error detection integration is clear
- Benefits are explicit

- [ ] **Step 5: Update Workflow section with visibility note**

Find the "Parallel Processing Workflow" subsection in the Workflow section. After the "Subagent Role:" paragraph, add:

```markdown
**Visibility:** insights-N.md files are immediately readable during processing. User can monitor quality at any time without waiting for merge completion.
```

- [ ] **Step 6: Verify Workflow update**

Read the Parallel Processing Workflow subsection to ensure:
- Visibility note is concise
- Placement makes sense (after Subagent Role)

- [ ] **Step 7: Update validation (from Task 4) with incremental visibility**

Find the "Early Detection (recommended):" subsection in Validation section (added in Task 4). After the "After first 10 items processed" paragraph, add:

```markdown
**Parallel mode early detection:**

After first 10 items complete (check `grep -c '^\- \[x\]' todos.md`):
1. List insights files: `ls insights-*.md`
2. Read 2-3 insights-N.md files randomly
3. Validate findings per standard checks
4. If errors found: stop processing, update context.md, reset todos.md, restart
```

- [ ] **Step 8: Verify validation update**

Read the Early Detection subsection to ensure:
- Parallel mode early detection is integrated
- Commands are correct
- Workflow matches sequential mode approach

- [ ] **Step 9: Commit Task 8**

```bash
git add SKILL.md
git commit -m "feat: add incremental results visibility for parallel mode"
```

---

### Task 9: Self-Review and Consistency Check

**Files:**
- Read: `SKILL.md` (entire file)
- Read: `templates/*.md` (all template files)

**Interfaces:**
- Consumes: All previous task changes to SKILL.md and templates
- Produces: Consistency verification before final review

- [ ] **Step 1: Check for placeholder violations**

Search SKILL.md for placeholder patterns:
```bash
grep -i 'tbd\|todo\|fill in\|implement later' SKILL.md
```

Expected: No matches (exit code 1)

- [ ] **Step 2: Check for broken cross-references**

Read SKILL.md sections that reference other sections. Verify:
- "Detection Pattern" references "Parallel Processing Workflow" - check exists
- "Validation" references "Dry-run mode" - check Task 2 added it
- "Troubleshooting" references "Resume workflow" - check Task 1 added it
- "Quick Reference" patterns match full workflow sections

- [ ] **Step 3: Verify internal consistency**

Check adaptive calculation formula appears consistently:
```bash
grep -n 'min(ceil' SKILL.md
```

Expected: 3+ matches with same formula `min(ceil(estimated_total / 20), 5)` or `min(ceil(T * item_count / 20), 5)` from Task 3

- [ ] **Step 4: Verify merge strategy consistency**

Check merge strategy appears in Detection Pattern, Parallel Processing Workflow, and templates:
```bash
grep -n 'merge strategy\|Category-based\|Chronological\|Frequency' SKILL.md templates/*.md
```

Expected: Multiple matches across files showing all three strategies

- [ ] **Step 5: Check section ordering**

Read SKILL.md table of contents (section headings). Expected order:
1. Overview
2. When to Use
3. Parallel vs Sequential (with adaptive calculation)
4. Detection Pattern (with dry-run, resume, merge strategy)
5. Quick Reference
6. Autonomous Processing
7. Workflow (with dry-run, early validation)
8. Parallel Processing Workflow (with resume, backoff, visibility)
9. Processing Loop
10. Handling Context Compaction
11. Red Flags
12. Common Mistakes
13. Ready-to-Use Templates (now references templates/ dir)
14. Validation (with sampling methodology)
15. Troubleshooting (with parallel resume)
16. Why This Works

- [ ] **Step 6: Verify token count reduction**

Count lines in original vs updated SKILL.md:
```bash
git show HEAD~9:SKILL.md | wc -l  # Original before Task 1
wc -l SKILL.md                     # Current after all tasks
```

Expected: Current should be ~150-200 lines fewer (templates moved to separate files)

- [ ] **Step 7: Check template file completeness**

Verify all template files exist and have content:
```bash
for f in templates/*.md; do echo "$f: $(wc -l < "$f") lines"; done
```

Expected: Each template file should have 50-100 lines, README.md should have 20-40 lines

- [ ] **Step 8: Verify no duplicate content**

Read SKILL.md Ready-to-Use Templates section. Ensure:
- No full template prompts inline (should be references only)
- Template list matches templates/ directory contents
- Usage instructions don't duplicate template README.md

- [ ] **Step 9: Check all cross-task integrations**

Verify tasks that reference each other:
- Task 1 (resume) integrated with Task 3 (adaptive calculation) - check "Resuming Interrupted" subsection has adaptive formula
- Task 2 (dry-run) integrated with Task 4 (validation) - check validation references dry-run
- Task 4 (validation) integrated with Task 8 (visibility) - check validation mentions incremental visibility for parallel
- Task 5 (merge strategy) integrated with Task 6 (templates) - check templates reference merge strategies

- [ ] **Step 10: Final read-through for clarity**

Read entire SKILL.md from top to bottom. Check:
- Each section flows logically to next
- New content integrates smoothly with existing content
- No orphaned references to removed content
- Formatting is consistent (headers, code blocks, lists)

- [ ] **Step 11: Commit Task 9**

```bash
git add SKILL.md templates/
git commit -m "docs: self-review and consistency check - all improvements integrated"
```

---

## Plan Complete

All high and medium priority improvements implemented:

**High priority:**
1. ✅ Parallel resume workflow (Task 1)
2. ✅ Dry-run mode (Task 2)
3. ✅ Adaptive agent calculation (Task 3)
4. ✅ Validation sampling methodology (Task 4)

**Medium priority:**
5. ✅ Configurable merge strategy (Task 5)
6. ✅ Templates moved to separate files (Task 6)
7. ✅ Exponential backoff for monitoring (Task 7)
8. ✅ Incremental results visibility (Task 8)

**Quality:**
9. ✅ Self-review and consistency check (Task 9)

**Token savings:** ~150-200 lines removed from SKILL.md (templates moved to separate files)

**Backward compatibility:** All existing workflows (sequential and original parallel) continue to work unchanged. All improvements are additive.
