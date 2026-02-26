# Enterprise Resilience Architecture

**Implementation Date:** 2026-02-19  
**Status:** ✅ COMPLETE

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                  Enterprise Resilience Framework                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │              Retry Layer                                     │    │
│  │  ┌──────────────────────────────────────────────────────┐  │    │
│  │  │  Exponential Backoff with Jitter                     │  │    │
│  │  │  - Max Retries: 5                                    │  │    │
│  │  │  - Base Delay: 1s → Max: 60s                         │  │    │
│  │  │  - Backoff: 2x                                       │  │    │
│  │  │  - Jitter: 10%                                       │  │    │
│  │  └──────────────────────────────────────────────────────┘  │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │              Queue System                                    │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │    │
│  │  │  Failed  │  │  Failed  │  │  Failed  │  │  Failed  │   │    │
│  │  │  API     │  │  Posts   │  │  Accounting│  │  Other   │   │    │
│  │  │  Calls   │  │          │  │  Entries  │  │  Ops     │   │    │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │    │
│  │                                                              │    │
│  │  Priorities: CRITICAL > HIGH > NORMAL > LOW                 │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │              Watchdog Process                                │    │
│  │  ┌──────────────────────────────────────────────────────┐  │    │
│  │  │  Process Monitoring & Auto-Restart                   │  │    │
│  │  │  - Check Interval: 30s                               │  │    │
│  │  │  - Max Restarts: 5                                   │  │    │
│  │  │  - Cooldown: 60s                                     │  │    │
│  │  └──────────────────────────────────────────────────────┘  │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │              File Quarantine                                 │    │
│  │  ┌──────────────────────────────────────────────────────┐  │    │
│  │  │  Corrupted File Isolation                            │  │    │
│  │  │  - MD5 Hash Tracking                                 │  │    │
│  │  │  - Manifest Management                               │  │    │
│  │  │  - Restore/Delete Operations                         │  │    │
│  │  └──────────────────────────────────────────────────────┘  │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │              Health Monitor                                  │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │    │
│  │  │  CPU     │  │  Memory  │  │  Disk    │  │  Uptime  │   │    │
│  │  │  Usage   │  │  Usage   │  │  Usage   │  │  Track   │   │    │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │              JSON Logging                                    │    │
│  │  All operations logged to /Logs/                            │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Retry Logic with Exponential Backoff

### Configuration

```python
RetryConfig(
    max_retries=5,
    base_delay=1.0,      # 1 second
    max_delay=60.0,      # 60 seconds
    backoff_multiplier=2.0,
    jitter=0.1,          # 10%
    strategy=RetryStrategy.EXPONENTIAL
)
```

### Delay Calculation

```
Attempt 0: 1s ± 10%
Attempt 1: 2s ± 10%
Attempt 2: 4s ± 10%
Attempt 3: 8s ± 10%
Attempt 4: 16s ± 10%
Attempt 5: 32s ± 10% (capped at 60s)
```

### Usage

```python
from resilience.resilience_framework import with_retry, RetryConfig

@with_retry(RetryConfig(max_retries=5))
def call_api(endpoint, data):
    # API call that may fail
    pass

# Or use directly
from resilience.resilience_framework import RetryHandler

handler = RetryHandler()
result = handler.execute_with_retry(call_api, 'endpoint', {'key': 'value'})
```

---

## Queue System

### Queue Structure

```
/Action_Queue/
├── api_call_20260219120000_1234.json
├── post_20260219120001_5678.json
└── accounting_entry_20260219120002_9012.json
```

### Item Format

```json
{
  "item_id": "api_call_20260219120000_1234",
  "item_type": "api_call",
  "operation": "create_invoice",
  "parameters": {"partner_id": 12, "amount": 1000},
  "created_at": "2026-02-19T12:00:00",
  "priority": 1,
  "retry_count": 0,
  "max_retries": 5,
  "last_error": "Connection timeout",
  "next_retry_at": "2026-02-19T12:01:00"
}
```

