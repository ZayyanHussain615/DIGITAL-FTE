# MCP Integration Guide

Execute approved tasks via external MCP (Model Context Protocol) servers.

## Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                        MCP Action Executor                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  /Approved/                                                          │
│  ├── task_email.md ──────► Email MCP Server ──────► Send Email      │
│  ├── task_tweet.md ──────► Social MCP Server ─────► Post to Twitter│
│  └── task_browser.md ────► Browser MCP Server ────► Web Automation │
│                                                                      │
│  Output:                                                              │
│  - Dashboard.md updated                                              │
│  - Files moved to /Done/                                             │
│  - Audit log in /Logs/mcp_audit_YYYY-MM-DD.json                     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## Quick Start

### 1. Configure MCP Servers

Copy `mcp.env.template` to `.env` and configure:

```bash
# MCP Server URLs
MCP_EMAIL_SERVER=http://localhost:8080/email
MCP_BROWSER_SERVER=http://localhost:8081/browser
MCP_SOCIAL_SERVER=http://localhost:8082/social
```

**Important:** Credentials are loaded from environment variables, never stored in vault.

### 2. Create Approved Task

Move a task to `/Approved/` with `status: approved`:

```markdown
---
task_id: TASK-xxx
status: approved
action_type: send_email
email_to: team@example.com
email_subject: Status Report
---

# Task Description

Send the weekly status report.
```

### 3. Run Executor

```bash
# Dry-run mode (test without executing)
python mcp_action_executor.py --dry-run

# Execute all approved tasks
python mcp_action_executor.py

# Execute specific task
python mcp_action_executor.py --task weekly_status_email.md
```

## Supported Action Types

### Email Actions

```markdown
---
action_type: send_email
email_to: recipient@example.com
email_subject: Meeting Reminder
---

Email body content here...
```

**Parameters:**
- `email_to`: Comma-separated recipient list
- `email_subject`: Email subject line
- Content becomes email body

### Browser Actions

**Open URL:**
```markdown
---
action_type: open_url
url: https://example.com
wait_for: .main-content
---
```

**Click Element:**
```markdown
---
action_type: click_element
url: https://example.com
selector: #submit-button
---
```

**Fill Form:**
```markdown
---
action_type: fill_form
url: https://example.com/form
selector: #name-input
value: John Doe
---
```

### Social Media Actions

```markdown
---
action_type: post_social
platform: twitter
---

Post content here (truncated to 280 chars for Twitter)
```

**Platforms:** `twitter`, `linkedin`, `facebook`

## Task Status Workflow

```
/Needs_Action/          /Approved/              /Done/
     │                      │                       │
     │ 1. Import            │                       │
     ▼                      │                       │
  [pending] ──► 2. Review ──► [approved] ──► 3. Execute ──► [completed]
                                  │
                                  │ (if rejected)
                                  ▼
                             [rejected]
```

## Dry-Run Mode

Test your configuration without executing actions:

```bash
python mcp_action_executor.py --dry-run
```

Output:
```
✓ weekly_status_email.md: Action completed: send_email (dry_run)
✓ product_launch_tweet.md: Action completed: post_social (dry_run)
```

## Error Handling

### Retry Logic

- Transient errors (timeout, connection, 502, 503) are retried up to 3 times
- Permanent errors (auth failure, invalid params) fail immediately
- All retries logged to audit file

### Error Categories

| Error Type | Retry? | Action |
|------------|--------|--------|
| Timeout | Yes | Retry with backoff |
| Connection refused | Yes | Retry |
| 502/503 | Yes | Retry |
| 401/403 | No | Log and skip |
| Invalid params | No | Log and skip |

## Audit Log

All actions are logged to `/Logs/mcp_audit_YYYY-MM-DD.json`:

```json
{
  "timestamp": "2026-02-19T13:30:00",
  "task_id": "TASK-20260219130000-email",
  "task_file": "weekly_status_email.md",
  "action_type": "send_email",
  "status": "success",
  "result": {
    "message_id": "msg_abc12345",
    "status": "sent",
    "recipients": ["team@example.com"]
  },
  "error": null,
  "retries": 0,
  "duration_ms": 234,
  "dry_run": false
}
```

## Dashboard Integration

Completed actions are logged to `Dashboard.md`:

```markdown
## Recent Activity

- [2026-02-19 13:30:00] [MCP] weekly_status_email.md: success - msg_abc12345
- [2026-02-19 13:30:01] [MCP] product_launch_tweet.md: success - post_xyz789
```

## Security

### Credentials

- **NEVER** store credentials in vault files
- Use environment variables or `.env` file
- Add `.env` to `.gitignore`

```bash
# .gitignore
.env
*.env
```

### MCP Server Authentication

Configure authentication in your MCP server, not in the executor:

```bash
# Server-side configuration (example)
export EMAIL_API_KEY=secret_key
export SOCIAL_API_TOKEN=secret_token
```

## Creating Custom MCP Actions

### 1. Define Action Type

Add to `ActionType` enum in `mcp_action_executor.py`:

```python
class ActionType(Enum):
    CUSTOM_ACTION = "custom_action"
```

### 2. Add Detection Logic

```python
def _detect_action_type(self, task_content, metadata):
    if 'custom trigger' in task_content.lower():
        return ActionType.CUSTOM_ACTION.value
```

### 3. Add Parameter Extraction

```python
def _extract_action_parameters(self, task_content, metadata, action_type):
    if action_type == ActionType.CUSTOM_ACTION:
        return {
            'param1': metadata.get('param1', ''),
            'param2': metadata.get('param2', '')
        }
```

### 4. Implement MCP Handler

```python
def _simulate_execution(self, request):
    if request.action_type == ActionType.CUSTOM_ACTION:
        # Your custom logic here
        return {"status": "completed"}
```

## Troubleshooting

### MCP Server Not Responding

1. Check server is running: `curl http://localhost:8080/health`
2. Verify URL in environment variables
3. Check firewall/network configuration

### Task Not Processing

1. Verify `status: approved` in front matter
2. Check action_type is recognized
3. Review `/Logs/mcp_audit_*.json` for errors

### Dry-Run Not Simulating

1. Ensure `--dry-run` flag is passed
2. Check output shows `(dry_run)` status

## API Reference

### MCPActionExecutor Methods

```python
# Initialize executor
executor = MCPActionExecutor(dry_run=False)

# Process single task
success, message = executor.process_task(Path("Approved/task.md"))

# Process all approved tasks
results = executor.process_all_approved()
```

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `MCP_EMAIL_SERVER` | Email MCP server URL | "" |
| `MCP_BROWSER_SERVER` | Browser MCP server URL | "" |
| `MCP_SOCIAL_SERVER` | Social MCP server URL | "" |
| `MCP_MAX_RETRIES` | Max retry attempts | 3 |
| `MCP_RETRY_DELAY` | Delay between retries (sec) | 5 |

## Example Workflows

### Weekly Status Email

```bash
# 1. Create task in Needs_Action
# 2. Review and approve
mv Approved/weekly_status.md Approved/

# 3. Update status
# Edit file: status: pending → status: approved

# 4. Execute
python mcp_action_executor.py --task weekly_status.md
```

### Social Media Campaign

```bash
# Create multiple social posts
# - product_launch.md
# - feature_announcement.md
# - hiring_post.md

# Execute all
python mcp_action_executor.py

# Or dry-run first
python mcp_action_executor.py --dry-run
```

### Browser Automation

```markdown
---
status: approved
action_type: open_url
url: https://dashboard.example.com
---

Navigate to the dashboard and verify all systems are operational.
```

```bash
python mcp_action_executor.py --task dashboard_check.md
```
