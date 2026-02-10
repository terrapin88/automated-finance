# Roadmap: Automated Finance — From Specifications to Software

This roadmap lays out a phased plan to turn the soul sheet specification framework into a functioning back office software suite for investment managers.

---

## Guiding Principles

1. **One agent end-to-end before many agents partially.** Prove the architecture works with a single soul sheet before scaling to all 12.
2. **Deterministic math, LLM orchestration.** Calculations (NAV, waterfall, tax) are code. The LLM reads the soul sheet, decides what to run, reviews outputs, and produces the standard envelope.
3. **Integrate one system at a time.** Each external integration (GL, custodian, bank) is its own project. Start with mocks, graduate to one real provider, then abstract.
4. **Control posture is not optional.** Audit trail, dual control, and approval routing ship in every phase, not bolted on later.
5. **Evidence-first design.** Every operation produces a traceable artifact. If it isn't evidenced, it didn't happen.

---

## Phase 0 — Foundation

**Goal:** Establish project infrastructure, core data model, and the agent execution framework. Nothing user-facing yet — this is scaffolding.

### 0.1 Project Setup
- [ ] Choose tech stack and initialize repository structure
  - Recommended: Python (FastAPI backend), PostgreSQL, Redis (task queue), React or similar (frontend later)
  - Agent framework: Claude API with structured tool use
- [ ] Set up CI/CD pipeline (linting, type checking, tests on every PR)
- [ ] Set up development environment (Docker Compose for local: API, DB, Redis, mock services)
- [ ] Define coding standards, PR review process, and branch strategy

### 0.2 Core Data Model
- [ ] Design and implement the database schema for foundational entities:
  - **Org & Config**: Organization, fund entities, service providers (mirrors org_profile.yaml)
  - **Fund Structure**: Funds, LPs/investors, commitment amounts, ownership percentages
  - **Chart of Accounts**: GL accounts, account types, fund mappings
  - **Journal Entries**: Header + line items, posting status, approval status, source reference
  - **Positions**: Holdings by fund, security, quantity, cost basis, market value
  - **Transactions**: Trades, cash movements, distributions, contributions
  - **Periods**: Accounting periods with open/closed/locked status
- [ ] Build org profile loader (parse org_profile.yaml → database seed)
- [ ] Write seed data generator for development and testing (synthetic fund with realistic data)

### 0.3 Agent Execution Framework
- [ ] Build the soul sheet parser — read a soul sheet markdown file and extract structured metadata (mission, scope, inputs, outputs, quality bar, escalation triggers)
- [ ] Build the agent runner — given a parsed soul sheet and a task, construct an LLM prompt with:
  - Soul sheet context (mission, scope, constraints)
  - Org profile context (relevant variables)
  - Available tools (database queries, calculation engines, file operations)
  - Task-specific data (inputs, prior outputs, open items)
- [ ] Implement the standard envelope output parser — validate that agent output conforms to the 7-section structure (summary, deliverable, tie-outs, assumptions, open items, escalations, evidence pointers)
- [ ] Build tool interface layer — agents call tools (run_query, post_journal_entry, run_reconciliation, create_evidence, escalate) and the framework executes them with audit logging
- [ ] Implement agent audit trail — log every agent invocation, tool call, input, and output with timestamps

### 0.4 Evidence & Document Store
- [ ] Implement evidence storage service (file system or object store with metadata database)
- [ ] Define evidence schema: type, period, fund, process, created_by, file_path, hash
- [ ] Build evidence indexer — create and query the master evidence index (referenced in 11_DOCS-SCRIBE)
- [ ] Implement retention tagging (retention_years, destruction_eligible_date)

### 0.5 Task & Workflow Engine
- [ ] Build task model: task_id, type, status (pending/in_progress/blocked/completed/failed), owner, due_date, parent_task, dependencies
- [ ] Implement basic task queue (Redis-backed) for async agent execution
- [ ] Build approval model: item_id, item_type, approver_role, status (pending/approved/rejected), threshold_rule
- [ ] Implement escalation model: trigger_condition, severity, owner, status, created_at, resolved_at

