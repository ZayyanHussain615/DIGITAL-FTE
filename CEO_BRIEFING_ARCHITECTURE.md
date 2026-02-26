# CEO Briefing - Data Flow Architecture

**Implementation Date:** 2026-02-19  
**Status:** ✅ COMPLETE

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CEO Briefing Generator                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │              Data Collection Layer                           │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │    │
│  │  │  Odoo    │  │  Meta    │  │ Twitter  │  │  Vault   │   │    │
│  │  │  MCP     │  │  MCP     │  │  MCP     │  │  Logs    │   │    │
│  │  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘   │    │
│  └───────┼─────────────┼─────────────┼─────────────┼──────────┘    │
│          │             │             │             │                │
│          ▼             ▼             ▼             ▼                │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │              Data Processing Layer                           │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │    │
│  │  │  Revenue     │  │  Social      │  │  Risk        │      │    │
│  │  │  Analysis    │  │  Metrics     │  │  Detection   │      │    │
│  │  └──────────────┘  └──────────────┘  └──────────────┘      │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              │                                       │
│                              ▼                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │              Report Generation Layer                         │    │
│  │  ┌──────────────────────────────────────────────────────┐  │    │
│  │  │  CEO Briefing (Markdown)                             │  │    │
│  │  │  /Briefings/YYYY-MM-DD_CEO_Briefing.md               │  │    │
│  │  └──────────────────────────────────────────────────────┘  │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │              Alert Layer                                     │    │
│  │  ┌──────────────────────────────────────────────────────┐  │    │
│  │  │  Anomaly Alerts                                      │  │    │
│  │  │  /Pending_Approval/ALERT_*.md                        │  │    │
│  │  └──────────────────────────────────────────────────────┘  │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Data Sources

### 1. Odoo MCP Server

| Data Type | Endpoint | Frequency |
|-----------|----------|-----------|
| Revenue | get_revenue_summary | Weekly |
| Expenses | get_expense_summary | Weekly |
| Cashflow | get_cashflow | Weekly |
| Unpaid Invoices | get_unpaid_invoices | Weekly |

**Data Flow:**
```
CEO Briefing Generator
    │
    ▼
MCP Router
    │
    ▼
Odoo MCP Server
    │
    ▼
Odoo JSON-RPC API
    │
    ▼
account.move (invoices)
account.payment (payments)
```

### 2. Meta Social MCP Server

| Data Type | Endpoint | Frequency |
|-----------|----------|-----------|
| Facebook Metrics | get_page_insights | Weekly |
| Instagram Metrics | get_ig_insights | Weekly |
| Combined Summary | generate_weekly_social_summary | Weekly |

### 3. Twitter MCP Server

| Data Type | Endpoint | Frequency |
|-----------|----------|-----------|
| Tweet Metrics | get_tweet_metrics | Weekly |
| Account Metrics | get_account_metrics | Weekly |
| Weekly Summary | generate_weekly_x_summary | Weekly |

### 4. Vault Logs

