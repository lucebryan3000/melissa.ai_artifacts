# KBGEN Queue Management Playbook

> **Purpose**: Complete specification for KB generation batch queue management, monitoring, and orchestration
> **Version**: 1.0 (Refactored from kbgen-queue.md)
> **Last Updated**: 2025-12-31

This playbook defines the behavior for managing KB generation queues, including status monitoring, article batching, dependency resolution, parallel execution, and orchestration control.

---

## Mental Model

```
User → /kbgen-queue → Queue File (YAML) → Orchestrator → Parallel Tasks
                      [State Machine]     [Scheduler]    [Task(agent)]
```

**Core Concepts:**

1. **Queue File**: Single YAML file (`.claude/queue/queue.yaml`) containing all batches and articles
2. **Batch**: Logical grouping of articles with dependencies, priority, and parallelism settings
3. **Orchestrator**: Background process that consumes queue and spawns parallel generation tasks
4. **State File**: JSON checkpoint tracking current execution progress
5. **Lock File**: Mutex preventing multiple orchestrator instances

**Execution Flow:**

```
Queue YAML → Dependency Resolution → Batch Selection → Parallel Task Spawn → State Update
     ↓                                                                            ↓
Status Display ← Progress Calculation ← Article Completion ← Agent Results ← Tasks
```

---

## Inputs

| Input | Type | Source | Purpose |
|-------|------|--------|---------|
| Subcommand | String | CLI args | `status`, `add`, `edit`, `run`, `stop` |
| Topic (for `add`) | String | CLI args | Article topic to add to queue |
| `--dry-run` flag | Boolean | CLI args | Preview execution without running |
| `--force` flag | Boolean | CLI args | Force stop orchestrator (SIGKILL) |
| Queue file | YAML | `.claude/queue/queue.yaml` | Batch and article definitions |
| State file | JSON | `.claude/queue/kbgen-state.json` | Current execution state |
| Lock file | File | `.claude/queue/kbgen.lock` | Orchestrator PID and lock |
| Environment | Env var | `KBGEN_QUEUE_DIR` | Override queue directory |

---

## Outputs

| Output | Type | Destination | Content |
|--------|------|-------------|---------|
| Status report | Text | stdout | Queue overview, batch status, next actions |
| Updated queue | YAML | `.claude/queue/queue.yaml` | Modified article/batch state |
| State file | JSON | `.claude/queue/kbgen-state.json` | Orchestrator progress checkpoint |
| Lock file | File | `.claude/queue/kbgen.lock` | Orchestrator PID (created/removed) |
| Orchestrator logs | Text | `docs/kb/_state/logs/orchestration.log` | Execution logs |
| Generated articles | Markdown | `docs/kb/<category>/<slug>.md` | KB articles |
| Updated index | JSON | `docs/kb/index.json` | Search index |

---

## Canonical Entities

### Queue File Schema

**Location**: `.claude/queue/queue.yaml`

**Structure**:

```yaml
created: "2025-12-26T02:30:00Z"
total_articles: 36
batches:
  - id: batch-1-foundation
    status: pending | in_progress | completed | failed
    priority: CRITICAL | HIGH | MEDIUM | LOW
    dependencies: []              # List of batch IDs (empty = no deps)
    parallel_jobs: 6              # Max concurrent articles in this batch
    articles:
      - slug: powershell-fundamentals-version-migration
        type: reference | tutorial | troubleshooting | quickstart
        status: pending | in_progress | completed | failed | needs_audit
        attempts: 0               # Generation attempts
        score: null | 73 | 89     # Quality score (null = not scored)
        cost: 0.00                # Accumulated cost in USD
```

**Batch Status Values**:
- `pending` - Waiting for dependencies
- `in_progress` - Currently processing
- `completed` - All articles done
- `failed` - Critical error, needs intervention

**Article Status Values**:
- `pending` - Not started
- `in_progress` - Currently generating
- `completed` - Scored ≥80, saved to production
- `needs_audit` - Scored <80, enhancement cycle needed
- `failed` - Failed after 2 audit cycles

**Priority Levels**:
- `CRITICAL` - Must complete before anything else
- `HIGH` - Important, high priority
- `MEDIUM` - Standard priority
- `LOW` - Low priority, nice-to-have

### State File Schema

**Location**: `.claude/queue/kbgen-state.json`

**Structure**:

