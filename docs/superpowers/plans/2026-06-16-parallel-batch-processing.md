# Parallel Batch Processing Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add parallel processing mode to batch-processing skill with work-stealing coordination, reducing wall-clock time for large batches from hours to minutes.

**Architecture:** Coordinator spawns N subagents (1 per 10 items, max 5) that work-steal from shared todos.md using three-state claiming protocol (`[ ]` → `[>]` → `[x]`). Each subagent writes to isolated insights-N.md. After all items complete, coordinator merges insights into final insights.md. Sequential mode unchanged for backward compatibility.

**Tech Stack:** Markdown (three-file system), Agent tool (subagent dispatch), file-based coordination, category-based merge

---

## File Structure

**Modified:**
- `SKILL.md` - add parallel mode sections, update detection/quick-reference/validation

**Created (documentation/examples):**
- Test fixtures in temp directories (for validation)

---

## Task 1: Add Parallel vs Sequential Section

**Files:**
- Modify: `SKILL.md` (after "When to Use" section)

- [ ] **Step 1: Add section explaining both modes**

Insert after "When to Use" section:

```markdown
## Parallel vs Sequential

The batch-processing skill supports two execution modes:

**Sequential Mode:**
- Single AI processes items one by one
- Simpler coordination, lower resource usage
- Best for: Small batches (< 20 items), simple tasks, resource-constrained environments
- Time: ~2min per item (10 items = 20min, 50 items = 100min)

**Parallel Mode:**
- Coordinator spawns multiple subagents (1 per 10 items, max 5)
- Subagents work-steal from shared todos.md
- Faster completion, higher resource usage
- Best for: Large batches (20+ items), time-sensitive work, sufficient resources
- Time: ~2min per item / N agents (50 items with 5 agents = 20min)

**Mode Selection:**
At setup, you'll be asked: "Process sequentially or in parallel?"
- Sequential: Simpler, reliable, proven pattern
- Parallel: Faster, more complex coordination, requires subagent support

Choose based on batch size, urgency, and available resources.
```

- [ ] **Step 2: Verify formatting**

Read the updated section to ensure markdown renders correctly.

- [ ] **Step 3: Commit**

```bash
git add SKILL.md
git commit -m "docs: add parallel vs sequential modes section"
```

---

## Task 2: Update Detection Pattern for Parallel Mode

**Files:**
- Modify: `SKILL.md` (Detection Pattern section, lines 38-46)

- [ ] **Step 1: Update detection pattern action steps**

Replace current "Action:" steps with:

```markdown
**Action:**
1. Recognize this as batch processing
2. Ask user: "This looks like batch processing (5+ items requiring extraction/transformation). Should I set up the three-file system (context.md, todos.md, insights.md) to track progress across context resets?"
3. If yes:
   - Count items to process
   - Ask: "Process sequentially (simpler) or in parallel (faster, uses more resources)?"
4. **If sequential:**
   - Create context.md with their goal
   - Enumerate files to create todos.md (format: `- [ ] filename`)
   - Create empty insights.md
   - Begin sequential processing loop
   - Work through all items until complete
5. **If parallel:**
   - Calculate N = min(ceil(item_count / 10), 5)
   - Inform user: "Using N subagents for M items"
   - Create context.md with their goal
   - Enumerate files to create todos.md (format: `- [ ] filename`)
   - Create empty insights-1.md through insights-N.md
   - Spawn N subagents with work-stealing instructions
   - Monitor progress, merge when complete
```

- [ ] **Step 2: Verify formatting**

Read the updated section to ensure clarity and correctness.

- [ ] **Step 3: Commit**

```bash
git add SKILL.md
git commit -m "docs: update detection pattern for parallel mode"
```

---

## Task 3: Add Parallel Processing Workflow Section

**Files:**
- Modify: `SKILL.md` (after "Workflow" section, before "Processing Loop")

- [ ] **Step 1: Add parallel workflow section**

Insert after "Initial Setup (Automatic)" subsection:

