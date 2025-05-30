# 🦉 OWL Testing Pattern — *Task-Numbered, Self-Cleaning, History-Preserving*

> The quick score: **task-numbered test directories, automatic cleanup, no conflicts between runs, preserved history for debugging.**

---

## 1  Key Rules in One Glance

| 📏 Rule                                                                                                           | Why it matters                                                                                                                        |
| ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| **Task-numbered directories** — every test run gets `task_001`, `task_002`, etc.                                  | Prevents output conflicts, enables parallel testing, preserves run history.                                                           |
| **Auto-increment** task numbers based on existing directories.                                                    | No manual tracking needed, always uses next available number.                                                                         |
| **Auto-cleanup** keeps last N runs (typically 10).                                                               | Prevents disk bloat while maintaining recent history for debugging.                                                                   |
| **Consistent base path** — all tests write to `tests/test-output/task_XXX/`.                                     | Makes it trivial to find and compare test results.                                                                                   |
| **Docker requirement** — always test with `cwltool` using Docker containers.                                      | Ensures reproducibility and matches production behavior.                                                                              |

---

## 2  Canonical Test Script Structure

```bash
#!/bin/bash
set -e

echo "========================================"
echo "Project Test Suite"
echo "========================================"

# Test data paths
METADATA_FILE="/path/to/test-data/metadata.json"
FILES_DIR="/path/to/test-data"
BASE_OUTPUT_DIR="/path/to/tests/test-output"

# Find next task number
TASK_NUM=$(find $BASE_OUTPUT_DIR -maxdepth 1 -name "task_*" -type d 2>/dev/null | wc -l)
TASK_NUM=$((TASK_NUM + 1))
OUTPUT_DIR="$BASE_OUTPUT_DIR/task_$(printf "%03d" $TASK_NUM)"

# Create task-specific output directory
mkdir -p $OUTPUT_DIR
echo "Running tests in: $OUTPUT_DIR"

# Run test with cwltool
cwltool \
  --outdir $OUTPUT_DIR \
  /path/to/workflow.cwl \
  --input_file $METADATA_FILE \
  --data_directory $FILES_DIR

# Cleanup old test directories (keep last 10)
echo "Cleaning up old test directories..."
cd $BASE_OUTPUT_DIR
ls -dt task_* 2>/dev/null | tail -n +11 | xargs rm -rf 2>/dev/null || true
echo "Kept last 10 test runs"
```

---

## 3  Directory Structure After Multiple Runs

```text
tests/
├─ test-cwl.sh                    # Main test script
├─ test-sb-style.sh               # Seven Bridges style test
├─ test-data/                     # Input test data
│  ├─ metadata.json
│  └─ files/
│     └─ sample.fastq.gz
└─ test-output/                   # All test outputs
   ├─ task_001/                   # First test run
   │  ├─ upload-report.tsv
   │  └─ process.log
   ├─ task_002/                   # Second test run
   │  ├─ upload-report.tsv
   │  └─ process.log
   └─ task_010/                   # Most recent (older ones cleaned up)
      ├─ upload-report.tsv
      └─ process.log
```

---

## 4  Integration with CLAUDE.md

Add to your `CLAUDE.md`:

```markdown
## Testing

### Test Directory Management
Tests use task-numbered directories to organize outputs:
- Each test run creates a new `task_XXX` directory under `tests/test-output/`
- Task numbers auto-increment (task_001, task_002, etc.)
- Old test directories are automatically cleaned up (keeps last 10 runs)
- This prevents test output conflicts and maintains history

### Running Tests
```bash
./tests/test-cwl.sh
# Output will be in: tests/test-output/task_XXX/
```
```

---

## 5  Advanced Patterns

### Parallel Testing
```bash
# Each parallel test gets its own task number
parallel -j 4 ./test-cwl.sh ::: test1 test2 test3 test4
```

### Test Result Comparison
```bash
# Compare outputs between runs
diff test-output/task_008/report.tsv test-output/task_009/report.tsv
```

### CI Integration
```yaml
# GitHub Actions example
- name: Run CWL Tests
  run: |
    ./tests/test-cwl.sh
    # Archive the specific task directory
    LATEST=$(ls -dt tests/test-output/task_* | head -1)
    echo "Test outputs in: $LATEST"
```

---

## 6  Best Practices Checklist

| ✅ Do                                                                                    | 🚫 Don't                                     |
| --------------------------------------------------------------------------------------- | -------------------------------------------- |
| Use task numbers for every test run                                                      | Write to a fixed output directory            |
| Clean up old runs automatically                                                          | Let test outputs accumulate forever          |
| Keep enough history for debugging (5-10 runs)                                           | Delete all history after each run            |
| Use consistent paths across all test scripts                                            | Hardcode absolute paths to test outputs      |
| Test with Docker via cwltool                                                            | Run tests directly without containers        |
| Include cleanup in the test script itself                                               | Require manual cleanup                       |

---

## 7  FAQ

| Question                                        | Quick answer                                                                                          |
| ----------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| **Why not use timestamps?**                     | Task numbers are cleaner, sort naturally, and are easier to reference                                 |
| **What if I run out of task numbers?**          | After task_999, it wraps to task_1000. The cleanup keeps it manageable                               |
| **Can I disable cleanup?**                      | Yes, comment out the cleanup lines, but monitor disk usage                                           |
| **How to preserve a specific run?**             | Copy it outside test-output/ or increase the retention count                                         |
| **What about test fixtures?**                   | Keep input data in test-data/, only outputs go in task directories                                   |

---

### TL;DR for Implementation

1. **Find next number**: `find test-output -name "task_*" | wc -l`
2. **Create directory**: `mkdir -p test-output/task_$(printf "%03d" $NUM)`  
3. **Run tests**: Output everything to that task directory
4. **Cleanup**: `ls -dt task_* | tail -n +11 | xargs rm -rf`

This pattern ensures clean, conflict-free testing with automatic history management perfect for OWL workflows.