### Priority Levels

| Priority | Value | Use Case |
|----------|-------|----------|
| CRITICAL | 0 | Payment processing |
| HIGH | 1 | API calls |
| NORMAL | 2 | Social posts |
| LOW | 3 | Background tasks |

### Queue Operations

```python
from resilience.resilience_framework import RetryQueue, QueuePriority

queue = RetryQueue()

# Enqueue failed operation
item_id = queue.enqueue(
    item_type='api_call',
    operation='create_invoice',
    parameters={'partner_id': 12},
    priority=QueuePriority.HIGH
)

# Process queue
def processor(item):
    # Try to re-execute
    return True  # Success

results = queue.process_queue(processor)
# {'success': 5, 'failed': 2, 'skipped': 0}
```

---

## Watchdog Process

### Configuration

```python
WATCHDOG_CHECK_INTERVAL = 30  # seconds
WATCHDOG_MAX_RESTARTS = 5
WATCHDOG_RESTART_COOLDOWN = 60  # seconds
```

### Process Registration

```python
from resilience.resilience_framework import Watchdog

watchdog = Watchdog()

# Register processes to monitor
watchdog.register_process(
    name='filesystem_watcher',
    command=['python', 'filesystem_watcher.py'],
    auto_restart=True,
    max_restarts=5,
    cooldown=60
)

watchdog.register_process(
    name='mcp_router',
    command=['python', 'orchestration/mcp_router.py'],
    auto_restart=True
)

# Start monitoring
watchdog.start_monitoring()
```

### Process States

| State | Description |
|-------|-------------|
| running | Process is running |
| stopped | Process was stopped |
| crashed | Process crashed, will restart |
| failed | Exceeded max restarts |

---

## File Quarantine

### Quarantine Structure

```
/Quarantine/
├── Q_20260219120000_corrupted.md
├── Q_20260219120001_invalid.json
└── quarantine_manifest.json
```

### Manifest Format

```json
{
  "Q_20260219120000_corrupted.md": {
    "original_name": "task.md",
    "original_location": "/Needs_Action",
    "quarantine_path": "/Quarantine/Q_20260219120000_corrupted.md",
    "reason": "Invalid YAML front matter",
    "file_hash": "abc123...",
    "quarantined_at": "2026-02-19T12:00:00",
    "file_size": 1024,
    "status": "active"
  }
}
```

### Quarantine Operations

```python
from resilience.resilience_framework import FileQuarantine

quarantine = FileQuarantine()

# Quarantine corrupted file
qid = quarantine.quarantine_file(
    Path('/Needs_Action/corrupted.md'),
    reason='Invalid YAML front matter'
)

# Restore file
quarantine.restore_file(qid)

# Delete permanently
quarantine.delete_quarantined(qid)

# Get stats
stats = quarantine.get_quarantine_stats()
# {'total': 5, 'active': 3, 'restored': 1, 'deleted': 1}
```

---

## System Health Monitor

### Health Metrics

| Metric | Source | Threshold |
|--------|--------|-----------|
| CPU % | psutil | >90% = UNHEALTHY |
| Memory % | psutil | >90% = UNHEALTHY |
| Disk % | psutil | >90% = UNHEALTHY |
| Uptime | process create time | - |
| Scripts Running | process list | - |
| Queued Items | queue stats | - |
| Quarantined Files | quarantine stats | - |

### Health Status

| Status | Conditions |
|--------|------------|
| HEALTHY | CPU <70%, Memory <70%, Failures <10 |
| DEGRADED | CPU >70% OR Memory >70% OR Failures >10 |
| UNHEALTHY | CPU >90% OR Memory >90% OR Failures >50 |

### Health Report Format

```json
{
  "timestamp": "2026-02-19T12:00:00",
  "status": "healthy",
  "cpu_percent": 45.2,
  "memory_percent": 62.5,
  "disk_percent": 55.0,
  "uptime_seconds": 3600,
  "scripts_running": 3,
  "last_successful_run": "2026-02-19T11:55:00",
  "failed_operations_24h": 5,
  "queued_items": 12,
  "quarantined_files": 3
}
```