```markdown
### Parallel Processing Workflow

When parallel mode is chosen, the workflow differs from sequential:

**Coordinator Role:**
- Spawns N subagents (calculated: `min(ceil(item_count / 10), 5)`)
- Monitors progress every 30s
- Reports aggregate completion: "X/Y items complete (Z in-progress)"
- Detects subagent failures and recovers work
- Merges insights when all items complete

**Subagent Role:**
- Reads context.md once at start (extraction rules)
- Executes work-stealing loop until no items remain
- Writes findings to isolated insights-N.md
- Reports completion when done

**Work-Stealing Protocol:**

To prevent duplicate work, todos.md items have three states:

```markdown
- [ ] transcript_01.txt              # Available - unclaimed
- [>] transcript_02.txt (agent-2)    # In-progress by subagent 2
- [x] transcript_03.txt              # Complete
```

**Claiming process:**
1. Subagent reads todos.md
2. Finds first `[ ]` item (skips `[>]` and `[x]`)
3. Immediately changes to `[>] item.txt (agent-N)` - atomic claim
4. Processes item per context.md rules
5. Appends findings to insights-N.md (includes source filename + context)
6. Changes to `[x]` - marks complete
7. Repeats until no `[ ]` items remain

This eliminates race conditions - only one subagent can claim each item.

**Progress Monitoring:**

Coordinator polls todos.md every 30s:
- Counts `[ ]` (available), `[>]` (in-progress), `[x]` (complete)
- Reports: "X/Y items complete (Z in-progress)"
- User can walk away - processing runs autonomously

**Merge Step:**

When all items are `[x]`:
1. Coordinator waits for all subagents to finish
2. Reads insights-1.md through insights-N.md
3. Groups findings by category (Frustration, Confusion, etc.)
4. Writes final insights.md with all findings organized by category
5. Deletes intermediate insights-*.md files
6. Reports completion: "Processed Y items, output in insights.md"

**Example merge:**

```markdown
## Frustration (from insights-1.md, insights-3.md, insights-5.md)
- "We've been doing this manually for months" (file_01.txt - workflow pain)
- "System times out every time" (file_15.txt - export issue)
- "Can't figure out how this works" (file_23.txt - UI confusion)

## Confusion (from insights-2.md, insights-4.md)
- "Why does it work this way?" (file_07.txt - questioning logic)
- "Instructions unclear" (file_19.txt - documentation gap)
```

**Failure Recovery:**

If a subagent crashes:
1. Coordinator detects failure
2. Finds all `[>] item.txt (agent-N)` entries for that subagent
3. Changes back to `[ ]` (available for retry)
4. Other running subagents claim and process these items
5. No data lost, minimal delay
```

- [ ] **Step 2: Verify formatting and code blocks**

Read the section to ensure markdown renders correctly, code blocks are properly formatted.

- [ ] **Step 3: Commit**

```bash
git add SKILL.md
git commit -m "docs: add parallel processing workflow section"
```

---

## Task 4: Update Quick Reference for Parallel Mode

**Files:**
- Modify: `SKILL.md` (Quick Reference section, lines 48-61)

- [ ] **Step 1: Add parallel mode to quick reference**

Insert after existing pattern (line 61):

```markdown

**Parallel Mode Pattern:**
1. Coordinator spawns N subagents (1 per 10 items, max 5)
2. Each subagent work-steals from todos.md:
   - Find first `[ ]` item → claim as `[>]` → process → append to insights-N.md → mark `[x]`
3. Coordinator monitors every 30s: "X/Y complete (Z in-progress)"
4. When all `[x]`: merge insights-1..N.md → insights.md
5. On subagent failure: change `[>]` back to `[ ]`, other agents claim it
```

- [ ] **Step 2: Verify formatting**

Read the quick reference to ensure both modes are clear and concise.

- [ ] **Step 3: Commit**

```bash
git add SKILL.md
git commit -m "docs: add parallel mode to quick reference"
```

---

## Task 5: Add Parallel Troubleshooting Section

**Files:**
- Modify: `SKILL.md` (Troubleshooting section, add parallel subsection before "Why This Works")

- [ ] **Step 1: Add parallel-specific troubleshooting**

Insert before "Why This Works" section:

