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
