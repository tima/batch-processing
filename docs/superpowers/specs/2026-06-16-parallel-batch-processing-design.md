# Parallel Batch Processing Design

**Date:** 2026-06-16  
**Status:** Proposed

## Problem

Current batch-processing skill processes items sequentially. For large batches (50+ items), this takes 1-2 hours. Users want parallel processing to reduce wall-clock time by distributing work across multiple subagents.

## Goals

- Add parallel processing mode to batch-processing skill
- Reduce processing time for large batches from hours to minutes
- Maintain existing sequential mode for simplicity when appropriate
- Preserve fault tolerance - handle subagent failures gracefully
- Prevent duplicate work through proper coordination

## Non-Goals

- Real-time collaboration between subagents (each works independently)
- Dynamic scaling during processing (subagent count fixed at start)
- Cross-batch optimization or caching

## Architecture

### Two Execution Modes

**Sequential Mode (existing):**
- Single AI processes items one by one from todos.md
- Updates todos.md after each item: `[ ]` → `[x]`
- Appends findings to single insights.md
- Resumes from todos.md after context reset

**Parallel Mode (new):**
- Coordinator spawns N subagents (calculated: `min(ceil(item_count / 10), 5)`)
- All subagents work-steal from shared todos.md
- Each subagent writes to isolated insights-N.md
- Coordinator polls progress every 30s, reports aggregate completion
- After all items complete, coordinator merges insights-*.md → insights.md

### Mode Selection

At setup, always offer both modes:
- "Process sequentially (simpler) or in parallel (faster, uses more resources)?"
- User chooses based on urgency, batch size, and resource availability

### File Structure (Parallel Mode)

```
context.md          # Shared - all subagents read for extraction rules
todos.md            # Shared - all subagents read/write with three states
insights-1.md       # Subagent 1's findings (isolated)
insights-2.md       # Subagent 2's findings (isolated)
...
insights-N.md       # Subagent N's findings (isolated)
insights.md         # Final merged output (created at end)
```

### Three-State Item Tracking

To prevent race conditions, todos.md items have three states:

```markdown
- [ ] transcript_01.txt              # Available - unclaimed
- [>] transcript_02.txt (agent-2)    # In-progress by subagent 2
- [x] transcript_03.txt              # Complete
```

**Claiming protocol:**
1. Subagent reads todos.md
2. Finds first `[ ]` item (skips `[>]` and `[x]`)
3. Immediately changes to `[>] item.txt (agent-N)` - atomic claim
4. Processes item per context.md rules
5. Appends findings to insights-N.md
6. Changes to `[x]` - marks complete

This eliminates race conditions - two subagents cannot claim the same item.

## Components

### 1. Coordinator

**Responsibilities:**
- Calculate subagent count: `N = min(ceil(item_count / 10), 5)`
- Create context.md, todos.md, insights-1.md through insights-N.md (empty)
- Spawn N subagents with work-stealing instructions
- Monitor progress every 30s
- Detect subagent failures and recover
- Merge insights when all items complete
- Clean up intermediate files

**Progress monitoring:**
```
every 30s:
  read todos.md
  count [ ], [>], [x] items
  report: "X/Y items complete (Z in-progress)"
```

**Failure detection:**
```
track which subagents are alive
if subagent died:
  find its [>] items in todos.md
  change back to [ ] for retry by other subagents
```

**Merge logic:**
```
merge_insights():
  findings_by_category = {}
  
  for i in 1..N:
    read insights-{i}.md
    parse by category (Frustration, Confusion, etc.)
    append to findings_by_category
  
  write insights.md:
    for each category:
      write category header
      write all findings for that category (from all subagents)
      
  cleanup: delete insights-1.md through insights-N.md
```

### 2. Subagent

**Responsibilities:**
- Read context.md once at startup (extraction rules)
- Execute work-stealing loop until no items remain
- Write findings to isolated insights-N.md
- Report completion when done

**Work-stealing loop:**
```
work_stealing_loop(agent_id):
  read context.md (once at start)
  
  while True:
    read todos.md
    find first [ ] item
    if none found: 
      exit (no more work)
    
    claim item: 
      change [ ] to [>] item.txt (agent-{agent_id})
    
    process item per context.md extraction rules
    
    append findings to insights-{agent_id}.md:
      - exact quote or extracted data
      - source filename
      - brief context (1 sentence)
    
    mark complete:
      change [>] to [x]
```

### 3. Merge Component

Combines findings from insights-1.md through insights-N.md into final insights.md.