---

## JSON Logging

### Log Format

```json
{
  "timestamp": "2026-02-19T12:00:00",
  "action_type": "RETRY",
  "actor": "resilience_framework",
  "result": "success",
  "details": {
    "operation": "create_invoice",
    "attempt": 2,
    "delay": 2.5
  }
}
```

### Log Locations

| Log Type | Location |
|----------|----------|
| General | /Logs/YYYY-MM-DD.json |
| Retry | /Logs/retry_*.json |
| Health | /system_health/health_history.json |
| Status | /system_health/status_*.json |

---

## Usage Examples

### Execute with Resilience

```python
from resilience.resilience_framework import ResilienceFramework

framework = ResilienceFramework()

# Execute with automatic retry and queue on failure
result = framework.execute_with_resilience(
    operation='create_invoice',
    func=odoo_client.create_invoice,
    partner_id=12,
    amount=1000,
    item_type='accounting_entry'
)
```

### Get System Status

```bash
# Via CLI
python resilience/resilience_framework.py --action status

# Via Agent Skill
python .claude/skills/resilience/scripts/resilience.py --action status
```

### Process Retry Queue

```bash
python resilience/resilience_framework.py --action process-queue
```

### Quarantine Corrupted File

```bash
# Via CLI
python resilience/resilience_framework.py --action quarantine \
  --file /Needs_Action/corrupted.md \
  --reason "Invalid YAML"

# Via Agent Skill
python .claude/skills/resilience/scripts/resilience.py \
  --action quarantine-file \
  --file /Needs_Action/corrupted.md \
  --reason "Invalid YAML"
```

### Generate Health Report

```bash
python resilience/resilience_framework.py --action health
```

---

## Implementation Checklist

| Requirement | Status | Location |
|-------------|--------|----------|
| Retry with exponential backoff | ✅ | `RetryHandler` class |
| Queue for failed API calls | ✅ | `RetryQueue` class |
| Queue for failed posts | ✅ | `RetryQueue` class |
| Queue for failed accounting | ✅ | `RetryQueue` class |
| Watchdog process | ✅ | `Watchdog` class |
| Restart crashed watchers | ✅ | `_handle_crash` method |
| Restart MCP servers | ✅ | `register_process` method |
| File quarantine | ✅ | `FileQuarantine` class |
| System health report | ✅ | `HealthMonitor` class |
| CPU usage tracking | ✅ | `psutil` integration |
| Script uptime tracking | ✅ | `HealthReport` |
| Last successful run | ✅ | `HealthReport` |
| JSON logging | ✅ | All operations |
| Never crash silently | ✅ | Exception handling |
| Agent Skill implementation | ✅ | `.claude/skills/resilience/` |

---

## Files Created

| File | Purpose | Lines |
|------|---------|-------|
| `resilience/resilience_framework.py` | Core framework | 1500+ |
| `.claude/skills/resilience/SKILL.md` | Skill documentation | - |
| `.claude/skills/resilience/scripts/resilience.py` | Agent skill CLI | 250+ |
| `RESILIENCE_ARCHITECTURE.md` | This document | - |

---

## Summary

**All requirements implemented:**

✅ Retry logic with exponential backoff  
✅ Queue system for failed operations  
✅ Watchdog process monitoring  
✅ File quarantine for corrupted files  
✅ System health monitoring  
✅ Comprehensive JSON logging  
✅ Graceful error handling  
✅ Agent Skill implementation  

**Key Features:**
- 5 retry attempts with exponential backoff
- Priority-based queue system
- Auto-restart crashed processes
- MD5 hash tracking for quarantined files
- Real-time health monitoring
- All operations logged to JSON

**Total Implementation:**
- 1 core framework file (1500+ lines)
- 1 Agent Skill CLI (250+ lines)
- 4 major components
- Full enterprise-grade resilience