```json
{
  "current_batch": "batch-1-foundation",
  "current_article": "powershell-fundamentals-version-migration",
  "last_checkpoint": "2025-12-26T03:45:12Z",
  "total_cost": 4.23,
  "articles_completed": 12,
  "articles_failed": 1
}
```

### Lock File Format

**Location**: `.claude/queue/kbgen.lock`

**Content**:

```
PID=12345
STARTED=2025-12-26T03:30:00Z
QUEUE_FILE=/home/luce/apps/hex/.claude/queue/queue.yaml
```

---

## Plays

### PLAY: Display Queue Status

**Trigger**: `/kbgen-queue` or `/kbgen-queue status`

**Responsibilities**:
1. Read queue YAML file
2. Calculate progress metrics
3. Resolve batch dependencies
4. Format status report
5. Highlight next actions

**Inputs**:
- Queue file path (from `KBGEN_QUEUE_DIR` or default)

**Outputs**:
- Formatted status report to stdout

**Invariants**:
- MUST show overall progress (completed/total)
- MUST indicate which batches are ready vs blocked
- MUST display status emojis for visual clarity
- MUST show total cost accumulated
- MUST highlight next actionable batch

**Algorithm**:

```
1. Read queue YAML
   - Parse batches array
   - Extract total_articles, created timestamp

2. Calculate metrics
   - total_articles = count all articles across batches
   - completed_count = count articles with status='completed'
   - progress_percent = (completed_count / total_articles) * 100
   - total_cost = sum of all article.cost values

3. For each batch:
   - Resolve dependency status:
     IF dependencies array is empty:
       batch_ready = true
     ELSE:
       batch_ready = all(dep_batch.status == 'completed' for dep in dependencies)

   - Count batch articles by status:
     batch_completed = count(article.status == 'completed')
     batch_pending = count(article.status == 'pending')
     batch_in_progress = count(article.status == 'in_progress')

   - Map batch.status to emoji:
     'pending' + ready → ⏸️  (READY TO START)
     'pending' + blocked → ⏸️  (BLOCKED)
     'in_progress' → ▶️
     'completed' → ✅
     'failed' → ❌

   - Map batch.priority to emoji:
     'CRITICAL' → 🔴
     'HIGH' → 🟠
     'MEDIUM' → 🟡
     'LOW' → 🟢

4. Format output (see I/O Contracts section)

5. Identify ready batches:
   ready_batches = [batch for batch in batches if batch_ready AND batch.status != 'completed']

6. Show next steps:
   IF ready_batches:
     PRINT orchestrator start command
     PRINT monitor command
   ELSE:
     PRINT "No batches ready (all blocked or completed)"
```

---

### PLAY: Add Article to Queue

**Trigger**: `/kbgen-queue add <topic>`

**Responsibilities**:
1. Analyze topic to detect article type
2. Assign to appropriate batch (or create new)
3. Add article entry to queue YAML
4. Display confirmation with batch assignment

**Inputs**:
- `topic` (string): Article topic description

**Outputs**:
- Updated queue YAML file
- Confirmation message with batch assignment

**Invariants**:
- MUST auto-detect article type (reference, tutorial, quickstart, troubleshooting)
- MUST assign to existing batch if appropriate, OR create new batch
- MUST generate valid slug from topic
- MUST initialize article with `status=pending`, `attempts=0`, `score=null`, `cost=0.00`

**Algorithm**:

```
1. Generate slug:
   slug = topic.lower().replace(" ", "-").replace("'", "")
   slug = sanitize(slug)  # Remove special chars, limit length

2. Detect article type:
   IF topic contains ["how to", "tutorial", "guide", "walkthrough"]:
     type = "tutorial"
   ELSE IF topic contains ["quick", "getting started", "setup", "install"]:
     type = "quickstart"
   ELSE IF topic contains ["troubleshoot", "debug", "error", "fix"]:
     type = "troubleshooting"
   ELSE:
     type = "reference"

3. Find matching batch:
   FOR each batch in queue.batches:
     IF batch contains similar articles (by topic keywords):
       target_batch = batch
       BREAK

   IF no match found:
     Create new batch:
       batch_id = "batch-{next_number}-{category}"
       status = "pending"
       priority = "MEDIUM"
       dependencies = [most_recent_completed_batch_id]  # Default to sequential
       parallel_jobs = 6

4. Add article to batch:
   article = {
     slug: slug,
     type: type,
     status: "pending",
     attempts: 0,
     score: null,
     cost: 0.00
   }
   batch.articles.append(article)

5. Update queue file:
   Write YAML with updated batch

6. Display confirmation (see I/O Contracts section)
```

