# CEO Briefing Generator

Automated weekly executive briefing generation from vault data.

## Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CEO Briefing Generator                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Input Files:                                                        │
│  - Business_Goals.md (targets, deadlines, budgets)                  │
│  - Bank_Transactions.md (revenue, expenses, subscriptions)          │
│  - /Done/ (completed tasks)                                         │
│                                                                      │
│  Processing:                                                         │
│  - Revenue analysis (weekly, MTD, trends)                           │
│  - Task completion tracking                                         │
│  - Bottleneck detection                                             │
│  - Cost anomaly detection                                           │
│  - Approval item generation                                         │
│                                                                      │
│  Output:                                                             │
│  - /Briefings/YYYY-MM-DD_Monday_Briefing.md                         │
│  - /Pending_Approval/*.md (approval items)                          │
│  - /Logs/ audit entries                                             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## Quick Start

```bash
# Generate briefing for current week
python ceo_briefing.py

# Generate for specific week
python ceo_briefing.py --week 2026-W07

# Custom output filename
python ceo_briefing.py --output Q1_Review.md
```

## Input Files

### Business_Goals.md

Define targets, deadlines, and budgets:

```markdown
# Business Goals 2026

## Q1 2026 Goals

### Revenue Targets
- Monthly Recurring Revenue (MRR): $50,000
- Weekly Revenue Target: $12,500

## Upcoming Deadlines

| Date | Deadline | Owner | Status |
|------|----------|-------|--------|
| 2026-02-28 | Q1 Board Meeting | CEO | Pending |

## Budget Allocation 2026

| Category | Q1 Budget | Annual |
|----------|-----------|--------|
| Engineering | $200,000 | $1,000,000 |
```

### Bank_Transactions.md

Track revenue and expenses:

```markdown
# Bank Transactions 2026

## February 2026

### Week 7 (Feb 17-23)

| Date | Description | Type | Amount | Category |
|------|-------------|------|--------|----------|
| 2026-02-19 | Stripe Payment - Customer #1042 | Revenue | +$499.00 | Revenue |
| 2026-02-19 | AWS Infrastructure | Expense | -$1,250.00 | Infrastructure |

## Recurring Subscriptions

| Service | Monthly Cost | Next Billing | Status |
|---------|--------------|--------------|--------|
| AWS | $1,250 | Monthly | Active |
```

## Output Structure

### Briefing Sections

1. **Executive Summary**
   - Week performance status (🟢/🟡/🔴)
   - Revenue vs target
   - Key highlights

2. **Revenue Analysis**
   - Weekly revenue table
   - Month-to-date metrics
   - Week-over-week trend

3. **Completed Tasks**
   - Grouped by category
   - Priority indicators
   - Completion dates

4. **Bottlenecks & Delays**
   - Expected vs actual duration
   - Delay reasons
   - Business impact

5. **Proactive Suggestions**
   - Cost optimization opportunities
   - Upcoming deadlines table

6. **Actions Requiring Approval**
   - Financial overruns
   - Upcoming CEO deadlines
   - Subscription renewals

### Sample Output

```markdown
# CEO Weekly Briefing

**Week 8, 2026** (2026-02-16 to 2026-02-22)

## Executive Summary

**Week 8 Performance**: 🟡 At Risk

- Revenue: $8,500 / $12,500 target (68%)
- Customers Acquired: 5
- Tasks Completed: 12

## Revenue Analysis

| Metric | Value |
|--------|-------|
| This Week | $8,500 |
| Weekly Target | $12,500 |
| Achievement | 68% |

📈 Week-over-Week: +12.5%

## Actions Requiring Approval

### 1. Infrastructure Cost Review 🟡

**Description**: AWS costs ($1,580) exceed baseline ($1,250).

**Financial Impact**: $330
```

## Approval Items

Items requiring CEO approval are written to `/Pending_Approval/`:

```markdown
---
title: Infrastructure Cost Review
urgency: Medium
deadline: 2026-02-26
financial_impact: 330
status: pending
---

# Approval Request

## Description

Infrastructure costs exceed expected threshold...

## Decision

- [ ] Approved
- [ ] Rejected
- [ ] Approved with modifications
```

## Cost Anomaly Detection

The generator detects anomalies using these thresholds:

| Category | Threshold | Action |
|----------|-----------|--------|
| Infrastructure | > $1,500/week | Flag for review |
| Tools | > $500/week | Audit subscriptions |
| Operations | > $3,000/week | Review expenses |

### Subscription Overlap Detection

Detects potential duplicate services:
- Datadog + Sentry → Monitoring overlap
- Multiple project management tools
- Redundant cloud services

## Revenue Trend Calculation

```python
trend_percentage = (this_week - previous_week) / previous_week * 100

📈 Positive trend: Growth
📉 Negative trend: Decline
➡️ Flat: < 5% change
```

## Task Categorization

Tasks are auto-categorized based on content:

| Keywords | Category |
|----------|----------|
| email, send, notify | Communication |
| code, develop, implement | Engineering |
| report, analysis, data | Analytics |
| meeting, review, planning | Management |
| budget, expense, payment | Finance |

## Logging

All briefing generations are logged to `/Logs/YYYY-MM-DD.json`:

```json
{
  "timestamp": "2026-02-19T12:08:35",
  "action_type": "BRIEFING_GENERATED",
  "actor": "skill:ceo_briefing",
  "result": "success",
  "details": {
    "week": 8,
    "year": 2026,
    "revenue": 2796.0,
    "tasks_completed": 7,
    "approvals_needed": 7
  }
}
```

## Configuration

### Revenue Targets

Set in `Business_Goals.md`:
- Weekly revenue target
- MRR target
- Customer acquisition goals

### Expense Thresholds

Modify in `ceo_briefing.py`:

```python
# Line ~280
if expenses.infrastructure_cost > 1500:  # Change threshold
```

### Subscription Monitoring

Update subscription list in `Bank_Transactions.md`:

```markdown
## Recurring Subscriptions

| Service | Monthly Cost | Next Billing | Status |
|---------|--------------|--------------|--------|
```

## Weekly Schedule

| Day | Action |
|-----|--------|
| Monday | Briefing generated for previous week |
| Tuesday | CEO reviews briefing |
| Wednesday | Approval items addressed |
| Thursday | Follow-up on action items |
| Friday | Week-end status update |

## Integration Points

### With Plan Generator

```bash
# Generate plans for tasks
python plan_generator_v2.py

# Then generate briefing
python ceo_briefing.py
```

### With MCP Action Executor

Approval items can trigger MCP actions:

```bash
# Execute approved actions
python mcp_action_executor.py --task APPROVAL_*.md
```

### With Stop Hook

Monitor briefing completion:

```bash
python stop_hook.py --task Briefing_Review.md
```

## Troubleshooting

### No Revenue Data

1. Check `Bank_Transactions.md` exists
2. Verify transaction table format matches expected pattern
3. Ensure dates are in current week

### No Tasks Showing

1. Verify files exist in `/Done/`
2. Check file modification dates are in current week
3. Ensure files are valid markdown with front matter

### Missing Approval Items

1. Check `Business_Goals.md` has deadlines table
2. Verify subscription list is populated
3. Check expense thresholds in code

## Best Practices

### 1. Update Transactions Daily

```markdown
| Date | Description | Type | Amount | Category |
|------|-------------|------|--------|----------|
```

### 2. Keep Goals Current

Update `Business_Goals.md` quarterly with new targets.

### 3. Review Approval Items Weekly

Clear `/Pending_Approval/` after CEO decisions.

### 4. Archive Old Briefings

Move old briefings to archive folder monthly.

## Security Notes

- Briefings may contain sensitive financial data
- Store in secure vault location
- Don't commit to version control
- Restrict access to executives only

```bash
# .gitignore
Briefings/
Pending_Approval/
Bank_Transactions.md
Business_Goals.md
```