**Phase 0 exit criteria:** You can load an org profile, seed a test fund with synthetic data, invoke an agent against a soul sheet, and get a structured output logged with evidence. No UI, no real integrations.

---

## Phase 1 — Fund Accounting MVP

**Goal:** Get 02_FUND-ACCOUNTING working end-to-end for a single fund. This is the center of gravity — most other agents feed into or out of fund accounting.

### 1.1 GL Engine
- [ ] Implement journal entry posting (create, approve, post, reverse)
- [ ] Build trial balance generator (sum debits/credits by account for a period)
- [ ] Implement period management (open, soft-close, hard-close, lock)
- [ ] Build sub-ledger roll-up (cash, positions, AP, AR → GL summary)
- [ ] Implement MJE (manual journal entry) workflow with approval routing

### 1.2 Reconciliation Engine
- [ ] Design reconciliation framework:
  - Configurable source pairs (GL vs. custodian, GL vs. PM system, etc.)
  - Matching rules (exact, tolerance-based with configurable bps/USD threshold)
  - Match statuses: auto-matched, manual-matched, unmatched (break)
- [ ] Implement break management: identification, classification, aging, owner assignment
- [ ] Build reconciliation report generator (matched count, break count, break aging, tolerance usage)
- [ ] Implement auto-resolution for breaks within materiality tolerance
- [ ] Build manual resolution workflow (investigate → document → resolve → approve)

### 1.3 NAV Calculation Engine
- [ ] Implement NAV calculator:
  - Total assets (positions at market value + cash + receivables)
  - Total liabilities (AP + accrued expenses + payables)
  - NAV = assets - liabilities
  - NAV per unit/share
- [ ] Build NAV variance check (vs. prior period, vs. materiality threshold from org profile)
- [ ] Implement NAV approval workflow (fund manager + accounting sign-off)
- [ ] Build NAV audit trail (calculation inputs, intermediate values, final NAV, approvers)

### 1.4 Data Import Adapters
- [ ] Build generic import framework (CSV/XLSX/JSON → staging table → validation → core table)
- [ ] Implement mock custodian adapter (generate realistic position/transaction files)
- [ ] Implement mock GL adapter (for testing against synthetic GL data)
- [ ] Build import validation rules (required fields, referential integrity, duplicate detection)
- [ ] Implement import audit trail (file received, rows parsed, rows accepted/rejected, errors)

### 1.5 Fund Accounting Agent Integration
- [ ] Wire the FUND-ACCOUNTING soul sheet agent to the GL, reconciliation, and NAV engines
- [ ] Implement the agent's tool set:
  - `query_trial_balance(fund, period)` → trial balance data
  - `run_reconciliation(source_a, source_b, tolerance)` → reconciliation report
  - `calculate_nav(fund, period)` → NAV result
  - `post_journal_entry(entries, description, source)` → JE reference
  - `get_breaks(fund, period, status)` → break list
  - `create_evidence(type, period, fund, content)` → evidence reference
  - `escalate(severity, description, owner)` → escalation record
- [ ] Build close checklist runner — agent walks through period-end close steps, calls tools, produces standard envelope output
- [ ] Implement quality bar validation — automated checks against the soul sheet's quality bar criteria after agent completes

### 1.6 Basic Review UI
- [ ] Build minimal web UI:
  - Dashboard: current period status, open tasks, pending approvals
  - Close checklist view: step-by-step with status indicators
  - Reconciliation viewer: matched/unmatched items, break details
  - NAV review: calculation breakdown, variance analysis, approve/reject
  - Journal entry viewer: entries by period, approval status
  - Evidence viewer: list evidence by period/type, download originals
- [ ] Implement role-based access (Controller, CFO, Finance Analyst — read/write/approve permissions)
- [ ] Build notification system (email or in-app) for pending approvals, escalations, deadline warnings

**Phase 1 exit criteria:** You can run a full monthly close for a single synthetic fund — import positions and transactions, post to GL, reconcile, calculate NAV, approve, and produce a close package with evidence. One human reviewer approves through the UI.

---

## Phase 2 — Orchestration & Adjacent Agents