---

### PLAY: Edit Queue File

**Trigger**: `/kbgen-queue edit`

**Responsibilities**:
1. Check if orchestrator is running (lock file check)
2. Open queue YAML in editor
3. Validate YAML syntax after save
4. Display summary of changes

**Inputs**:
- `--force` flag (optional): Edit even if locked

**Outputs**:
- Modified queue YAML file
- Change summary to stdout

**Invariants**:
- MUST warn if orchestrator is running (locked)
- MUST validate YAML syntax before accepting changes
- MUST preserve file integrity (no corruption)
- MUST show diff summary after editing

**Algorithm**:

```
1. Check lock file:
   IF exists(.claude/queue/kbgen.lock) AND NOT --force:
     PRINT "Warning: Orchestrator is running (PID {pid})"
     PRINT "Use --force to edit anyway, or stop orchestrator first"
     EXIT

2. Create backup:
   backup_path = queue_file + ".backup." + timestamp()
   copy(queue_file, backup_path)

3. Open in editor:
   editor = ENV['EDITOR'] || 'nano'
   EXEC(editor, queue_file)

4. Validate YAML:
   TRY:
     parsed = YAML.load(queue_file)
     ASSERT parsed.batches is array
     ASSERT parsed.total_articles is number
   CATCH SyntaxError:
     PRINT "❌ YAML syntax error detected"
     PRINT "Restore backup? (y/n)"
     IF user confirms:
       copy(backup_path, queue_file)
     EXIT

5. Calculate changes:
   old = YAML.load(backup_path)
   new = YAML.load(queue_file)

   changes = []
   FOR each batch in new.batches:
     old_batch = find_by_id(old.batches, batch.id)
     IF old_batch:
       IF batch.status != old_batch.status:
         changes.append("Batch {id}: status changed ({old} → {new})")
       IF batch.priority != old_batch.priority:
         changes.append("Batch {id}: priority changed ({old} → {new})")
       IF len(batch.articles) > len(old_batch.articles):
         diff = len(batch.articles) - len(old_batch.articles)
         changes.append("Batch {id}: {diff} new articles added")
     ELSE:
       changes.append("New batch added: {id}")

6. Display summary (see I/O Contracts section)
```

---

### PLAY: Start Queue Orchestrator

**Trigger**: `/kbgen-queue run`

**Responsibilities**:
1. Check for existing lock file
2. Start orchestrator script in background
3. Create lock file with PID
4. Display orchestrator info and monitoring commands

**Inputs**:
- `--dry-run` flag (optional): Preview without starting

**Outputs**:
- Running orchestrator process
- Lock file with PID
- Initial state file
- Log files
- Status display to stdout

**Invariants**:
- MUST prevent multiple orchestrator instances (lock file mutex)
- MUST run orchestrator in background (user can close terminal)
- MUST create lock file before spawning orchestrator
- MUST display how to monitor and stop

**Algorithm**:

```
1. Check for existing orchestrator:
   IF exists(.claude/queue/kbgen.lock):
     lock = read_lock_file()
     IF process_exists(lock.PID):
       PRINT "Orchestrator already running (PID {lock.PID})"
       PRINT "Stop with: /kbgen-queue stop"
       EXIT
     ELSE:
       PRINT "Stale lock file detected, removing..."
       remove(.claude/queue/kbgen.lock)

2. Read queue file:
   queue = YAML.load(.claude/queue/queue.yaml)
   total = queue.total_articles
   pending = count articles with status='pending'

   ready_batch = find first batch where:
     - status != 'completed'
     - dependencies all completed

3. IF --dry-run:
   EXECUTE PLAY: Dry Run Preview
   EXIT

4. Start orchestrator in background:
   script = .claude/scripts/kb-gen/kbgen-queue.sh
   pid = spawn_background(bash, script, "run")

5. Create lock file:
   lock_content = """
   PID={pid}
   STARTED={now()}
   QUEUE_FILE={queue_file_path}
   """
   write(.claude/queue/kbgen.lock, lock_content)

6. Display status (see I/O Contracts section)
```

---

### PLAY: Stop Queue Orchestrator

**Trigger**: `/kbgen-queue stop`