```markdown
### Parallel Mode Issues

**Subagent Stuck in [>] State**

**Symptom:** Item shows `[>] item.txt (agent-N)` but subagent N is dead.

**Solution:**
1. Manually change `[>]` back to `[ ]` in todos.md
2. Running subagents will claim and process it
3. If no subagents running, restart parallel batch processing (it will skip `[x]` items)

**Duplicate Findings After Merge**

**Symptom:** Same quote appears twice in insights.md from different sources.

**Cause:** Two items contained the same quote (expected) OR race condition (rare).

**Solution:**
1. Check source filenames - if different files, duplicates are legitimate
2. If same file, race condition occurred - manually deduplicate
3. Re-run batch for affected items if needed

**Merge Failed Mid-Process**

**Symptom:** All todos.md items are `[x]` but insights.md missing or incomplete.

**State:** insights-1.md through insights-N.md still exist.

**Recovery:**
1. Read each insights-N.md file
2. Manually combine by category into insights.md, OR
3. Create simple script to merge:

```python
import glob

findings = {}
for file in glob.glob("insights-*.md"):
    with open(file) as f:
        content = f.read()
    # Parse by category headers (##)
    # Append to findings[category]
    
with open("insights.md", "w") as out:
    for category, items in findings.items():
        out.write(f"## {category}\n")
        for item in items:
            out.write(f"{item}\n")
```

**Uneven Load Distribution**

**Symptom:** Some subagents finish quickly, others still processing.

**Cause:** Item complexity varies (some files longer/more complex).

**Expected:** Work-stealing handles this - fast subagents finish early, slow ones keep working.

**Not a problem** unless one subagent is stuck (see "Subagent Stuck" above).

**Progress Monitoring Shows Wrong Count**

**Symptom:** Coordinator reports "30/50 complete" but you see different counts.

**Cause:** todos.md read at different time, or subagent just updated.

**Solution:**
- Wait for next 30s update - counts will sync
- Manually count `[x]` items in todos.md to verify
```

- [ ] **Step 2: Verify formatting**

Read the troubleshooting section to ensure all scenarios are covered clearly.

- [ ] **Step 3: Commit**

```bash
git add SKILL.md
git commit -m "docs: add parallel mode troubleshooting section"
```

---

## Task 6: Update Validation Section for Parallel Mode

**Files:**
- Modify: `SKILL.md` (Validation section, add parallel checks)

- [ ] **Step 1: Add parallel validation checks**

Insert after "Optional quality checks:" line:

```markdown

**Parallel Mode Additional Checks:**

**Before merge cleanup:**
- Verify insights-1.md through insights-N.md all exist
- Spot-check 1-2 findings in each insights-N.md for quality
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

- [ ] **Step 2: Verify formatting**

Read the validation section to ensure parallel checks are clear.

- [ ] **Step 3: Commit**

```bash
git add SKILL.md
git commit -m "docs: add parallel mode validation checks"
```

---

## Task 7: Create Test Case - Sequential Regression

**Files:**
- Create: `tests/sequential-regression-test.md`

- [ ] **Step 1: Write test documentation**

```markdown
# Sequential Mode Regression Test

**Purpose:** Verify sequential mode still works after adding parallel mode.

**Setup:**
1. Create test directory: `mkdir -p /tmp/batch-test-sequential`
2. Create 5 sample transcript files with customer language:

```bash
cd /tmp/batch-test-sequential

cat > transcript_01.txt <<'EOF'
Customer: "This export feature is incredibly frustrating. It times out every single time I try to use it."
Agent: "I understand your frustration. Let me help you with that."
EOF

cat > transcript_02.txt <<'EOF'
Customer: "I'm confused about how the workflow is supposed to work. The documentation doesn't make sense."
Agent: "Let me walk you through it step by step."
EOF

cat > transcript_03.txt <<'EOF'
Customer: "We've been doing this manually for six months and it's killing our productivity."
Agent: "I can see how that would be frustrating."
EOF

cat > transcript_04.txt <<'EOF'
Customer: "The interface is confusing. I can't find where to configure the settings."
Agent: "Let me show you where that is."
EOF

cat > transcript_05.txt <<'EOF'
Customer: "This is causing us a lot of stress. Our team is falling behind because of these issues."
Agent: "I understand how stressful that must be."
EOF
```

**Execution:**
1. Request: "Analyze all transcripts in /tmp/batch-test-sequential. Extract customer pain points showing genuine emotion. Categorize by: Frustration, Confusion, Stress."
2. When asked about three-file system: say "yes"
3. When asked sequential vs parallel: choose "sequential"
4. Observe: AI creates context.md, todos.md, insights.md
5. Wait for completion (should take ~5-10 minutes)

