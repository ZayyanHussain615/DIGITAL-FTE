# MCP Orchestration Layer - Architecture Summary

**Implementation Date:** 2026-02-19  
**Status:** ✅ COMPLETE

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MCP Orchestration Layer                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    MCP Router                                │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │    │
│  │  │  Intent      │  │  Argument    │  │  Retry       │      │    │
│  │  │  Detection   │  │  Validation  │  │  Logic       │      │    │
│  │  └──────────────┘  └──────────────┘  └──────────────┘      │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              │                                       │
│         ┌────────────────────┼────────────────────┐                 │
│         │                    │                    │                  │
│         ▼                    ▼                    ▼                  │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐           │
│  │ mcp_email   │     │ mcp_odoo    │     │ mcp_meta    │           │
│  │ (Gmail)     │     │ (Accounting)│     │ (Social)    │           │
│  └─────────────┘     └─────────────┘     └─────────────┘           │
│                                                                      │
│  ┌─────────────┐     ┌─────────────┐                               │
│  │ mcp_twitter │     │ mcp_browser │                               │
│  │ (X/Twitter) │     │ (Automation)│                               │
│  └─────────────┘     └─────────────┘                               │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │              Supporting Services                             │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │    │
│  │  │  Audit Log   │  │  Action      │  │  Dashboard   │      │    │
│  │  │  /Logs/      │  │  Queue       │  │  Updates     │      │    │
│  │  └──────────────┘  └──────────────┘  └──────────────┘      │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## File Structure

```
AI_Employee_Vault/
├── orchestration/
│   ├── mcp_router.py          # Main router (800+ lines)
│   └── README.md              # Documentation
│
├── mcp_servers/
│   ├── odoo_accounting/       # Accounting server
│   ├── meta_social/           # Facebook/Instagram server
│   ├── twitter/               # Twitter/X server
│   └── browser/               # Browser automation server
│
├── Action_Queue/              # Queued actions for retry
│
├── Logs/
│   └── mcp_router_*.json      # Audit logs
│
└── Dashboard.md               # Human notifications
```

---

## MCP Router Features

| Feature | Description |
|---------|-------------|
| **Intent Detection** | Auto-detect server from task description |
| **Argument Validation** | Validate parameters before execution |
| **Retry Logic** | Exponential backoff for transient failures |
| **Dry-Run Mode** | Simulate actions without executing |
| **Audit Logging** | Comprehensive action logging |
| **Action Queuing** | Queue failed actions for retry |
| **Human Notification** | Dashboard updates on failures |

---

## Supported MCP Servers

| Server | Tools | Environment |
|--------|-------|-------------|
| **email** (Gmail) | send_email | EMAIL_ADDRESS, EMAIL_PASSWORD |
| **odoo_accounting** | create_invoice, get_revenue_summary, weekly_audit | ODOO_URL, ODOO_DB, ODOO_USERNAME, ODOO_PASSWORD |
| **meta_social** | post_facebook_message, post_instagram_caption | META_ACCESS_TOKEN, META_PAGE_ID, META_IG_ID |
| **twitter** | post_tweet, get_tweet_metrics | X_API_KEY, X_API_SECRET, X_ACCESS_TOKEN, X_ACCESS_SECRET |
| **browser** | open_url, click_element, fill_form | (optional) |

---

## Intent Detection Patterns

| Keywords | Detected Server |
|----------|-----------------|
| email, send, mail, message, notify | gmail |
| facebook, instagram, social, meta, post | meta_social |
| twitter, x, tweet, retweet | twitter |
| browse, open url, click, navigate, web | browser |
| invoice, revenue, expense, accounting, payment | odoo_accounting |

### Example
```python
router.detect_server("Send email to team about meeting")
# Returns: 'gmail'

router.detect_server("Post to Instagram about product")
# Returns: 'meta_social'

router.detect_server("Tweet about launch")
# Returns: 'twitter'
```

---

## Audit Log Format

All actions logged to `/Logs/mcp_router_YYYY-MM-DD.json`:

```json
{
  "timestamp": "2026-02-19T12:00:00",
  "server": "twitter",
  "action": "post_tweet",
  "parameters": {
    "text": "Business update! #growth"
  },
  "approval_status": "approved",
  "result": "success",
  "duration_ms": 234,
  "retries": 0,
  "error": null,
  "dry_run": false
}
```

---

## Action Queue Format

Failed actions queued to `/Action_Queue/`:

```json
{
  "action_id": "action_20260219120000_twitter",
  "server": "twitter",
  "tool": "post_tweet",
  "parameters": {
    "text": "Business update!"
  },
  "queued_at": "2026-02-19T12:00:00",
  "retry_count": 0,
  "max_retries": 3,
  "last_error": "Server unavailable: Connection timeout",
  "priority": 0
}
```

---

## Retry Logic

| Parameter | Value |
|-----------|-------|
| Max Retries | 3 |
| Base Delay | 2 seconds |
| Backoff Multiplier | 2x (exponential) |
| Delay Sequence | 2s → 4s → 8s |

### Flow
```
Action Fails
    │
    ▼
Retry 1 (wait 2s)
    │
    ├─ Success ──► Complete
    │
    ▼
Retry 2 (wait 4s)
    │
    ├─ Success ──► Complete
    │
    ▼
Retry 3 (wait 8s)
    │
    ├─ Success ──► Complete
    │
    ▼
All Retries Exhausted
    │
    ├─ Queue Action
    └─ Notify Human
```

---

## Usage Examples

### Check Server Status
```bash
python orchestration/mcp_router.py --status

# Output:
# MCP Server Status
# ============================================================
#   ✓ email: available
#   ✓ odoo_accounting: available
#   ⚠ meta_social: degraded
#   ✓ twitter: available
#   ? browser: unknown
```