**Responsibilities**:
1. Find orchestrator PID from lock file
2. Send graceful shutdown signal (SIGTERM)
3. Wait for graceful shutdown (max 30s)
4. Force kill if needed (SIGKILL with --force)
5. Remove lock file
6. Display final state

**Inputs**:
- `--force` flag (optional): Immediate SIGKILL

**Outputs**:
- Stopped orchestrator process
- Removed lock file
- Preserved state file
- Final status display

**Invariants**:
- MUST attempt graceful shutdown first (unless --force)
- MUST remove lock file only after process stopped
- MUST preserve state file (never delete)
- MUST show final state from state file

**Algorithm**:

```
1. Check lock file:
   IF NOT exists(.claude/queue/kbgen.lock):
     PRINT "No orchestrator running (lock file not found)"
     EXIT

   lock = read_lock_file()
   pid = lock.PID

2. Verify process exists:
   IF NOT process_exists(pid):
     PRINT "Orchestrator not running (stale lock)"
     remove(.claude/queue/kbgen.lock)
     EXIT

3. Stop orchestrator:
   IF --force:
     PRINT "Sending SIGKILL to {pid}..."
     kill(pid, SIGKILL)
     wait_for_exit(pid, timeout=5)
   ELSE:
     PRINT "Sending SIGTERM to {pid}..."
     kill(pid, SIGTERM)
     PRINT "Waiting for graceful shutdown..."
     IF wait_for_exit(pid, timeout=30):
       PRINT "✅ Stopped"
     ELSE:
       PRINT "⚠️  Process did not stop gracefully"
       PRINT "Use --force to send SIGKILL? (y/n)"
       IF user confirms:
         kill(pid, SIGKILL)

4. Remove lock file:
   remove(.claude/queue/kbgen.lock)

5. Read final state:
   IF exists(.claude/queue/kbgen-state.json):
     state = JSON.load(.claude/queue/kbgen-state.json)
     PRINT final state (see I/O Contracts section)

6. Show resume command:
   PRINT "Resume with: /kbgen-queue run"
```

---

### PLAY: Dry Run Preview

**Trigger**: `/kbgen-queue run --dry-run`

**Responsibilities**:
1. Read queue and state files
2. Analyze batch dependencies
3. Identify next batch to process
4. Show execution plan with parallel tasks
5. Estimate time and token costs
6. Show files that would be affected

**Inputs**:
- Queue file
- State file (if exists)

**Outputs**:
- Preview report to stdout

**Invariants**:
- MUST NOT spawn orchestrator
- MUST NOT modify any files
- MUST show realistic execution plan
- MUST estimate costs accurately

**Algorithm**:

```
1. Load queue and state:
   queue = YAML.load(.claude/queue/queue.yaml)
   state = JSON.load(.claude/queue/kbgen-state.json) if exists else null

2. Calculate progress:
   total = queue.total_articles
   completed = count articles with status='completed'
   pending = count articles with status='pending'
   percent = (completed / total) * 100

3. Find next batch:
   next_batch = find first batch where:
     - status != 'completed'
     - all dependencies have status='completed'

   IF not found:
     PRINT "No batches ready to process"
     EXIT

4. Identify articles ready to generate:
   ready_articles = [
     article for article in next_batch.articles
     if article.status == 'pending'
   ]

5. Calculate estimates:
   FOR each article in ready_articles:
     BASED ON article.type:
       IF type == "reference":
         target_score = "85-95/109"
         estimated_time = "8-10 min"
       ELSE IF type == "tutorial":
         target_score = "80-88/109"
         estimated_time = "6-8 min"
       ELSE IF type == "quickstart":
         target_score = "75-85/109"
         estimated_time = "5-7 min"
       ELSE:  # troubleshooting
         target_score = "80-90/109"
         estimated_time = "7-9 min"

   wall_clock = max(estimated_time for all articles)  # Parallel execution
   token_estimate = len(ready_articles) * 8000  # Rough average

6. Generate execution plan:
   tasks = [
     "Task {i}: /kbgen \"{article.slug}\""
     for i, article in enumerate(ready_articles, 1)
   ]

7. List affected files:
   new_files = [
     "docs/kb/{category}/{article.slug}.md"
     for article in ready_articles
   ]
   updated_files = [
     ".claude/queue/kbgen-state.json",
     "docs/kb/index.json"
   ]

8. Format output (see I/O Contracts section)
```

---

## I/O Contracts

### Queue Status Output Format

**Header Section**:

```
KB GENERATION QUEUE STATUS
==========================

Queue File: .claude/queue/queue.yaml
Created: {queue.created}
Total Articles: {total_articles}
Progress: {completed}/{total} ({percent}%)
Total Cost: ${total_cost}
```

**Batch Section** (for each batch):

```
Batch {n}: {batch.id}
---------------------------
Status: {emoji} {status} ({ready_state})
Priority: {priority_emoji} {priority}
Dependencies: {dep_list or "None (can start immediately)"}
Articles: {total} total ({completed} completed, {pending} pending)
Parallel Jobs: {parallel_jobs} ({description})

Articles:
  {emoji} {slug} ({type}) - {attempts} attempts{score_display}
  ...
```

**Ready to Process Section**:

```
READY TO PROCESS
================

The following batches can start immediately:
- Batch {n}: {batch.id} ({article_count} articles, {priority} priority)

All {count} articles in Batch {n} can run in parallel.
```

**Next Steps Section**:

```
NEXT STEPS
==========

1. Start orchestrator with:
   bash .claude/scripts/kb-gen/kbgen-queue.sh run

2. Monitor progress with:
   bash .claude/scripts/kb-gen/kbgen-queue.sh status

3. Or use in Claude Code:
   /kbgen-queue run
```

### Add Article Confirmation Format

```
Article Added to Queue
======================

Slug: {slug}
Type: {type} (auto-detected)
Batch: {batch.id} (matched: {keywords})
Priority: {priority}
Dependencies: {dependency_list}

Added to: .claude/queue/queue.yaml
Position: Batch {n}, Article {m} of {total}

Ready to Process: {Yes/No} ({reason})
```

### Edit Queue Change Summary Format

```
Queue Updated
=============

Changes Detected:
- Batch {id}: {change_description}
- Batch {id}: {change_description}

Validation: ✅ YAML syntax valid

Updated: .claude/queue/queue.yaml
```

### Run Orchestrator Output Format

```
Starting KB Generation Orchestrator
====================================

Queue: .claude/queue/queue.yaml
Total Articles: {total} ({completed} completed, {pending} pending)
Starting Batch: {batch.id} ({article_count} articles)

Orchestrator PID: {pid}
Lock File: .claude/queue/kbgen.lock
State File: .claude/queue/kbgen-state.json

Logs:
- Output: docs/kb/_state/logs/orchestration.log
- Errors: docs/kb/_state/logs/orchestration-error.log

Monitor progress:
  bash .claude/scripts/kb-gen/kbgen-queue.sh status
  tail -f docs/kb/_state/logs/orchestration.log

Stop orchestrator:
  /kbgen-queue stop
```

### Stop Orchestrator Output Format

```
Stopping KB Generation Orchestrator
====================================

Found orchestrator: PID {pid}
Sending {SIGTERM/SIGKILL}...
Waiting for graceful shutdown... ✅ Stopped

Final State:
- Current Batch: {batch.id}
- Last Article: {slug} ({status})
- Total Completed: {count} articles
- Total Failed: {count} articles
- Total Cost: ${cost}

Lock file removed: .claude/queue/kbgen.lock
State preserved: .claude/queue/kbgen-state.json

Resume with: /kbgen-queue run
```

### Dry Run Preview Output Format

```
═══════════════════════════════════════════
DRY RUN MODE - QUEUE EXECUTION PREVIEW
═══════════════════════════════════════════

Queue: .claude/queue/queue.yaml
Total articles: {total}
Completed: {completed} ({percent}%)
Pending: {pending} ({percent}%)

═══════════════════════════════════════════
NEXT BATCH TO PROCESS
═══════════════════════════════════════════

Batch: {batch.id}
Priority: {priority}
Parallel jobs: {parallel_jobs}
Dependencies: {dependency_list} ✓ COMPLETE

Articles ready to generate ({count}):

{n}. [PENDING] {slug}
   Type: {type}
   Target: {target_score}
   Estimated: {estimated_time}

...

═══════════════════════════════════════════
EXECUTION PLAN
═══════════════════════════════════════════

Orchestrator would spawn {count} parallel tasks:
{task_list}

Estimated wall-clock time: {time_range}
Estimated token cost: ~{tokens}K Sonnet + ~{tokens}K Haiku + $0 Codex

═══════════════════════════════════════════
FILES THAT WOULD BE AFFECTED
═══════════════════════════════════════════

New articles ({count}):
{file_list}

Updated files:
{updated_file_list}

═══════════════════════════════════════════
DRY RUN COMPLETE

Run without --dry-run to start orchestrator.
Command: /kbgen-queue run
```

