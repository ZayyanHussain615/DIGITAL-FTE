# Stop Hook - AI Task Completion Monitor

Keeps the AI agent running until tasks in `/Needs_Action` are fully complete.

## Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Stop Hook Monitor                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Send prompt to AI                                                │
│  2. Check for completion signals:                                    │
│     - Promise: <promise>TASK_COMPLETE</promise>                      │
│     - File: Task moved to /Done/                                     │
│  3. If incomplete:                                                   │
│     - Build context-aware re-prompt                                  │
│     - Increment iteration counter                                    │
│     - Continue until complete or max iterations                      │
│  4. Log all iterations to /Logs/                                     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## Quick Start

```bash
# Run demo mode (simulated AI)
python stop_hook.py --demo

# Monitor specific task
python stop_hook.py --task my_task.md

# Custom max iterations
python stop_hook.py --max-iterations 20 --task my_task.md
```

## Completion Detection

### Promise-Based Detection

AI includes this tag when task is complete:

```markdown
I've processed the task and moved the file to Done.

<promise>TASK_COMPLETE</promise>
```

The stop hook detects this pattern and marks the task complete.

### File-Based Detection

Task is complete when:
- File is moved from `/Needs_Action/` to `/Done/`
- File no longer exists in Needs_Action

## Integration Example

```python
from stop_hook import StopHook

# Initialize
hook = StopHook(
    max_iterations=10,
    check_interval=5,
    idle_timeout=300
)

# Define your AI processing function
def process_with_ai(prompt: str) -> str:
    # Call your AI API here
    response = my_ai_client.chat(prompt)
    return response

# Process with monitoring
success, status, iterations = hook.process_with_monitoring(
    task_file="Needs_Action/my_task.md",
    initial_prompt="Please process this task",
    process_func=process_with_ai
)

if success:
    print(f"Task completed: {status}")
else:
    print(f"Task incomplete: {status}")
    print(f"Iterations used: {len(iterations)}")
```

## Configuration

### Constructor Options

| Parameter | Default | Description |
|-----------|---------|-------------|
| `max_iterations` | 10 | Maximum retry attempts |
| `check_interval` | 5 | Seconds between checks |
| `idle_timeout` | 300 | Seconds before timeout |

### File Movement Monitor

```python
# Wait for file to be moved to Done
completed = hook.monitor_file_movement(
    task_file="Needs_Action/task.md",
    timeout=60  # Wait up to 60 seconds
)
```

## Context Preservation

The stop hook maintains context across iterations:

### Saved Context (in /Logs/stop_hook_context/)

```json
{
  "task_file": "Needs_Action/task.md",
  "task_id": "TASK-xxx",
  "original_prompt": "Initial user request",
  "current_iteration": 3,
  "accumulated_context": "Previous AI responses...",
  "iterations": [
    {
      "iteration": 1,
      "completion_status": "incomplete",
      "error": null
    }
  ]
}
```

### Re-injection Prompt

When task is incomplete, the AI receives:

```markdown
# Task Continuation Request

**Task:** Needs_Action/task.md
**Task ID:** TASK-xxx
**Iteration:** 4 / 10

## Original Request

[Original user prompt]

## Previous Iterations

### Iteration 2
**Status:** incomplete
**Summary:** I started processing but didn't finish...

### Iteration 3
**Status:** incomplete
**Summary:** Made progress but still pending...

---

## Instructions

Continue working on this task. The previous attempt did not 
fully complete the requirements.

**Completion Criteria:**
1. Task file processed and moved to /Done/
2. OR include `<promise>TASK_COMPLETE</promise>` in your response

**Current Status:** Task still in Needs_Action, awaiting completion.

Please continue from where you left off.
```

## Logging

All iterations are logged to `/Logs/YYYY-MM-DD.json`:

```json
{
  "timestamp": "2026-02-19T12:00:00",
  "action_type": "ITERATION",
  "actor": "skill:stop_hook",
  "result": "promise_complete",
  "details": {
    "iteration": 1,
    "task_file": "demo_task.md",
    "task_id": "TASK-DEMO-001",
    "duration_ms": 150,
    "completion_signal": "<promise>TASK_COMPLETE</promise>",
    "error": null
  }
}
```

## Error Handling

### Transient Errors

- Automatically retries on next iteration
- Error logged with full context
- Context preserved for recovery

### Max Iterations Reached

- Task marked as incomplete
- Context saved to disk
- Can resume later with same context

### Graceful Shutdown

```bash
# Press Ctrl+C to stop
# Context is automatically saved
```

## Best Practices

### 1. Set Appropriate Max Iterations

```python
# Simple tasks: 3-5 iterations
hook = StopHook(max_iterations=5)

# Complex tasks: 10-20 iterations
hook = StopHook(max_iterations=20)
```

### 2. Use Clear Completion Criteria

Tell the AI exactly what "done" means:

```python
prompt = """
Process this task and:
1. Move the file to /Done/
2. Include <promise>TASK_COMPLETE</promise> when finished
"""
```

### 3. Monitor Long-Running Tasks

```python
# Start processing
success, status, _ = hook.process_with_monitoring(...)

# If still processing, monitor file movement
if not success:
    completed = hook.monitor_file_movement(
        task_file="Needs_Action/task.md",
        timeout=600  # 10 minutes
    )
```

### 4. Resume Interrupted Tasks

```python
# Context is automatically saved on interruption
# Load existing context and continue
hook = StopHook()
success, status, _ = hook.process_with_monitoring(
    task_file="Needs_Action/task.md",
    initial_prompt="Continue processing this task",
    process_func=ai_process
)
```

## API Reference

### StopHook Class

```python
class StopHook:
    def __init__(self, max_iterations=10, check_interval=5, idle_timeout=300)
    
    def process_with_monitoring(task_file, initial_prompt, process_func) 
        -> Tuple[bool, str, List[IterationRecord]]
    
    def monitor_file_movement(task_file, timeout=None) -> bool
    
    def stop()  # Called via Ctrl+C
```

### CompletionStatus Enum

```python
class CompletionStatus:
    INCOMPLETE = "incomplete"
    PROMISE_COMPLETE = "promise_complete"
    FILE_COMPLETE = "file_complete"
    TIMEOUT = "timeout"
    MAX_ITERATIONS = "max_iterations"
    ERROR = "error"
```

## Troubleshooting

### Task Never Completes

1. Check if AI is receiving prompts
2. Verify completion signal pattern is correct
3. Increase max_iterations
4. Check error logs in /Logs/

### Context Lost on Restart

Context is saved to `/Logs/stop_hook_context/{task_id}.json`

Ensure this directory is writable and not cleared between runs.

### Too Many Iterations

If tasks consistently hit max iterations:
- Review if completion criteria are clear
- Check if AI has necessary tools/access
- Consider if task should be split into smaller tasks

## Security Notes

- Context files may contain sensitive data
- Store logs securely
- Clear context after successful completion
- Don't commit context files to version control

```bash
# .gitignore
Logs/stop_hook_context/
```