**Goal:** Bring FIN-ORCH online as the orchestrator and add the agents that directly support fund accounting: TREASURY-CUSTODY, INVESTMENTS-OPS, and VALUATION-ASC820.

### 2.1 FIN-ORCH Orchestrator
- [ ] Implement request intake system — structured form for finance work requests (what, why, scope, deadline)
- [ ] Build task planner — FIN-ORCH agent decomposes a request into subtasks, assigns to subagents, identifies dependencies and critical path
- [ ] Implement task dependency graph — sequential and parallel task execution with blocking dependencies
- [ ] Build output consolidation — collect subagent standard-envelope outputs into a single coherent deliverable
- [ ] Implement QA gate runner — FIN-ORCH validates consolidated output against quality bars before delivery
- [ ] Build request lifecycle tracking (intake → planning → execution → QA → delivery → archive)

### 2.2 Treasury & Custody Agent (01)
- [ ] Build wire request workflow (initiate → validate beneficiary → threshold check → approval routing → execute → confirm)
- [ ] Implement beneficiary management (KYC/AML document tracking, approval status, expiration monitoring)
- [ ] Build cash position tracker (aggregate across banks/custodians, daily sweep logic)
- [ ] Implement custody reconciliation (positions and cash vs. custodian statements)
- [ ] Wire agent to tools: `submit_wire_request`, `check_beneficiary_status`, `get_cash_positions`, `run_custody_reconciliation`

### 2.3 Investments Ops Agent (03)
- [ ] Build trade lifecycle tracker (order → execution → confirmation → settlement → GL posting)
- [ ] Implement deal pipeline tracker (stage, milestones, key dates, closing docs)
- [ ] Build position reconciliation (internal positions vs. custodian holdings)
- [ ] Implement trade break detection and resolution workflow
- [ ] Wire agent to tools: `get_trades`, `get_positions`, `reconcile_positions`, `log_trade_break`, `update_deal_status`

### 2.4 Valuation Agent (04)
- [ ] Build pricing engine:
  - Level 1: Exchange/market price lookup (from pricing feed)
  - Level 2: Broker/pricing service quote (from feed or manual input)
  - Level 3: PM mark submission and independent review workflow
- [ ] Implement ASC 820 hierarchy classifier (assign level per position based on pricing source availability)
- [ ] Build stale price detection (configurable thresholds: 1 day liquid, 1 week illiquid)
- [ ] Implement valuation report generator (position list with prices, levels, sources, variances)
- [ ] Build fair-value-to-NAV tie-back validation
- [ ] Wire agent to tools: `get_prices`, `classify_hierarchy`, `check_stale_prices`, `get_pm_marks`, `generate_valuation_report`

### 2.5 Multi-Agent Close Cycle
- [ ] Implement the full monthly close as an orchestrated workflow:
  1. FIN-ORCH receives "monthly close" request
  2. TREASURY-CUSTODY reconciles cash and custody positions
  3. INVESTMENTS-OPS reconciles portfolio positions and settles trades
  4. VALUATION-ASC820 prices all positions and produces valuation report
  5. FUND-ACCOUNTING posts final entries, runs reconciliations, calculates NAV
  6. FIN-ORCH consolidates outputs, runs QA gates, delivers close package
- [ ] Build close calendar with deadline tracking and SLA monitoring
- [ ] Implement cross-agent data passing (output of one agent becomes input to next)

**Phase 2 exit criteria:** A monthly close runs as an orchestrated multi-agent workflow. FIN-ORCH coordinates 4 agents, each producing standard-envelope outputs. The close package is consolidated and delivered through the UI with full evidence trail.

---

## Phase 3 — Reporting, Tax & Compliance

**Goal:** Add the agents that consume close data to produce investor-facing and regulatory outputs.

### 3.1 LP Reporting Agent (05)
- [ ] Build capital account engine:
  - Track contributions, distributions, income allocations per LP per period
  - Ending balance = opening + contributions + allocations - distributions
  - Validate: sum of all LP accounts = fund NAV