---

## Invariants

### File System Invariants

- MUST use single queue file: `.claude/queue/queue.yaml`
- MUST NOT create multiple queue files
- MUST respect `KBGEN_QUEUE_DIR` environment variable if set
- MUST create lock file before starting orchestrator
- MUST remove lock file only after orchestrator stops
- MUST preserve state file (never delete, only update)

### Concurrency Invariants

- MUST prevent multiple orchestrator instances (lock file mutex)
- MUST check lock file before starting orchestrator
- MUST validate PID from lock file (check if process running)
- MUST remove stale lock files (PID not running)

### Data Integrity Invariants

- MUST validate YAML syntax before accepting queue edits
- MUST preserve queue structure (batches array, total_articles)
- MUST maintain article metadata (slug, type, status, attempts, score, cost)
- MUST update state file atomically (write temp + rename)

### Dependency Resolution Invariants

- MUST resolve batch dependencies before execution
- MUST NOT start batch if dependencies incomplete
- MUST allow parallel batch execution if no cross-dependencies
- MUST respect batch priority ordering within dependency constraints

### Parallelization Invariants

- MUST respect batch `parallel_jobs` setting
- MUST spawn tasks in parallel (not sequential)
- MUST use Task tool for each article generation
- MUST wait for all tasks in batch before proceeding to next batch

---

## Error Handling

### Lock File Errors

**Error**: Lock file exists but PID not running

**Handling**:
```
1. Detect stale lock:
   IF exists(lock_file) AND NOT process_exists(lock.PID):
     PRINT "Stale lock file detected (PID {pid} not running)"
     PRINT "Removing stale lock..."
     remove(lock_file)
     CONTINUE with operation
```

**Error**: Lock file exists and orchestrator running

**Handling**:
```
1. Detect active orchestrator:
   IF exists(lock_file) AND process_exists(lock.PID):
     IF operation is "run":
       PRINT "Orchestrator already running (PID {pid})"
       PRINT "Stop with: /kbgen-queue stop"
       EXIT
     ELSE IF operation is "edit" AND NOT --force:
       PRINT "Warning: Orchestrator is running"
       PRINT "Use --force to edit anyway"
       EXIT
```

### Queue File Errors

**Error**: Queue file not found

**Handling**:
```
1. Check default location:
   default_path = .claude/queue/queue.yaml
   IF NOT exists(default_path):
     PRINT "❌ Queue file not found: {default_path}"
     PRINT "Create queue file first or set KBGEN_QUEUE_DIR"
     EXIT
```

**Error**: Invalid YAML syntax

**Handling**:
```
1. Attempt parse:
   TRY:
     queue = YAML.load(queue_file)
   CATCH YAMLError as e:
     PRINT "❌ Invalid YAML syntax in queue file"
     PRINT "Error: {e.message}"
     PRINT "Line {e.line}: {e.context}"
     EXIT
```

**Error**: Missing required fields

**Handling**:
```
1. Validate schema:
   required_fields = ["batches", "total_articles", "created"]
   FOR field in required_fields:
     IF field NOT IN queue:
       PRINT "❌ Missing required field: {field}"
       PRINT "Queue file must contain: {required_fields}"
       EXIT
```

### Orchestrator Errors

**Error**: Orchestrator script not found

**Handling**:
```
1. Check script path:
   script_path = .claude/scripts/kb-gen/kbgen-queue.sh
   IF NOT exists(script_path):
     PRINT "❌ Orchestrator script not found: {script_path}"
     PRINT "Reinstall KB generation tools or check repository"
     EXIT
```

**Error**: Orchestrator won't stop gracefully

**Handling**:
```
1. Timeout on graceful shutdown:
   kill(pid, SIGTERM)
   IF NOT wait_for_exit(pid, timeout=30):
     PRINT "⚠️  Orchestrator did not stop after 30 seconds"
     PRINT "Force kill with SIGKILL? (y/n)"
     IF user confirms:
       kill(pid, SIGKILL)
       wait_for_exit(pid, timeout=5)
     ELSE:
       PRINT "Orchestrator still running (PID {pid})"
       PRINT "Lock file NOT removed"
       EXIT
```

### State File Errors

**Error**: State file corrupted (invalid JSON)

