# DIGITAL-FTE
An AI Powered Application which manages all task
🚀 Personal AI Employee – Autonomous Digital FTE (Bronze → Platinum)

Hackathon: Personal AI Employee Hackathon 0
Tagline: Your life and business on autopilot. Local-first, agent-driven, human-in-the-loop.

🧠 Project Overview

This project implements a fully autonomous Digital FTE (Full-Time Equivalent) — an AI-powered employee that proactively manages personal and business operations using:

Claude Code as the reasoning engine

Obsidian as the local knowledge base and dashboard

Python Watchers for perception

MCP Servers for real-world actions

Human-in-the-Loop (HITL) safeguards for security

Cloud + Local split architecture (Platinum Tier)

This system transforms AI from a chatbot into a proactive operational partner capable of:

Monitoring Gmail & WhatsApp

Managing accounting via Odoo

Posting to social media

Generating invoices

Performing weekly audits

Producing Monday Morning CEO Briefings

Running 24/7 autonomously

🏆 Tier Achievement

✅ Bronze – Foundation
✅ Silver – Functional Assistant
✅ Gold – Autonomous Employee
✅ Platinum – Always-On Cloud + Local Executive

This repository represents the Platinum Tier Completion.

🏗 Architecture Overview

This system follows a Perception → Reasoning → Action → Persistence model.

1️⃣ Perception Layer (Watchers)

Python-based daemon processes monitor external systems and create structured markdown files in the vault.

Watchers Implemented

Gmail Watcher (Google API)

WhatsApp Watcher (Playwright automation)

File System Watcher

Finance Watcher (Bank CSV / API ingestion)

Each watcher:

Runs continuously

Creates .md files inside /Needs_Action

Logs activity

Handles retries with exponential backoff

Watchers are managed using PM2 for:

Auto-restart

Boot persistence

Log management

2️⃣ Knowledge & State Layer (Obsidian Vault)

The vault acts as:

Long-term memory

GUI dashboard

Workflow engine

Audit system

Core Structure
/Vault
 ├── Dashboard.md
 ├── Company_Handbook.md
 ├── Business_Goals.md
 ├── /Needs_Action/
 ├── /Plans/
 ├── /Pending_Approval/
 ├── /Approved/
 ├── /Rejected/
 ├── /Done/
 ├── /Logs/
 ├── /Briefings/
 ├── /Accounting/
 ├── /Invoices/
 ├── /Updates/ (Cloud → Local sync)
 ├── /In_Progress/<agent>/
Key Design Principles

File-based workflow

Claim-by-move rule

Single-writer rule for Dashboard.md

Markdown as structured state

Human-readable audit trail

3️⃣ Reasoning Layer (Claude Code)

Claude Code acts as the central AI reasoning engine.

Responsibilities:

Reads /Needs_Action

Generates /Plans

Writes approval requests

Updates dashboard

Creates CEO Briefings

Performs subscription audits

Executes multi-step reasoning loops

Ralph Wiggum Loop

Used for multi-step autonomy.

Execution continues until:

File moved to /Done
OR

<promise>TASK_COMPLETE</promise> detected

This prevents incomplete reasoning cycles.

4️⃣ Action Layer (MCP Servers)

Claude does not directly execute external actions. Instead, it uses MCP servers.

MCP Servers Implemented

Email MCP

Browser MCP

Calendar MCP

Social Media MCP

Odoo MCP (JSON-RPC integration)

Filesystem MCP

All sensitive actions require approval.

5️⃣ Human-in-the-Loop (HITL)

For safety:

Sensitive actions create approval files:

/Pending_Approval/

User moves file to:

/Approved/

/Rejected/

Only then does the Orchestrator execute via MCP.

Approval always required for:

New payments

New email recipients

Banking actions

Social replies

6️⃣ Platinum Cloud + Local Architecture

Work-Zone Specialization:

Cloud:

Email triage

Draft replies

Social post drafts

Accounting drafts

Local:

WhatsApp session

Payments

Final approvals

Send/post execution

Vault Sync

Phase 1:

Git or Syncthing

Markdown-only sync

Secrets excluded

Secrets never leave local machine.

🔁 Weekly Autonomous CEO Briefing

Every Sunday:

Claude reads:

Business_Goals.md

Accounting data

Completed tasks

Calculates:

Weekly revenue

Bottlenecks