- [ ] Implement waterfall/carried interest calculator:
  - Parse LPA terms (preferred return, carry %, catch-up, clawback)
  - Calculate allocation waterfall step-by-step
  - Produce allocation schedule with audit trail
- [ ] Build performance calculator (IRR, MOIC, TWR from cash flow series)
- [ ] Implement LP statement generator (PDF template populated from capital account data)
- [ ] Build board package assembler (aggregate fund performance, risk, compliance, open items)
- [ ] Wire agent to tools: `get_capital_accounts`, `calculate_waterfall`, `calculate_performance`, `generate_lp_statement`, `assemble_board_package`

### 3.2 Tax & Withholding Agent (06)
- [ ] Build withholding calculator:
  - Federal backup withholding rates by investor classification
  - State withholding rates by investor domicile
  - NRA treaty withholding rates by country and distribution type
- [ ] Implement W-9/W-8BEN document tracker (collection status, expiration dates, renewal alerts)
- [ ] Build 1099/K-1 data aggregator (reconcile distributions to GL by investor, type, and period)
- [ ] Implement withholding remittance tracker (liability accounts, payment schedules, agency confirmations)
- [ ] Wire agent to tools: `calculate_withholding`, `get_w9_status`, `aggregate_1099_data`, `track_remittance`

### 3.3 Compliance & Controls Agent (07)
- [ ] Build AML/KYC tracker (investor documentation status, refresh calendar, gap alerts)
- [ ] Implement sanctions screening integration point (API interface; mock initially, real provider later)
- [ ] Build fund limit monitor:
  - Leverage ratio calculator (borrowings / NAV)
  - Concentration calculator (position value / NAV per holding)
  - Liquidity ratio calculator
  - Compare all ratios to LPA/prospectus limits
- [ ] Implement control testing framework (test procedure templates, sample generation, evidence capture, exception tracking)
- [ ] Build regulatory filing tracker (Form PF, Form ADV — deadline tracking, draft prep, submission confirmation)
- [ ] Wire agent to tools: `check_kyc_status`, `screen_sanctions`, `calculate_fund_limits`, `run_control_test`, `track_filing`

### 3.4 AP & Expenses Agent (08)
- [ ] Build invoice intake pipeline (OCR/parsing for PDF invoices; manual entry fallback)
- [ ] Implement 3-way matching engine (PO ↔ invoice ↔ receipt matching with configurable tolerance)
- [ ] Build expense allocation engine (apply formula from org profile to split invoice across funds)
- [ ] Implement vendor master (approved vendors, tax IDs, W-9 status, payment instructions)
- [ ] Build AP aging report generator (current, 30-60, 60-90, 90+ with escalation flags)
- [ ] Implement payment file generator (create wire/ACH batch file for treasury execution)
- [ ] Wire agent to tools: `intake_invoice`, `match_invoice`, `allocate_expense`, `check_vendor`, `generate_payment_file`, `get_ap_aging`

**Phase 3 exit criteria:** A full quarter-end cycle runs: monthly closes for 3 months, then quarterly LP statements, tax withholding calculations, compliance report, and board package — all orchestrated by FIN-ORCH.

---

## Phase 4 — Real Integrations

**Goal:** Replace mock data adapters with connections to real financial systems. Each integration is independent and can be prioritized based on the target customer's systems.

### 4.1 GL System Integrations
- [ ] **QuickBooks Online** adapter (REST API: chart of accounts, journal entries, trial balance, vendors, invoices)
- [ ] **NetSuite** adapter (SuiteTalk/REST API: same scope)
- [ ] **Sage Intacct** adapter (API: same scope)
- [ ] Generic CSV/XLSX import for unsupported GL systems

### 4.2 Custodian Integrations
- [ ] **Fidelity** adapter (SFTP file feeds: positions, transactions, cash balances)
- [ ] **BNY Mellon** adapter (SFTP/API: same scope)
- [ ] **Schwab** adapter (SFTP: same scope)
- [ ] Generic custodian file parser (configurable column mapping for CSV/XLSX feeds)