**Merge strategy:**
- Group by category (Frustration, Confusion, Feature Requests, etc.)
- Preserve source attribution (filename + context)
- Maintain chronological order within categories (by item order in todos.md)
- Remove duplicates if any (shouldn't exist due to claiming protocol)

**Output format:**
```markdown
## Frustration
- "Quote from customer" (file_01.txt - context)
- "Another quote" (file_15.txt - context)
- "Third quote" (file_23.txt - context)

## Confusion
- "Confusing statement" (file_07.txt - context)
...
```

## Data Flow

### Parallel Mode Flow

```
User request
  ↓
AI detects batch processing (5+ items)
  ↓
AI offers: "Sequential or parallel?"
  ↓
User chooses parallel
  ↓
Coordinator:
  - Counts items
  - Calculates N = min(ceil(count / 10), 5)
  - Creates context.md, todos.md, insights-1..N.md
  - Informs user: "Using N subagents for M items"
  ↓
Spawns N subagents in parallel
  ↓
Each subagent:
  - Reads context.md (once)
  - Loop:
    * Reads todos.md
    * Claims first [ ] item → [>]
    * Processes item
    * Appends to insights-N.md
    * Marks [x]
    * Repeats until no [ ] items
  - Exits when done
  ↓
Coordinator (parallel with subagents):
  - Every 30s:
    * Reads todos.md
    * Counts [x] vs total
    * Reports: "X/Y complete (Z in-progress)"
  - Detects failures:
    * If subagent dies
    * Finds [>] items from that agent
    * Changes back to [ ]
  ↓
When all items [x]:
  - Waits for subagents to finish
  - Merges insights-1..N.md → insights.md
  - Deletes intermediate files
  - Reports completion: "Processed Y items, output in insights.md"
```

### File Interaction Pattern

```
context.md:  [Coordinator writes] → [All subagents read-only]
todos.md:    [Coordinator creates] → [All subagents read/write for claiming]
insights-N:  [Subagent N write-only] → [Coordinator reads for merge]
insights.md: [Coordinator writes final] → [User reads result]
```

## Error Handling

### 1. Subagent Failure During Processing

**Scenario:** Subagent crashes or times out while processing items.

**Detection:**
- Coordinator tracks subagent process IDs or execution status
- If subagent stops responding or crashes, coordinator detects it

**Recovery:**
1. Find all `[>] item.txt (agent-N)` entries where N = failed agent ID
2. Change back to `[ ]` (available for retry)
3. Other running subagents will claim these items naturally (work-stealing)
4. No need to spawn replacement subagent - existing agents handle the load

**Result:** No data lost, minimal delay (other agents pick up the work).

### 2. File Conflicts on todos.md

**Scenario:** Two subagents try to claim the same item simultaneously.

**Mitigation:**
- File write operations are typically atomic at OS level for small edits
- Race window is milliseconds between read and claim
- If conflict occurs, subagent retries: re-read todos.md, find next `[ ]` item

**Fallback:**
- If repeated conflicts (unlikely), subagent reports error
- Coordinator can throttle that subagent (check todos.md less frequently)

**Result:** Very rare occurrence, self-correcting through retry.

### 3. Merge Failure

**Scenario:** Coordinator crashes during merge step after all items processed.

**State at failure:**
- todos.md shows all items `[x]` (processing complete)
- insights-1.md through insights-N.md exist and contain findings
- insights.md does not exist or is partial

**Recovery:**
1. User or new coordinator session can retry merge
2. Read all insights-N.md files (preserved)
3. Combine by category
4. Write final insights.md

**Result:** No reprocessing needed, just re-run merge step.

### 4. Partial Item Processing

**Scenario:** Subagent crashes mid-item (after claiming `[>]` but before marking `[x]`).

**Detection:**
- Item stuck in `[>] item.txt (agent-N)` state
- Subagent N is dead (coordinator detected failure)

**Recovery:**
1. Coordinator changes `[>]` back to `[ ]`
2. Another subagent claims and reprocesses the item
3. Previous partial findings in insights-N.md are ignored (findings include source filename, so duplicates can be detected if any)

**Result:** Item processed fully by new subagent, minimal duplicate work.

### 5. Quality Issues in Parallel Output

**Scenario:** One subagent produces poor-quality extractions (too broad, too narrow, wrong category).

**Detection:**
- User spot-checks insights.md after completion
- Identifies low-quality section

**Recovery:**
1. Before cleanup, check which insights-N.md file had poor quality
2. Identify which items that subagent processed (review todos.md history if needed)
3. Manually uncheck those items in todos.md: change `[x]` back to `[ ]`
4. Update context.md with clearer extraction rules
5. Re-run batch processing (sequential or parallel) for unchecked items only

**Result:** Only affected items reprocessed, not entire batch.

## Testing Strategy

### 1. Sequential Mode Regression Tests

Ensure existing functionality still works after adding parallel mode:
- Process 5-10 items sequentially
- Test context reset/resume
- Test all four templates (customer language, feature requests, support tickets, generic)
- Verify todos.md checkpointing: `[ ]` → `[x]`

**Success criteria:** No behavior changes in sequential mode.

### 2. Parallel Mode Basic Tests

**Small batch (10 items):**
- Should spawn 1 subagent (ceil(10/10) = 1)
- Verify work-stealing loop executes
- Verify insights-1.md → insights.md merge works
- Verify all 10 items marked `[x]`

**Medium batch (25 items):**
- Should spawn 3 subagents (ceil(25/10) = 3, under max)
- Verify all 3 subagents process items
- Verify progress reports every 30s: "X/25 complete (Y in-progress)"
- Verify merge combines findings from insights-1.md, insights-2.md, insights-3.md correctly
- Verify category grouping preserved

**Large batch (50 items):**
- Should spawn 5 subagents (ceil(50/10) = 5, at max limit)
- Verify load balancing: some subagents finish before others
- Verify no duplicate processing (each item processed once)
- Verify category merging preserves all findings from all 5 subagents

**Success criteria:** All items processed exactly once, findings merged correctly by category.

### 3. Race Condition Tests

**Simultaneous claim:**
- Launch 2 subagents with only 1 unclaimed item left
- Verify only one claims it (one gets `[>]`, other sees `[>]` and exits)
- Verify no duplicate entries in final insights.md

**Success criteria:** No duplicate processing, clean claim/release.

### 4. Failure Recovery Tests

**Subagent crash mid-item:**
1. Launch parallel batch
2. Kill one subagent after it claims `[>]` but before `[x]`
3. Verify coordinator detects failure
4. Verify item changes back to `[ ]`
5. Verify another subagent picks it up and completes it

**Success criteria:** Item processed successfully, no lost work.

**Merge failure:**
1. Process batch to completion (all items `[x]`)
2. Simulate crash during merge step (before insights.md written)
3. Verify insights-1.md through insights-N.md files preserved
4. Manually or automatically retry merge
5. Verify insights.md created correctly

**Success criteria:** Merge recoverable without reprocessing.

### 5. Quality Tests

**Category preservation:**
- Process items that have findings in multiple categories (e.g., Frustration + Confusion)
- Verify merge preserves all categories
- Verify no duplicate entries within categories

**Source attribution:**
- Verify every finding in insights.md includes source filename
- Verify findings from different subagents are distinguishable pre-merge
- Verify merge preserves source attribution

**Success criteria:** All findings traceable to source files, categories intact.

## Implementation Notes

### Skill Structure Changes

Current skill has these sections:
- Overview
- When to Use
- Detection Pattern
- Quick Reference
- Autonomous Processing
- Workflow
- Processing Loop
- Handling Context Compaction
- Red Flags
- Common Mistakes
- Ready-to-Use Templates
- Troubleshooting
- Validation
- Why This Works

**New sections to add:**
- **Parallel vs Sequential** (after "When to Use") - explains both modes, when to choose each
- **Parallel Processing Workflow** (after "Workflow") - coordinator, subagent, merge steps
- **Work-Stealing Protocol** (in "Parallel Processing Workflow") - three-state items, claiming
- **Progress Monitoring** (in "Parallel Processing Workflow") - 30s polling, aggregate reporting
- **Parallel Troubleshooting** (add to "Troubleshooting") - subagent failures, merge issues

**Sections to update:**
- **Detection Pattern** - add parallel mode offer after setup confirmation
- **Quick Reference** - add parallel mode summary
- **Validation** - add merge validation checks

### Backward Compatibility

Sequential mode remains default and unchanged. Users who don't choose parallel will see no behavior differences. Existing workflows, templates, and patterns all work as before.

## Open Questions

None - design is complete and approved for implementation.

## Success Metrics

- Sequential mode behavior unchanged (regression tests pass)
- Parallel mode reduces wall-clock time for 50-item batch from ~100min to ~20min
- No duplicate processing (each item processed exactly once)
- Fault tolerance verified (subagent crash recovery works)
- Merge preserves all findings with correct categorization