Cost anomalies

Generates:

/Briefings/YYYY-MM-DD_Monday_Briefing.md

Includes:

Revenue metrics

Performance insights

Subscription audit flags

Proactive recommendations

🧾 Odoo Integration (Gold & Platinum)

Odoo Community (self-hosted) deployed on cloud VM.

Integrated via JSON-RPC using MCP server.

Capabilities:

Draft invoices

Fetch accounting data

Revenue analysis

Expense categorization

All posting actions require local approval.

🔐 Security Architecture
Credential Handling

No secrets stored in vault

.env file (gitignored)

Environment variables for APIs

OS credential managers used for sensitive tokens

Monthly credential rotation

Example .env:

GMAIL_CLIENT_ID=
GMAIL_CLIENT_SECRET=
BANK_API_TOKEN=
WHATSAPP_SESSION_PATH=
ODOO_API_KEY=
Safeguards

DRY_RUN mode for development

Rate limiting (max emails/hour)

HITL mandatory for payments

Audit logs retained 90+ days

Audit Log Format

Stored in:

/Logs/YYYY-MM-DD.json

Includes:

Timestamp

Action type

Target

Parameters

Approval status

Result

⚙️ Setup Instructions
1️⃣ Prerequisites

Install:

Claude Code (Pro subscription)

Obsidian v1.10.6+

Python 3.13+

Node.js v24+

Git

PM2

Minimum:

8GB RAM

20GB storage

Recommended:

16GB RAM

SSD

Dedicated mini-PC or Cloud VM

2️⃣ Clone Repository
git clone <repo-url>
cd personal-ai-employee
3️⃣ Create Obsidian Vault

Create vault named:

AI_Employee_Vault

Point Claude Code to this directory.

4️⃣ Install Dependencies
Python
pip install -r requirements.txt
Node
npm install
5️⃣ Configure MCP

Edit:

~/.config/claude-code/mcp.json

Add configured servers.

6️⃣ Setup Environment Variables

Create .env file (DO NOT COMMIT):

cp .env.example .env

Add credentials.

7️⃣ Start Watchers

Using PM2:

pm2 start gmail_watcher.py --interpreter python3
pm2 start whatsapp_watcher.py --interpreter python3
pm2 start orchestrator.py --interpreter python3
pm2 save
pm2 startup
8️⃣ Start Ralph Loop
/ralph-loop "Process /Needs_Action until complete" --max-iterations 10
📊 Demo Flow (Platinum Passing Gate)

Email arrives (Local offline)

Cloud drafts reply

Writes approval file

Local user approves

Email sent via MCP

Log written

Files moved to /Done

📁 Repository Structure
/watchers
/mcp_servers
/orchestrator
/ralph_loop
/docs
README.md
.env.example
requirements.txt
package.json
📌 Innovation Highlights

Local-first privacy architecture

File-based workflow engine

Claim-by-move distributed coordination

Human-in-the-loop safety enforcement

Cloud + Local split autonomy

Self-auditing CEO Briefing generation

Autonomous subscription cost detection

🧠 Lessons Learned

File-based workflows are powerful and transparent

HITL is essential for financial automation

Process management is mandatory for 24/7 autonomy

Local-first drastically improves privacy and control

Vault sync must never include secrets

⚖️ Ethical & Responsible Automation

This system:

Logs all actions

Requires human approval for sensitive tasks

Discloses AI-generated emails (optional signature)

Avoids autonomous legal/medical decisions

Never auto-approves new financial recipients

Human remains accountable.

📽 Demo Video

Included in submission.

Shows:

WhatsApp invoice flow

Email draft approval

Odoo integration

CEO Briefing generation

Cloud + Local coordination

📝 Submission Details

Tier: Platinum

Security disclosure included
Documentation complete
Architecture fully implemented

🔮 Future Enhancements

A2A direct agent messaging

End-to-end encryption of vault

Real-time financial anomaly detection

Multi-agent specialization

Production-grade monitoring dashboard

👨‍💻 Built With

Claude Code

Obsidian

Python

Node.js

Playwright

Odoo Community

MCP Protocol

🎯 Final Statement

This project demonstrates that a Digital FTE can:

Work 24/7

Reduce operational costs by 85–90%

Maintain auditability

Preserve human oversight

Operate across personal and business domains

It is not a chatbot.

It is a fully autonomous AI Employee.

