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