### 4.3 Banking Integrations
- [ ] Bank statement ingestion (BAI2 format parser, MT940 parser, PDF statement OCR)
- [ ] Wire initiation API (bank-specific; start with one major bank's API)
- [ ] ACH file generation (NACHA format)

### 4.4 Pricing & Market Data
- [ ] **Bloomberg** adapter (B-PIPE or Data License for daily prices)
- [ ] **FactSet** adapter (API for pricing, reference data)
- [ ] **Exchange feeds** (free/delayed feeds for Level 1 pricing)
- [ ] Manual price entry interface for Level 3 / illiquid assets

### 4.5 Compliance & Screening
- [ ] **LexisNexis / Accuity** sanctions screening API integration
- [ ] **OFAC SDN list** direct download and local screening
- [ ] **SEC EDGAR** integration for Form PF / Form ADV filing

### 4.6 Document & Communication
- [ ] **SharePoint / OneDrive** integration for evidence storage
- [ ] **Google Drive** integration (alternative evidence storage)
- [ ] **Email** integration for LP statement distribution and investor query intake
- [ ] **PDF generation** engine for LP statements, board packages, audit workpapers

### Integration Architecture Notes
- Every integration should implement a standard adapter interface: `connect()`, `pull(entity_type, date_range)`, `push(entity_type, payload)`, `health_check()`
- All ingested data lands in staging tables first, then validated and promoted to core tables
- Every integration call is logged (timestamp, direction, payload hash, row count, errors)
- Credentials stored in secrets manager (Vault, AWS Secrets Manager, or similar) — never in config files

**Phase 4 exit criteria:** At least one real GL, one real custodian, and one real bank are connected. Data flows automatically on schedule. Manual CSV import remains as fallback for everything else.

---

## Phase 5 — QA, Audit & Documentation Agents

**Goal:** Bring the quality assurance and documentation layers online.

### 5.1 QA & Auditor Agent (10)
- [ ] Build analytical procedures engine:
  - Period-over-period variance analysis (GL accounts, NAV, positions)
  - Budget-to-actual comparison
  - Anomaly detection (statistical outliers on transaction amounts, frequencies)
- [ ] Implement substantive testing framework:
  - Sample selection engine (random, stratified, risk-based)
  - Vouching workflow (trace sample items to source documents)
  - Exception documentation and follow-up tracking
- [ ] Build control testing executor:
  - Test procedure runner (pull sample, check control evidence, document result)
  - Control matrix generator (control objective, test procedure, frequency, last test date, result)
- [ ] Implement audit workpaper generator (structured PDF with procedure, sample, findings, conclusion)
- [ ] Build external audit coordination (PBC list tracking, document request fulfillment, auditor query log)

### 5.2 Documentation & Scribe Agent (11)
- [ ] Build process documentation generator (auto-generate runbooks from soul sheet + actual system configuration)
- [ ] Implement evidence index manager (master index of all evidence by fund, period, type, with completeness checks)
- [ ] Build retention enforcement (flag documents approaching destruction date, require compliance sign-off for destruction)
- [ ] Implement version control for policies and procedures (track changes, approval dates, review dates)
- [ ] Build audit readiness dashboard (evidence completeness %, overdue reviews, policy currency)

### 5.3 Data Pipelines Agent (09)
- [ ] Build pipeline orchestrator (scheduled jobs for each data source per org profile cadence)
- [ ] Implement data quality rule engine (configurable checks: nulls, ranges, referential integrity, duplicates)
- [ ] Build warehouse reconciliation (compare warehouse aggregates to source system totals)
- [ ] Implement data lineage tracker (source row → transformation → warehouse row, with transformation metadata)
- [ ] Build pipeline monitoring dashboard (last run, row counts, error rates, SLA compliance)

**Phase 5 exit criteria:** A quarterly external audit preparation can be run: QA agent executes analytical procedures and substantive tests, DOCS-SCRIBE assembles the evidence index, and a complete PBC (prepared by client) list is fulfilled with linked workpapers.

---

## Phase 6 — Production Hardening

**Goal:** Make the system production-ready for real firms with real money.

### 6.1 Security
- [ ] Implement comprehensive RBAC (roles: Admin, CFO, Controller, Finance Analyst, Compliance Officer, PM, Auditor, Read-Only)
- [ ] Add multi-factor authentication
- [ ] Implement field-level encryption for sensitive data (SSN, tax IDs, bank account numbers)
- [ ] Add secrets management (API keys, database credentials, integration credentials)
- [ ] Implement IP allowlisting and session management
- [ ] Conduct penetration testing and vulnerability assessment
- [ ] Implement CSP, CORS, and other web security headers

### 6.2 Compliance & Certifications
- [ ] SOC 2 Type I preparation (security, availability, confidentiality controls)
- [ ] SOC 2 Type II audit (12-month observation period)
- [ ] SOC 1 / SSAE 18 if processing transactions on behalf of clients
- [ ] Data processing agreement (DPA) template for customers
- [ ] Privacy policy and terms of service

### 6.3 Reliability & Operations
- [ ] Implement comprehensive monitoring (application metrics, error rates, latency, agent success rates)
- [ ] Build alerting rules (pipeline failures, SLA breaches, escalation timeouts, system errors)
- [ ] Implement database backup and point-in-time recovery
- [ ] Build disaster recovery plan and test it
- [ ] Implement rate limiting and graceful degradation for LLM API failures
- [ ] Add health check endpoints and uptime monitoring
- [ ] Build operational runbook for on-call support

### 6.4 Scalability
- [ ] Multi-tenancy support (multiple firms on shared infrastructure with data isolation)
- [ ] Horizontal scaling for agent execution (queue-based worker scaling)
- [ ] Database connection pooling and query optimization
- [ ] Caching layer for frequently accessed data (trial balances, position snapshots)
- [ ] Background job management for long-running operations (close cycles, report generation)

### 6.5 User Experience Polish
- [ ] Onboarding wizard (walk new firm through org profile setup, system connections, initial data import)
- [ ] In-app help and documentation (contextual guidance linked to soul sheet definitions)
- [ ] Keyboard shortcuts and power-user features
- [ ] Mobile-responsive design for approval workflows
- [ ] Export capabilities (Excel, PDF, CSV for all reports and workpapers)
- [ ] Audit log viewer (searchable history of all system actions)

**Phase 6 exit criteria:** The system passes SOC 2 Type I readiness assessment. A real firm can onboard, connect their systems, and run a close cycle with confidence that their data is secure and the system is reliable.

---

## Phase Summary

| Phase | Focus | Key Milestone |
|-------|-------|--------------|
| **0** | Foundation | Agent framework runs, synthetic data loads, evidence stores |
| **1** | Fund Accounting MVP | Single-fund monthly close works end-to-end |
| **2** | Orchestration | Multi-agent close cycle coordinated by FIN-ORCH |
| **3** | Reporting + Compliance | Full quarter-end with LP statements, tax, compliance |
| **4** | Real Integrations | Connected to real GL, custodian, and bank |
| **5** | QA + Audit | Audit preparation and evidence assembly automated |
| **6** | Production | SOC 2 ready, multi-tenant, monitored, secure |

## Key Architectural Decisions to Make Before Starting

These decisions shape everything downstream and should be resolved before Phase 0 implementation begins:

1. **Deployment model**: SaaS (multi-tenant cloud) vs. single-tenant vs. on-prem? This affects data isolation, security model, and infrastructure complexity.

2. **LLM provider strategy**: Claude-only vs. multi-provider? How to handle API rate limits, failures, and cost management? What's the fallback when the LLM is down — does the system degrade to manual-only?

3. **Agent autonomy level**: How much can agents do without human approval? The soul sheets define escalation triggers, but the default posture (auto-execute vs. human-in-the-loop for everything) is a product decision.

4. **Data residency**: Financial data has regulatory requirements. Where does data live? How do you handle firms in different jurisdictions?

5. **Pricing model**: Per-fund? Per-entity? Per-agent-invocation? This affects how you meter usage and what you optimize for.

6. **Open source vs. proprietary**: The specs are MIT-licensed. Is the software? This affects community contribution, competitive moat, and go-to-market.
