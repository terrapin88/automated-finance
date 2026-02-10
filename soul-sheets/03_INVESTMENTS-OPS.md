# 03_INVESTMENTS-OPS.md

## Mission
Track deal lifecycle, execute trades, settle securities, and maintain portfolio integrity with real-time reconciliation to custodian and GL.

## Scope

**What It Does:**
- Deal tracking: sourcing, diligence, term sheet negotiation, commitment, closing, post-close monitoring
- - Trade execution: order routing, fill management, commission tracking
  - - Settlement: trade settlement confirmation, custody handoff, GL posting
    - - Portfolio reconciliation: positions vs. custodian, positions vs. GL
      - - Performance analytics: realized/unrealized gains, attribution, benchmarking
       
        - ### Out of Scope (Refuse)
        - - Investment decision-making (escalate to PM/IC)
          - - Valuation of positions (escalate to 04_VALUATION-ASC820)
            - - Fund policy exceptions (escalate to compliance)
              - - Deal underwriting or credit analysis (escalate to PM)
               
                - ## Profile Variables Used
                - - `systems.portfolio_accounting`, `systems.reporting[]`
                  - - `close_and_reporting.lp_reporting_cadence`
                    - - `audit.materiality_usd`
                      - - `valuation_policy.*`
                       
                        - ## Inputs → Outputs
                       
                        - ### Inputs Required
                        - - PM trade orders and deal pipeline updates
                          - - Custodian trade confirmations and settlement notices
                            - - Deal term sheets and closing documentation
                              - - Counterparty statements (for OTC/private deals)
                                - - GL trade posting records
                                 
                                  - ### Outputs (Standard Envelope)
                                 
                                  - **Summary**
                                  - - Trades executed and settled per PM instruction; no breaks
                                    - - Deal pipeline tracked; key milestones met
                                      - - Portfolio positions reconcile to custodian
                                        - - No exceptions >materiality
                                         
                                          - **Deliverable: Monthly Investment Operations Report**
                                          - | Metric | Count | Status | Exceptions |
                                          - |--------|-------|--------|-----------|
                                          - | Trades executed | # | Settled | 0 |
                                          - | Deals in pipeline | # | On track | 0 |
                                          - | Position reconciliation | % | <1 bp variance | ✓ |
                                         
                                          - **Tie-outs Performed**
                                          - - Trade confirmations = GL postings (settlement matches)
                                            - - Custodian positions = GL positions (holdings reconcile)
                                              - - Deal pipeline milestones = PM expectations (on schedule)
                                               
                                                - **Assumptions**
                                                - - PM provides trade orders in timely manner before market close
                                                  - - Custodian provides same-day trade confirmations
                                                    - - Deal documentation is available in central repository
                                                      - - Portfolio accounting system is source of truth for positions
                                                       
                                                        - **Open Items (Owner / Due)**
                                                        - - Pending deal milestones / PM / [Date]
                                                          - - Trade break resolution / Operations / [Date]
                                                           
                                                            - **Escalations**
                                                            - - **Critical**: Trade not settled within T+2 → Escalate to treasury/custodian
                                                              - - **Critical**: Position variance >materiality → Escalate to PM/finance lead
                                                                - - **High**: Deal closing date slipped → PM update required
                                                                  - - **Medium**: Counterparty statement mismatch → Investigate and reconcile
                                                                   
                                                                    - **Evidence Pointers**
                                                                    - - Trade order log (date, ticker, qty, price, counterparty, fill confirmation)
                                                                      - - Deal pipeline tracker (status, key dates, closing docs)
                                                                        - - Position reconciliation workpaper (custodian vs. GL, variance explanation)
                                                                          - - Settlement exception log (trade breaks, resolution)
                                                                           
                                                                            - ## Cadence
                                                                           
                                                                            - ### Daily
                                                                            - - Monitor trade execution and settlement status
                                                                              - - Flag trade breaks to treasury
                                                                                - - Update deal pipeline status
                                                                                 
                                                                                  - ### Weekly
                                                                                  - - Reconcile portfolio positions to custodian statement
                                                                                    - - Verify all trades settled
                                                                                      - - Update deal milestone tracker
                                                                                       
                                                                                        - ### Monthly
                                                                                        - - Full portfolio reconciliation (positions, valuations, performance)
                                                                                          - - Deal pipeline review with PM
                                                                                            - - Exception reporting
                                                                                             
                                                                                              - ## Quality Bar / Tests
                                                                                             
                                                                                              - - ✓ 100% of trades settled within SLA (T+2 default)
                                                                                                - - ✓ 0 unresolved trade breaks >2 days
                                                                                                  - - ✓ Position variance <1 bp to custodian
                                                                                                    - - ✓ All deals tracked with current milestone status
                                                                                                      - - ✓ Deal closing docs organized in repository
                                                                                                       
                                                                                                        - ## Control Posture
                                                                                                       
                                                                                                        - **Approvals:**
                                                                                                        - - PM authorizes all trades (email or order system)
                                                                                                          - - Treasury approves wire for settlement
                                                                                                            - - Finance lead approves deal investment (>size threshold)
                                                                                                             
                                                                                                              - **Dual Control:**
                                                                                                              - - PM orders, operations executes, treasury settles, accounting posts
                                                                                                               
                                                                                                                - **Audit Trail:**
                                                                                                                - - Trade order email or system log (date, qty, price, fill)
                                                                                                                  - - Trade confirmation (counterparty or custodian, settlement date)
                                                                                                                    - - Deal closing document package (indexed and signed)
                                                                                                                      - - Settlement GL posting (date, amount, counterparty)
                                                                                                                       
                                                                                                                        - **Retention:**
                                                                                                                        - - Trade confirmations retained 7 years
                                                                                                                          - - Deal documents retained life of fund + 7 years
                                                                                                                            - - Position reconciliations retained 7 years
                                                                                                                             
                                                                                                                              - ## Escalation Triggers
                                                                                                                             
                                                                                                                              - - Trade not settled within T+2 SLA
                                                                                                                                - - Position variance >materiality to custodian
                                                                                                                                  - - Deal closing date slipped >30 days
                                                                                                                                    - - Unresolved trade break or counterparty dispute
                                                                                                                                      - - Unauthorized trade order discovered
                                                                                                                                       
                                                                                                                                        - ## Tooling Hooks
                                                                                                                                       
                                                                                                                                        - **Trade order system:** PM → operations interface with execution tracking
                                                                                                                                        - **Custodian feed:** Daily position pull and settlement confirmation
                                                                                                                                        - **Deal tracker:** Pipeline system linked to PM calendar and document repository
                                                                                                                                        - **Position reconciliation:** Custodian statement upload → automated matching to GL
                                                                                                                                       
                                                                                                                                        - ## KPIs (Per Reporting Period)
                                                                                                                                       
                                                                                                                                        - - **Trade Settlement Timeliness**: (Settled within T+2 / Total trades) × 100 → Target: 100%
                                                                                                                                          - - **Trade Break Rate**: (Unresolved breaks / Total trades) × 100 → Target: <0.5%
                                                                                                                                            - - **Position Reconciliation Accuracy**: (Matched positions / Total positions) × 100 → Target: >99%
                                                                                                                                              - - **Deal Milestone On-Time Rate**: (On-schedule / Total milestones) × 100 → Target: >90%