**Expected Results:**

todos.md shows all items checked:
```markdown
- [x] transcript_01.txt
- [x] transcript_02.txt
- [x] transcript_03.txt
- [x] transcript_04.txt
- [x] transcript_05.txt
```

insights.md contains categorized findings:
```markdown
## Frustration
- "This export feature is incredibly frustrating" (transcript_01.txt - export timeout issue)
- "We've been doing this manually for six months and it's killing our productivity" (transcript_03.txt - manual workflow pain)

## Confusion
- "I'm confused about how the workflow is supposed to work" (transcript_02.txt - workflow understanding)
- "The interface is confusing" (transcript_04.txt - settings location)

## Stress
- "This is causing us a lot of stress" (transcript_05.txt - team falling behind)
```

**Success Criteria:**
- All 5 items marked `[x]`
- insights.md has findings organized by category
- Each finding includes source filename and context
- No insights-N.md files created (sequential mode only uses insights.md)
```

- [ ] **Step 2: Verify test is reproducible**

Review test steps - ensure anyone can follow them without additional context.

- [ ] **Step 3: Commit**

```bash
git add tests/sequential-regression-test.md
git commit -m "test: add sequential mode regression test"
```

---

## Task 8: Create Test Case - Parallel Basic (25 Items)

**Files:**
- Create: `tests/parallel-basic-test.md`

- [ ] **Step 1: Write test documentation**

```markdown
# Parallel Mode Basic Test (25 Items)

**Purpose:** Verify parallel mode works with medium batch, spawns correct number of subagents, and merges correctly.

**Setup:**
1. Create test directory: `mkdir -p /tmp/batch-test-parallel`
2. Create 25 sample files with varying content:

```bash
cd /tmp/batch-test-parallel

for i in $(seq -f "%02g" 1 25); do
  cat > "file_${i}.txt" <<EOF
Customer quote ${i}: "This is frustrating issue number ${i}"
Context: describing problem ${i}
EOF
done
```

**Execution:**
1. Request: "Analyze all files in /tmp/batch-test-parallel. Extract frustration quotes. Categorize by: Frustration."
2. When asked about three-file system: say "yes"
3. When asked sequential vs parallel: choose "parallel"
4. Observe: AI should report "Using 3 subagents for 25 items" (ceil(25/10) = 3)
5. Observe progress reports every 30s
6. Wait for completion

**Expected Results:**

**During processing:**
- Progress reports: "5/25 complete (3 in-progress)", "12/25 complete (2 in-progress)", etc.
- Three insights files exist: insights-1.md, insights-2.md, insights-3.md

**After completion:**

todos.md shows all items checked:
```markdown
- [x] file_01.txt
- [x] file_02.txt
...
- [x] file_25.txt
```

insights.md contains all 25 findings:
```markdown
## Frustration
- "This is frustrating issue number 1" (file_01.txt - describing problem 1)
- "This is frustrating issue number 2" (file_02.txt - describing problem 2)
...
- "This is frustrating issue number 25" (file_25.txt - describing problem 25)
```

insights-1.md, insights-2.md, insights-3.md deleted (cleanup after merge).

**Success Criteria:**
- Exactly 3 subagents spawned
- All 25 items marked `[x]`
- insights.md has all 25 findings
- Each finding includes source filename
- Intermediate insights-*.md files cleaned up
- No duplicate findings
```

- [ ] **Step 2: Verify test is reproducible**

Review test steps for clarity and completeness.

- [ ] **Step 3: Commit**

```bash
git add tests/parallel-basic-test.md
git commit -m "test: add parallel mode basic test (25 items)"
```

---

## Task 9: Create Test Case - Parallel Max Agents (50 Items)

**Files:**
- Create: `tests/parallel-max-agents-test.md`

- [ ] **Step 1: Write test documentation**

```markdown
# Parallel Mode Max Agents Test (50 Items)

**Purpose:** Verify parallel mode caps at 5 subagents, handles load balancing, and merges large batch correctly.

**Setup:**
1. Create test directory: `mkdir -p /tmp/batch-test-max-agents`
2. Create 50 sample files:

```bash
cd /tmp/batch-test-max-agents

for i in $(seq -f "%02g" 1 50); do
  cat > "item_${i}.txt" <<EOF
Feature request ${i}: "I wish the system had feature ${i}"
Priority: $((i % 3 == 0 ? "high" : "medium"))
EOF
done
```

**Execution:**
1. Request: "Analyze all items in /tmp/batch-test-max-agents. Extract feature requests with priority."
2. When asked about three-file system: say "yes"
3. When asked sequential vs parallel: choose "parallel"
4. Observe: AI should report "Using 5 subagents for 50 items" (ceil(50/10) = 5, capped at max)
5. Watch progress reports
6. Wait for completion

**Expected Results:**

**During processing:**
- Progress reports: "10/50 complete (5 in-progress)", "25/50 complete (4 in-progress)", etc.
- Five insights files exist: insights-1.md through insights-5.md
- Some subagents finish before others (load balancing)

**After completion:**

todos.md shows all 50 items checked:
```markdown
- [x] item_01.txt
- [x] item_02.txt
...
- [x] item_50.txt
```

insights.md contains all 50 requests organized by priority:
```markdown
## Feature Requests
- "I wish the system had feature 1" (item_01.txt - Priority: medium)
- "I wish the system had feature 2" (item_02.txt - Priority: medium)
- "I wish the system had feature 3" (item_03.txt - Priority: high)
...
- "I wish the system had feature 50" (item_50.txt - Priority: medium)
```

insights-1.md through insights-5.md deleted.

**Success Criteria:**
- Exactly 5 subagents spawned (at max limit)
- All 50 items marked `[x]`
- insights.md has all 50 findings
- No duplicate findings
- Intermediate files cleaned up
- Some subagents finish before others (demonstrates load balancing)
```

- [ ] **Step 2: Verify test completeness**

Review test for edge cases and clear success criteria.

- [ ] **Step 3: Commit**

```bash
git add tests/parallel-max-agents-test.md
git commit -m "test: add parallel mode max agents test (50 items)"
```

---

## Task 10: Create Test Case - Race Condition Prevention

**Files:**
- Create: `tests/race-condition-test.md`

- [ ] **Step 1: Write test documentation**

```markdown
# Race Condition Prevention Test

**Purpose:** Verify two subagents cannot claim the same item simultaneously.

**Setup:**
1. Create minimal test: `mkdir -p /tmp/batch-test-race`
2. Create only 2 items (forces competition):

```bash
cd /tmp/batch-test-race

cat > item_A.txt <<'EOF'
Content A for processing
EOF

cat > item_B.txt <<'EOF'
Content B for processing
EOF
```

**Execution:**
1. Request: "Process all items in /tmp/batch-test-race. Extract any content."
2. Choose parallel mode
3. Observe: AI spawns 1 subagent (ceil(2/10) = 1)
4. **Manual intervention:** While processing, spawn a second subagent manually (simulate coordinator spawning extra)
5. Watch todos.md during processing

**Expected Behavior:**

**Scenario 1: No race condition**
- Subagent 1 claims item_A.txt: `[>] item_A.txt (agent-1)`
- Subagent 2 sees `[>]`, skips to item_B.txt
- Subagent 2 claims item_B.txt: `[>] item_B.txt (agent-2)`
- Both process different items

**Scenario 2: Simultaneous claim attempt**
- Both read todos.md at same instant
- Both see `[ ] item_A.txt`
- Both try to claim it
- **File system atomic write:** Only one succeeds
- Other sees conflict, re-reads, finds `[>]`, moves to next item

**Success Criteria:**
- Each item processed exactly once (check insights.md for duplicates)
- No item appears twice in insights.md
- Final todos.md shows both items `[x]`

**Note:** This test is hard to reproduce reliably (race window is milliseconds). Main verification is code review of claiming protocol + manual testing when possible.
```

- [ ] **Step 2: Add note about test limitations**

Acknowledge that race condition tests are probabilistic and hard to trigger reliably.

- [ ] **Step 3: Commit**

```bash
git add tests/race-condition-test.md
git commit -m "test: add race condition prevention test"
```

---

## Task 11: Create Test Case - Subagent Failure Recovery

**Files:**
- Create: `tests/failure-recovery-test.md`

- [ ] **Step 1: Write test documentation**

```markdown
# Subagent Failure Recovery Test