### Auto-Detect Server from Intent
```bash
python orchestration/mcp_router.py \
  --intent "Send email to team about quarterly results" \
  --action send_email \
  --args '{"to": "team@example.com", "subject": "Q1 Results", "body": "See attached..."}'
```

### Explicit Server Selection
```bash
python orchestration/mcp_router.py \
  --server twitter \
  --action post_tweet \
  --args '{"text": "Exciting product launch! #innovation"}'
```

### Dry-Run Mode
```bash
python orchestration/mcp_router.py \
  --dry-run \
  --server meta_social \
  --action post_facebook_message \
  --args '{"message": "Test post"}'

# Output:
# Executing: meta_social/post_facebook_message
# [DRY-RUN MODE]
# [OK] meta_social/post_facebook_message completed
# [DRY-RUN] Simulated execution
```

### Retry Queued Actions
```bash
python orchestration/mcp_router.py --retry-queue

# Output:
# Retrying queued actions...
# Success: 2, Failed: 1
```

---

## Programmatic API

```python
from orchestration.mcp_router import MCPRouter

# Initialize router
router = MCPRouter(dry_run=False)

# Check all server statuses
status = router.get_server_status_summary()
# {'email': 'available', 'twitter': 'degraded', ...}

# Auto-detect server from intent
server = router.detect_server("Tweet about our new feature")
# Returns: 'twitter'

# Validate arguments before sending
valid, error = router.validate_arguments(
    'twitter', 
    'post_tweet', 
    {'text': 'Hello!'}
)
# valid = True

# Execute action with retry
result = router.call_server('twitter', 'post_tweet', {'text': 'Hello!'})

if result.success:
    print(f"Success: {result.result}")
else:
    print(f"Failed: {result.error}")
    print(f"Retries: {result.retries}")

# Retry queued actions
results = router.retry_queued_actions()
print(f"Success: {results['success']}, Failed: {results['failed']}")
```

---

## Dashboard Integration

Server failures automatically update `Dashboard.md`:

```markdown
## MCP Server Status

- [2026-02-19 12:00:00] ⚠️ **Server Error**: twitter/post_tweet - Connection timeout
- [2026-02-19 12:05:00] ⚠️ **Server Error**: meta_social/post_facebook_message - Rate limit exceeded
- [2026-02-19 12:10:00] ⚠️ **Server Error**: odoo_accounting/create_invoice - Auth failed
```

---

## Error Handling Matrix

| Error Type | Detection | Handling | Retry? | Notify? |
|------------|-----------|----------|--------|---------|
| Server unavailable | Status check | Queue action | Yes | Yes |
| Invalid arguments | Validation | Return error | No | No |
| Transient failure | Exception | Retry with backoff | Yes | No |
| Timeout | Timeout exception | Retry up to max | Yes | Yes |
| Rate limit | 429 response | Queue for later | Yes | Yes |
| Auth failure | 401/403 response | Return error | No | Yes |

---

## Implementation Checklist

| Requirement | Status | Location |
|-------------|--------|----------|
| MCP Router implemented | ✅ | `orchestration/mcp_router.py` |
| Auto-detect server from intent | ✅ | `detect_server()` method |
| Argument validation | ✅ | `validate_arguments()` method |
| Retry logic with backoff | ✅ | `call_server()` method |
| Dry-run mode | ✅ | `--dry-run` flag |
| Audit logging | ✅ | `/Logs/mcp_router_*.json` |
| Action queuing | ✅ | `/Action_Queue/` |
| Dashboard notification | ✅ | `Dashboard.md` updates |
| Support 5 MCP servers | ✅ | email, odoo, meta, twitter, browser |
| Modular/extensible design | ✅ | Add servers via config dict |

---

## Files Created

| File | Purpose | Lines |
|------|---------|-------|
| `orchestration/mcp_router.py` | Main router | 800+ |
| `orchestration/README.md` | Documentation | - |
| `ORCHESTRATION_ARCHITECTURE.md` | This document | - |

---

## Adding New Servers

### 1. Add Server Configuration

```python
MCP_SERVERS = {
    'new_server': {
        'path': VAULT_ROOT / 'mcp_servers' / 'new_server' / 'server.py',
        'tools': ['tool1', 'tool2', 'tool3'],
        'env_prefix': 'NEW_',
        'required_env': ['NEW_API_KEY', 'NEW_API_SECRET']
    }
}
```

### 2. Add Intent Patterns

```python
INTENT_PATTERNS = {
    'new_server': ['keyword1', 'keyword2', 'keyword3']
}
```

### 3. Add Argument Validation

```python
def validate_arguments(self, server_name, tool_name, arguments):
    if tool_name == 'new_tool':
        if 'required_param' not in arguments:
            return False, "Missing required argument: required_param"
    return True, None
```

---

## Security

| Feature | Implementation |
|---------|----------------|
| Credentials | Environment variables only |
| Audit trail | All actions logged with timestamps |
| Validation | Arguments validated before execution |
| Dry-run | Test without side effects |
| Queue encryption | Planned for future release |

---

## Summary

**All requirements implemented:**

✅ MCP Router with auto-detection  
✅ Argument validation  
✅ Retry logic with exponential backoff  
✅ Dry-run mode  
✅ Comprehensive audit logging  
✅ Action queuing for unavailable servers  
✅ Human notification via Dashboard.md  
✅ Support for 5 MCP servers  
✅ Modular and extensible design  

**Total Implementation:**
- 1 file (mcp_router.py)
- 800+ lines of code
- 5 MCP servers supported
- Full audit logging
- Retry logic with backoff
- Action queuing system
