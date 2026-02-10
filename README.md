# automated-finance

A specification framework for AI-driven financial operations at investment managers, RIAs, PE/VC funds, and hedge funds. It defines a system of coordinated AI agents ("soul sheets") that execute finance work — close, NAV, custody, tax, compliance, reporting — with audit-ready outputs and clear governance.

## How It Works

The framework has three layers:

1. **Org Profile** (`profiles/org_profile.template.yaml`) — A single YAML file capturing firm-specific facts: entities, systems, service providers, thresholds, policies, and team structure. This is the configuration that makes the agents portable across different firms.

2. **Soul Sheets** (`soul-sheets/`) — Each soul sheet is a complete specification for one specialized AI agent. It defines the agent's mission, scope, inputs/outputs, cadence, quality bar, control posture, escalation triggers, and KPIs.

3. **Orchestrator + Intake** (`prompts/`) — FIN-ORCH is the master orchestrator that triages finance requests, assigns work to subagents, and consolidates outputs. The intake prompts onboard a new firm by interviewing for org facts and generating a completed org profile.

## Soul Sheet Agents

| # | Agent | Responsibility |
|---|-------|---------------|
| 00 | **FIN-ORCH** | Orchestrator — intake, triage, task planning, QA gates |
| 01 | **TREASURY-CUSTODY** | Cash flows, banking, wires, custody reconciliation |
| 02 | **FUND-ACCOUNTING** | GL, NAV, period close, reconciliations |
| 03 | **INVESTMENTS-OPS** | Deal tracking, trade execution, settlement |
| 04 | **VALUATION-ASC820** | Fair value, pricing sources, ASC 820 hierarchy |
| 05 | **REPORTING-LP** | LP statements, performance, board packages |
| 06 | **TAX-WITHHOLDING** | 1099/K-1, W-9/W-8BEN, withholding remittance |
| 07 | **COMPLIANCE-CONTROLS** | AML/KYC, sanctions, fund governance, Form PF |
| 08 | **AP-EXPENSES** | Invoice processing, 3-way match, vendor management |
| 09 | **DATA-PIPELINES** | ETL ingestion, data quality, warehouse reconciliation |
| 10 | **QA-AUDITOR** | Analytical procedures, control testing, audit support |
| 11 | **DOCS-SCRIBE** | Documentation, evidence management, retention compliance |

## Standard Output Envelope

Every agent output follows the same structure for auditability:

- **Summary** (3 bullets)
- **Deliverable** (tabular/structured)
- **Tie-outs performed**
- **Assumptions**
- **Open items** (owner / due date)
- **Escalations** (critical / high / medium)
- **Evidence pointers**

## Getting Started

1. **Copy the org profile template**: `cp profiles/org_profile.template.yaml profiles/org_profile.yaml`
2. **Run the intake interview**: Use `prompts/intake/fin-orch.intake.prompt.md` to walk through the intake questions and populate your org profile
3. **Tailor the soul sheets**: Use `prompts/generation/generate-soul-sheets.prompt.md` to inject org-specific details (systems, cadences, thresholds) into the soul sheet templates

## Repository Structure

```
automated-finance/
  profiles/
    org_profile.template.yaml   # Org configuration schema
  prompts/
    intake/                     # FIN-ORCH intake interview
    generation/                 # Soul sheet tailoring prompt
  soul-sheets/                  # 12 agent specifications (00-11)
  templates/
    soul-sheet.template.md      # Blank soul sheet template
```

## License

MIT