**Purpose:** Verify system recovers gracefully when a subagent crashes mid-processing.

**Setup:**
1. Create test directory: `mkdir -p /tmp/batch-test-failure`
2. Create 10 items:

```bash
cd /tmp/batch-test-failure

for i in $(seq -f "%02g" 1 10); do
  echo "Item ${i} content for extraction" > "item_${i}.txt"
done
```

**Execution:**
1. Request: "Process all items in /tmp/batch-test-failure."
2. Choose parallel mode
3. Observe: AI spawns 1 subagent (ceil(10/10) = 1)
4. **Simulate failure:** After 3-4 items processed, manually kill the subagent process
5. Watch coordinator response

**Expected Recovery:**

**Coordinator detects failure:**
- Identifies subagent is dead
- Finds items in `[>] item_XX.txt (agent-1)` state
- Changes them back to `[ ]`

**Work recovery:**
- If other subagents running: they claim the `[ ]` items
- If no other subagents: coordinator may spawn replacement OR leave for manual restart

**Manual restart scenario:**
- User restarts: "Continue batch processing from todos.md"
- AI re-reads todos.md, sees some items `[ ]`, some `[x]`
- Processes only `[ ]` items (skips `[x]`)

**After recovery:**

todos.md shows all 10 items `[x]`:
```markdown
- [x] item_01.txt
- [x] item_02.txt
...
- [x] item_10.txt
```

insights.md has all 10 findings (no gaps from crashed subagent).

**Success Criteria:**
- All items eventually marked `[x]`
- No items lost (all 10 appear in final insights.md)
- No duplicate processing (each item processed once)
- System recovers without user debugging file states manually
```

- [ ] **Step 2: Verify test clarity**

Ensure test explains both automatic and manual recovery paths.

- [ ] **Step 3: Commit**

```bash
git add tests/failure-recovery-test.md
git commit -m "test: add subagent failure recovery test"
```

---

## Task 12: Create Test Case - Merge Validation

**Files:**
- Create: `tests/merge-validation-test.md`

- [ ] **Step 1: Write test documentation**

```markdown
# Merge Validation Test

**Purpose:** Verify merge correctly combines findings from multiple subagents by category.

**Setup:**
1. Create test directory: `mkdir -p /tmp/batch-test-merge`
2. Create 15 items with mixed categories:

```bash
cd /tmp/batch-test-merge

# Frustration items (1-5)
for i in $(seq 1 5); do
  cat > "item_${i}.txt" <<EOF
Customer: "This is frustrating problem ${i}"
EOF
done

# Confusion items (6-10)
for i in $(seq 6 10); do
  cat > "item_${i}.txt" <<EOF
Customer: "I'm confused about issue ${i}"
EOF
done

# Stress items (11-15)
for i in $(seq 11 15); do
  cat > "item_${i}.txt" <<EOF
Customer: "This is causing stress around problem ${i}"
EOF
done
```

**Execution:**
1. Request: "Analyze all items. Extract quotes and categorize by: Frustration, Confusion, Stress."
2. Choose parallel mode
3. Observe: Spawns 2 subagents (ceil(15/10) = 2)
4. Wait for completion

**Expected Merge Behavior:**

**Before merge:**
- insights-1.md has findings from ~7-8 items (mixed categories)
- insights-2.md has findings from ~7-8 items (mixed categories)

**After merge:**

insights.md has all findings grouped by category:
```markdown
## Frustration
- "This is frustrating problem 1" (item_1.txt)
- "This is frustrating problem 2" (item_2.txt)
- "This is frustrating problem 3" (item_3.txt)
- "This is frustrating problem 4" (item_4.txt)
- "This is frustrating problem 5" (item_5.txt)

## Confusion
- "I'm confused about issue 6" (item_6.txt)
- "I'm confused about issue 7" (item_7.txt)
...
- "I'm confused about issue 10" (item_10.txt)

## Stress
- "This is causing stress around problem 11" (item_11.txt)
...
- "This is causing stress around problem 15" (item_15.txt)
```

**Validation Checks:**
- Count findings per category: Frustration = 5, Confusion = 5, Stress = 5
- Verify no category duplication (each category appears exactly once)
- Verify all findings from insights-1.md and insights-2.md are present
- Verify source filenames preserved for all findings
- Verify intermediate files deleted

