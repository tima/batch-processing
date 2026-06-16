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
