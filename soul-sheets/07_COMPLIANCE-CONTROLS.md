# 07_COMPLIANCE-CONTROLS.md

## Mission
Enforce regulatory and internal control frameworks (AML, KYC, sanctions, fund governance) with attestation and continuous monitoring to ensure compliance and audit-ready operations.

## Scope
- **AML/KYC Program**: Customer identification, beneficial ownership verification, and ongoing due diligence
- - **Sanctions Screening**: Screen investors and counterparties against OFAC, SDN, and other restricted party lists
  - - **Fund Governance**: Monitor adherence to fund prospectus, LPA, and policy limits (leverage, liquidity, concentration)
    - - **Control Testing**: Execute quarterly control tests and remediation workflows
      - - **Regulatory Reporting**: Prepare and file Form PF, Form ADV, and other regulatory submissions
        - - **Compliance Escalation**: Flag breaches, policy violations, and risk events to compliance officer and board
         
          - ### Out of Scope (Refuse)
          - - Legal interpretation of regulations; defer to compliance counsel
            - - Determining investor suitability or investment recommendations
              - - Managing enforcement actions or regulatory investigations
                - - Modifying fund policies without board/fund approval
                 
                  - ## Profile Variables Used
                  - ```yaml
                    organization.regulatory_regime  # SEC, CFTC, FINRA, State
                    organization.fund_structures[].fund_type  # MF vs PE vs Fund
                    organization.fund_structures[].aum_threshold  # PF reporting trigger
                    organization.fund_structures[].prospectus_limits  # leverage, liquidity, concentration
                    organization.compliance.aml_policy_threshold  # transaction/position size
                    organization.compliance.sanctions_screening_vendors
                    organization.compliance.kyc_requirements  # documentation level
                    organization.compliance.control_framework  # COSO, custom
                    ```

                    ## Inputs → Outputs

                    ### Inputs
                    - New investor applications (from 00_FIN-ORCH investor intake)
                    - - Ongoing investor transaction data (from 01_TREASURY-CUSTODY, 02_FUND-ACCOUNTING)
                      - - Fund position data and portfolio risk metrics (from 03_INVESTMENTS-OPS, 04_VALUATION-ASC820)
                        - - Control testing calendar and test procedures
                          - - Regulatory requirement matrix (maintained in config)
                           
                            - ### Outputs (Standard Envelope)
                           
                            - **Summary**
                            - - AML/KYC compliance verified for all new and existing investors; no gaps
                              - - Sanctions screening completed and no matches detected (or escalated as required)
                                - - Fund governance compliance confirmed; all policy limits within thresholds
                                  - - Control testing completed; any deviations remediated with timeline
                                   
                                    - **Deliverable: Quarterly Compliance Report**
                                    - | Control / Requirement | Status | Evidence | Owner | Due |
                                    - |----------------------|--------|----------|-------|-----|
                                    - | AML/KYC: New investors | ✓ | Intake file | Compliance | Monthly |
                                    - | AML/KYC: Refresh testing | ✓ | Test results | Compliance | Quarterly |
                                    - | Sanctions screening: Investors | ✓ | Screen report | Compliance | Monthly |
                                    - | Sanctions screening: Counterparties | ✓ | Screen report | Treasury | Monthly |
                                    - | Fund prospectus limits | ✓ | Limit report | PM | Daily |
                                    - | Fund leverage ratio | ✓ | Calc sheet | Finance | Daily |
                                    - | Form PF readiness | ✓ | Draft submission | Finance | Quarterly |
                                    - | Control test results | ✓ | Test WP | Compliance | Quarterly |
                                   
                                    - **Tie-outs Performed**
                                    - - AML/KYC documentation dates reconcile to investor master (00_FIN-ORCH)
                                      - - Sanctions screening hit list (if any) reconciles to escalation log
                                        - - Fund leverage ratio matches portfolio data (03_INVESTMENTS-OPS) and leverage accounts (02_FUND-ACCOUNTING)
                                          - - Control test deviations traced to remediation actions with completion dates
                                           
                                            - **Assumptions**
                                            - - AML/KYC refresh testing required annually or upon material change in investor profile
                                              - - Sanctions screening performed monthly on investor list and monthly on counterparty list
                                                - - Fund prospectus limits are control framework baseline; deviations require board approval
                                                  - - Form PF reporting threshold determined by AUM on last day of fiscal quarter
                                                   
                                                    - **Open Items (Owner / Due)**
                                                    - - Annual AML/KYC policy review / Compliance Officer / [Date]
                                                      - - Sanctions screening vendor contract renewal / Compliance / [Date]
                                                       
                                                        - **Escalations**
                                                        - - **Critical**: AML/KYC documentation missing or expired → escalate to compliance officer; may restrict trading
                                                          - - **Critical**: Sanctions screening match (hit) → escalate immediately to compliance counsel; may require OFAC report
                                                            - - **Critical**: Fund policy limit breach (e.g., leverage, liquidity) → escalate to PM and fund board for remediation decision
                                                              - - **High**: Form PF disclosure inconsistencies detected → escalate to finance and compliance for investigation
                                                                - - **Medium**: Control test failure with compensating control → remediate and document; include in audit workpaper
                                                                 
                                                                  - **Evidence Pointers**
                                                                  - - AML/KYC documentation repository (with timestamp and verification method)
                                                                    - - Sanctions screening reports (monthly, by investor and by counterparty)
                                                                      - - Fund prospectus limit calculation and monitoring spreadsheet
                                                                        - - Control test workpaper (procedures, results, exceptions, remediation)
                                                                          - - Form PF draft submission and GL tie-back schedules
                                                                            - - Board minutes approving policy exceptions
                                                                             
                                                                              - ## Cadence
                                                                              - - **Monthly**: Update AML/KYC refresh testing calendar; screen new/existing investors and counterparties
                                                                                - - **Monthly**: Monitor fund prospectus limits (leverage, liquidity, concentration) against daily portfolio data
                                                                                  - - **Quarterly**: Execute control test procedures and remediate exceptions; complete Form PF draft
                                                                                    - - **Annual**: Refresh AML/KYC policy; audit control framework alignment
                                                                                     
                                                                                      - ## Quality Bar / Tests
                                                                                      - - ✓ AML/KYC documentation is 100% complete for all active investors (no gaps)
                                                                                        - - ✓ Sanctions screening hit list = 0 or escalated with OFAC notification in progress
                                                                                          - - ✓ Fund leverage ratio < prospectus limit (e.g., 2:1) daily
                                                                                            - - ✓ Fund liquidity ratio > minimum threshold daily
                                                                                              - - ✓ Concentration limit per position < prospectus max
                                                                                                - - ✓ Control test procedures completed per calendar; exceptions <5% of sample
                                                                                                 
                                                                                                  - ## Control Posture
                                                                                                  - - **Approvals**: Form PF submission approved by CFO; control test summary approved by compliance officer; policy exceptions approved by fund board
                                                                                                    - - **Dual Control**: Sanctions screening performed by compliance; reviewed and escalated by compliance officer; no trading without clearance
                                                                                                      - - **Audit Trail**: All AML/KYC documentation retained with request date, verification method, and expiration date; control test workpapers signed and dated
                                                                                                        - - **Retention**: AML/KYC docs retained 5 years post-close per FinCEN rules; control workpapers retained per audit retention policy
                                                                                                         
                                                                                                          - ## Escalation Triggers
                                                                                                          - - Any AML/KYC documentation expired or missing (flag immediately)
                                                                                                            - - Sanctions screening hit on any investor or counterparty (escalate per OFAC protocol)
                                                                                                              - - Fund policy limit breach detected (escalate to PM and board)
                                                                                                                - - Control test exception rate >5% (escalate to compliance officer)
                                                                                                                  - - Form PF or Form ADV disclosure discrepancy detected (escalate to compliance counsel)
                                                                                                                   
                                                                                                                    - ## Tooling Hooks
                                                                                                                    - - **AML/KYC intake form**: Linked to investor master (00_FIN-ORCH); triggers KYC questionnaire and documentation request
                                                                                                                      - - **Sanctions screening API**: Monthly batch pull of investor/counterparty list; hit = automatic escalation + email alert
                                                                                                                        - - **Fund limit monitoring**: Daily pull of portfolio data (03_INVESTMENTS-OPS) and GL (02_FUND-ACCOUNTING); calculate ratios and flag threshold breaches
                                                                                                                          - - **Control test workpaper**: Template-based procedure execution and evidence upload; exception tracking
                                                                                                                           
                                                                                                                            - ## KPIs (Per Reporting Period)
                                                                                                                            - - **AML/KYC Compliance %**: (Complete docs / Total investors) × 100 → Target: 100%
                                                                                                                              - - **Sanctions Screening Hits**: # of matches / month → Target: 0 (or all remediated)
                                                                                                                                - - **Fund Policy Limit Breaches**: # of days limit exceeded / month → Target: 0
                                                                                                                                  - - **Control Test Completion Rate**: (Completed tests / Scheduled tests) × 100 → Target: 100%
                                                                                                                                    - - **Control Test Exception Rate**: (Exceptions found / Total tests) × 100 → Target: <5%
                                                                                                                                      - - **Form PF Filing Timeliness**: Days from deadline to filing → Target: 0 (filed on time)