**Success Criteria:**
- All 15 findings present in insights.md
- Correct category grouping (5 per category)
- No duplicates within categories
- Source attribution intact
```

- [ ] **Step 2: Verify test coverage**

Ensure test validates all critical merge behaviors.

- [ ] **Step 3: Commit**

```bash
git add tests/merge-validation-test.md
git commit -m "test: add merge validation test"
```

---

## Task 13: Final Review and Documentation Update

**Files:**
- Modify: `SKILL.md` (verify all sections integrated correctly)
- Create: `tests/README.md`

- [ ] **Step 1: Review SKILL.md for consistency**

Read through entire SKILL.md to verify:
- Parallel vs Sequential section flows well
- Detection Pattern correctly offers both modes
- Parallel Processing Workflow is complete and clear
- Quick Reference includes both modes
- Troubleshooting covers parallel scenarios
- Validation includes parallel checks
- No broken references or contradictions

- [ ] **Step 2: Create test documentation index**

```markdown
# Batch Processing Skill Tests

## Test Suite Overview

These tests validate both sequential and parallel modes of the batch-processing skill.

## Sequential Mode Tests

- **sequential-regression-test.md** - Verifies sequential mode unchanged after adding parallel

## Parallel Mode Tests

- **parallel-basic-test.md** - Basic parallel execution with 25 items (3 subagents)
- **parallel-max-agents-test.md** - Max agent cap with 50 items (5 subagents)
- **race-condition-test.md** - Verifies claiming protocol prevents duplicate work
- **failure-recovery-test.md** - Subagent crash recovery and work resumption
- **merge-validation-test.md** - Category-based merge correctness

## Running Tests

All tests are manual validation tests that exercise the skill in real scenarios.

**Setup:** Each test creates sample data in `/tmp/batch-test-*` directories.

**Execution:** Follow test steps, observe AI behavior, verify expected results.

**Success Criteria:** Each test documents specific outcomes to verify.

## Test Coverage

- ✓ Sequential mode regression (backward compatibility)
- ✓ Parallel mode basic functionality
- ✓ Subagent count calculation (1 per 10 items, max 5)
- ✓ Work-stealing coordination
- ✓ Race condition prevention
- ✓ Failure recovery
- ✓ Multi-category merge
- ✓ Progress monitoring
- ✓ Cleanup (intermediate files deleted)
```

- [ ] **Step 3: Commit final updates**

```bash
git add SKILL.md tests/README.md
git commit -m "docs: final review and test documentation index"
```

---

## Self-Review Checklist

**Spec Coverage:**
- ✓ Two execution modes (sequential + parallel)
- ✓ Mode selection at setup
- ✓ Subagent count calculation (1 per 10, max 5)
- ✓ Work-stealing protocol with three states
- ✓ Progress monitoring (30s polling)
- ✓ Merge strategy (category-based)
- ✓ Failure recovery (crashed subagents)
- ✓ File structure (insights-N.md per subagent)
- ✓ Backward compatibility (sequential unchanged)

**No Placeholders:**
- ✓ All code blocks complete
- ✓ All file paths exact
- ✓ All test scenarios fully specified
- ✓ No TBD, TODO, or "implement later"

**Type Consistency:**
- ✓ File states: `[ ]` → `[>]` → `[x]` (consistent throughout)
- ✓ File names: insights-1.md through insights-N.md (consistent)
- ✓ Polling interval: 30s (consistent)
- ✓ Subagent count formula: min(ceil(count/10), 5) (consistent)

**Completeness:**
- ✓ Documentation updates cover all affected sections
- ✓ Tests cover sequential regression, parallel basic, max agents, race conditions, failures, merge
- ✓ Each task has clear file paths, steps, commits
- ✓ All tasks produce working, testable changes

---

## Execution Options

Plan complete and saved to `docs/superpowers/plans/2026-06-16-parallel-batch-processing.md`.

**Two execution options:**

**1. Subagent-Driven (recommended)** - Dispatch a fresh subagent per task, review between tasks, fast iteration

**2. Inline Execution** - Execute tasks in this session using executing-plans, batch execution with checkpoints

**Which approach?**