| Data Type | Source | Analysis |
|-----------|--------|----------|
| Error Patterns | /Logs/*.json | Bottleneck detection |
| Task Completions | /Done/*.md | Progress tracking |
| Audit Entries | /Logs/mcp_router_*.json | System health |

---

## Data Processing Pipeline

### Revenue Analysis

```
Raw Data (Odoo)
    │
    ▼
Calculate Weekly Revenue
    │
    ▼
Calculate MTD Revenue
    │
    ▼
Compare to Previous Week
    │
    ▼
Calculate Growth %
    │
    ▼
Detect Anomalies (>20% decline)
    │
    ▼
Generate Risk Alert (if needed)
```

### Expense Analysis

```
Raw Data (Odoo)
    │
    ▼
Group by Category
    │
    ▼
Calculate Weekly Total
    │
    ▼
Calculate Expense Ratio
    │
    ▼
Detect Anomalies (>80% of revenue)
    │
    ▼
Generate Recommendation (if needed)
```

### Social Media Aggregation

```
Meta MCP ─────┐
              ├──► Combine Metrics ──► Engagement Rate
Twitter MCP ──┘
```

### Risk Detection

```
Financial Data
    │
    ▼
┌─────────────────────────────────┐
│  Risk Detection Rules           │
│  - Revenue decline > 20%        │
│  - Expense ratio > 80%          │
│  - Negative cashflow            │
│  - Runway < 180 days            │
│  - Unpaid invoices > threshold  │
└─────────────────────────────────┘
    │
    ▼
Generate Risk Alerts
```

---

## Briefing Structure

```markdown
# CEO Weekly Briefing

## Executive Summary
- Status indicator (🟢/🟡/🔴)
- Key metrics snapshot
- Risk alert count

## Revenue
- Weekly revenue
- MTD revenue
- Growth % (WoW)
- Invoice count

## Expenses
- Weekly expenses
- MTD expenses
- By category breakdown

## Cashflow
- Cash inflow
- Cash outflow
- Net cashflow
- Runway (days)

## Unpaid Invoices
- Total outstanding
- Top 5 by amount

## Social Media Performance
- Facebook/Instagram metrics
- X/Twitter metrics
- Engagement rates

## Completed Tasks
- Grouped by domain
- Priority indicators

## Bottlenecks
- Identified blockers
- Impact assessment

## Risk Alerts
- Financial risks
- Operational risks
- Recommendations

## Strategic Recommendations
- Revenue actions
- Cost optimization
- Risk mitigation
```

---

## Anomaly Detection

### Thresholds

| Metric | Threshold | Alert Severity |
|--------|-----------|----------------|
| Revenue decline | > 30% WoW | Critical |
| Revenue decline | > 20% WoW | High |
| Expense ratio | > 80% of revenue | Medium |
| Negative cashflow | > $5,000 | High |
| Runway | < 90 days | Critical |
| Runway | < 180 days | High |

### Alert Flow

```
Anomaly Detected
    │
    ▼
Create Alert File
    │
    ▼
/Pending_Approval/ALERT_*.md
    │
    ├──► Dashboard.md notification
    │
    └──► Email to CEO (future)
```

---

## Agent Skill Integration

### Skill: ceo-briefing

```
User Request
    │
    ▼
Claude Code
    │
    ▼
Agent Skill (.claude/skills/ceo-briefing/)
    │
    ▼
ceo_briefing.py (CLI wrapper)
    │
    ▼
ceo_briefing_generator.py (main logic)
    │
    ├──► Data Collection
    ├──► Analysis
    ├──► Report Generation
    └──► Save to /Briefings/
```

### Usage

```bash
# Generate current week briefing
python .claude/skills/ceo-briefing/scripts/ceo_briefing.py

# Generate specific week
python .claude/skills/ceo-briefing/scripts/ceo_briefing.py \
  --date 2026-02-17

# List recent briefings
python .claude/skills/ceo-briefing/scripts/ceo_briefing.py --list

# Check pending alerts
python .claude/skills/ceo-briefing/scripts/ceo_briefing.py --alerts
```

---

## Scheduling

### Weekly Execution

```
Every Monday 9:00 AM
    │
    ▼
Scheduler (cron/Task Scheduler)
    │
    ▼
python scripts/ceo_briefing_generator.py
    │
    ├──► Generate briefing
    ├──► Save to /Briefings/
    └──► Create alerts if needed
```

### Windows Task Scheduler

```powershell
# Create task (run as Administrator)
schtasks /Create /TN "CEO Briefing" /TR "python C:\path\to\scripts\ceo_briefing_generator.py" /SC WEEKLY /D MON /ST 09:00
```

### Linux Cron

```bash
# Edit crontab
crontab -e

# Add entry (every Monday 9 AM)
0 9 * * 1 cd /path/to/vault && python scripts/ceo_briefing_generator.py
```

### macOS LaunchAgent

```xml
<!-- ~/Library/LaunchAgents/com.aiemployee.ceobriefing.plist -->
<key>StartCalendarInterval</key>
<dict>
    <key>Weekday</key><integer>1</integer>
    <key>Hour</key><integer>9</integer>
    <key>Minute</key><integer>0</integer>
</dict>
```

---

## Error Handling

### Data Source Failures

| Failure | Handling |
|---------|----------|
| Odoo unavailable | Show "data unavailable", continue with other sources |
| Social MCP unavailable | Show "social data unavailable" |
| Logs unreadable | Skip bottleneck analysis |

### Generation Failures

| Failure | Handling |
|---------|----------|
| Timeout (>2 min) | Return error, log to audit |
| File write error | Return error, notify user |
| Invalid data | Skip section, log warning |

---

## Audit Logging

All briefing generations logged to `/Logs/mcp_router_*.json`:

```json
{
  "timestamp": "2026-02-19T09:00:00",
  "action_type": "BRIEFING_GENERATED",
  "actor": "skill:ceo_briefing",
  "result": "success",
  "details": {
    "date": "2026-02-17",
    "week": 8,
    "revenue": 12500,
    "expenses": 8000
  }
}
```

---

## Files Created

| File | Purpose |
|------|---------|
| `scripts/ceo_briefing_generator.py` | Main briefing generator (1000+ lines) |
| `.claude/skills/ceo-briefing/SKILL.md` | Agent skill documentation |
| `.claude/skills/ceo-briefing/scripts/ceo_briefing.py` | Agent skill CLI |
| `CEO_BRIEFING_ARCHITECTURE.md` | This document |

---

## Summary

**All requirements implemented:**

✅ Executive Summary  
✅ Revenue (weekly, MTD, growth %)  
✅ Expenses (weekly, by category)  
✅ Cashflow analysis  
✅ Unpaid invoices  
✅ Social media performance  
✅ Completed tasks  
✅ Bottlenecks  
✅ Risk alerts  
✅ Strategic recommendations  
✅ Anomaly detection → Pending_Approval/  
✅ Agent Skill implementation  
✅ Scheduled execution (Monday 9 AM)  

**Data Sources:**
- Odoo MCP (accounting)
- Meta Social MCP (Facebook, Instagram)
- Twitter MCP (X)
- Vault Logs
- /Done folder

**Total Implementation:**
- 2 files (generator + skill)
- 1200+ lines of code
- 10 briefing sections
- Full anomaly detection
- Automated scheduling