**Handling**:
```
1. Attempt recovery:
   TRY:
     state = JSON.load(state_file)
   CATCH JSONError:
     PRINT "⚠️  State file corrupted, using backup"
     IF exists(state_file + ".backup"):
       copy(state_file + ".backup", state_file)
       state = JSON.load(state_file)
     ELSE:
       PRINT "No backup found, state reset"
       state = null
```

---

## Testing Checklist

### Functional Tests

- [ ] Display status for empty queue
- [ ] Display status for queue with completed batches
- [ ] Display status for queue with blocked batches
- [ ] Display status for queue with ready batches
- [ ] Add article with auto-type detection (tutorial)
- [ ] Add article with auto-type detection (reference)
- [ ] Add article to existing batch
- [ ] Add article creating new batch
- [ ] Edit queue file (successful save)
- [ ] Edit queue file with syntax error (validation catch)
- [ ] Start orchestrator when no lock exists
- [ ] Start orchestrator when stale lock exists
- [ ] Prevent start when orchestrator running
- [ ] Stop orchestrator gracefully (SIGTERM)
- [ ] Force stop orchestrator (SIGKILL)
- [ ] Dry run preview with ready batch
- [ ] Dry run preview with no ready batches

### Edge Cases

- [ ] Queue file with 0 batches
- [ ] Queue file with circular dependencies (detect)
- [ ] Batch with 0 articles
- [ ] Batch with all articles completed
- [ ] State file missing (orchestrator creates new)
- [ ] Lock file with invalid PID format
- [ ] Multiple dependency chains
- [ ] Batch priority override (CRITICAL before HIGH)

### Integration Tests

- [ ] Full workflow: add → edit → run → monitor → stop
- [ ] Resume after interruption (state preserved)
- [ ] Parallel execution (verify max parallel_jobs)
- [ ] Cost accumulation (verify state.total_cost updates)
- [ ] Index update after article generation

---

## Performance Considerations

### Parallelization Guidelines

**Batch-level parallelism**: All articles within a batch run concurrently

**Recommended `parallel_jobs` settings**:

| Batch Size | parallel_jobs | Rationale |
|------------|---------------|-----------|
| 1-4 articles | All (1-4) | No contention, max speed |
| 5-8 articles | 6 | Sweet spot for hardware |
| 9+ articles | 8 | Avoid rate limits, memory pressure |

**Hardware assumptions** (Codeswarm dev box):
- 24 logical CPUs
- ~123GB RAM
- NVMe storage
- Parallel Task tool spawns agents as subprocesses

**Token cost per article** (average):
- No audit cycle: ~$0.71
- With 1 audit cycle: ~$1.42
- Target 80% pass rate (no audit needed)

### Batch Execution Time

**Wall-clock time = max(individual article times)** due to parallelism

**Example**: Batch with 6 articles
- Sequential: 6 × 8 min = 48 min
- Parallel (6 jobs): max(8 min) = 8-10 min

**Cross-batch parallelism**: Batches with no dependencies can overlap execution

---

## Explicit Non-Goals

### Out of Scope

- ❌ Real-time progress streaming (use monitor script for that)
- ❌ Web UI for queue management (CLI only)
- ❌ Distributed orchestration (single-node only)
- ❌ Queue priority reordering on the fly (edit queue manually)
- ❌ Retry logic for individual articles (handled by /kbgen orchestrator)
- ❌ Automatic batch dependency inference (must be explicit in YAML)

### Delegated to Other Tools

- Article generation logic → `/kbgen` skill
- Real-time monitoring → `kbgen-monitor.sh` script
- Article validation → `/kbgen-validate` skill
- Batch orchestration implementation → `kbgen-queue.sh` bash script
- Search indexing → Handled by article generation

---

## Related Commands and Scripts

| Command/Script | Purpose | Relationship |
|----------------|---------|--------------|
| `/kbgen <topic>` | Generate single article | Bypasses queue, direct generation |
| `/kbgen-validate <slug>` | Validate existing article | Quality check for queue output |
| `kbgen-queue.sh` | Bash script implementation | Implements plays defined here |
| `kbgen-monitor.sh` | Real-time progress dashboard | Monitors queue execution |
| `kbgen-orchestrator.sh` | Direct orchestrator invocation | Advanced usage, called by queue.sh |

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2025-12-31 | Initial playbook created via refactoring from kbgen-queue.md. Extracted all algorithmic content, I/O contracts, and specifications into canonical structure. |
