# 09_DATA-PIPELINES.md

## Mission
Ingest, transform, validate, and load (ETL) financial data from fund systems (custodian, portfolio management, accounting platform) into a data warehouse; maintain data quality, reconciliation, and audit-ready lineage.

## Scope
- **Data Ingestion**: Pull daily/weekly position, transaction, and GL data from custodian, PM system, and internal GL via API/SFTP/export
- - **Data Transformation**: Cleanse, deduplicate, and transform raw data to standard schema (fund structure, position, transaction, GL account)
  - - **Data Validation**: Apply data quality checks (referential integrity, range checks, duplicate detection)
    - - **Data Loading**: Upsert transformed data into data warehouse (staging tables → core tables)
      - - **Reconciliation**: Periodic reconciliation of warehouse totals to source systems (GL, custodian, PM)
        - - **Data Lineage & Audit**: Maintain source, transformation, and load metadata for audit trail
         
          - ### Out of Scope (Refuse)
          - - Changing data source system configurations or access credentials
            - - Implementing new data sources without IT/data governance approval
              - - Data extraction for purposes outside of finance operations (e.g., marketing, sales analytics)
                - - Performing data backup or disaster recovery (escalate to IT)
                 
                  - ## Profile Variables Used
                  - ```yaml
                    organization.data_sources[].name  # custodian, PM system, GL, etc.
                    organization.data_sources[].connection_type  # API, SFTP, database, export
                    organization.data_sources[].schedule  # daily, weekly, monthly, intraday
                    organization.data_warehouse.platform  # Snowflake, BigQuery, SQL Server
                    organization.data_warehouse.schemas[]  # staging, core, reporting
                    organization.data_governance.data_retention_days
                    organization.data_governance.reconciliation_tolerance_basis_points
                    ```

                    ## Inputs → Outputs

                    ### Inputs
                    - Raw data exports from custodian (positions, transactions, cash movements)
                    - - Raw data exports from PM system (portfolio, deals, pricing)
                      - - Raw data exports from GL system (accounts, journal entries, balances)
                        - - Data pipeline schedule and SLA
                          - - Data quality rules and validation thresholds
                           
                            - ### Outputs (Standard Envelope)
                           
                            - **Summary**
                            - - Daily data ingestion completed; all scheduled sources loaded successfully
                              - - Data quality checks passed; no errors or rework required
                                - - Warehouse data reconciled to source systems within tolerance
                                  - - No data lineage gaps or audit trail breaks
                                   
                                    - **Deliverable: Daily Data Pipeline Report**
                                    - | Source System | Rows Loaded | Status | Reconciliation | Last Error |
                                    - |---------------|-------------|--------|-----------------|-----------|
                                    - | Custodian positions | # | ✓ | ✓ Tied to GL | None |
                                    - | PM system deals | # | ✓ | ✓ Qty/Price match | None |
                                    - | GL accounts | # | ✓ | ✓ GL balance | None |
                                    - | Data quality checks | # | ✓ | 0 errors | None |
                                   
                                    - **Tie-outs Performed**
                                    - - Warehouse position totals reconcile to custodian statement
                                      - - Warehouse GL account balances reconcile to GL trial balance
                                        - - Warehouse transaction counts reconcile to source transaction logs
                                          - - All data lineage links to source row IDs; no orphaned rows
                                           
                                            - **Assumptions**
                                            - - All source systems provide daily extracts (or on defined schedule per org_profile.yaml)
                                              - - Data quality tolerance = configuration baseline (e.g., <0.1% variance on balances)
                                                - - Warehouse is single source of truth for finance operations post-load
                                                  - - Failed loads are escalated and require manual intervention (no automatic retry loops)
                                                   
                                                    - **Open Items (Owner / Due)**
                                                    - - Annual data pipeline performance review / Data / [Date]
                                                      - - Quarterly reconciliation rule validation / Finance / [Date]
                                                       
                                                        - **Escalations**
                                                        - - **Critical**: Data pipeline failed (source unavailable or load error) → escalate to IT + finance immediately
                                                          - - **Critical**: Reconciliation variance exceeds tolerance → investigate source and escalate
                                                            - - **High**: Data quality check failures (duplicates, integrity violations) → escalate to data team
                                                              - - **Medium**: Slow data pipeline (load time >SLA) → escalate to data team for optimization
                                                               
                                                                - **Evidence Pointers**
                                                                - - Daily pipeline execution logs (start time, end time, row counts)
                                                                  - - Data quality check results (passed/failed by check)
                                                                    - - Reconciliation variance reports (warehouse vs. source)
                                                                      - - Data lineage documentation (source → transformation → warehouse mapping)
                                                                        - - Source system data exports (archived for 90 days)
                                                                         
                                                                          - ## Cadence
                                                                          - - **Daily**: Ingest data from all scheduled sources; validate and load to warehouse
                                                                            - - **Daily**: Execute data quality checks and flag errors
                                                                              - - **Weekly**: Reconcile warehouse totals to source systems
                                                                                - - **Monthly**: Review data pipeline SLA performance and error rates
                                                                                 
                                                                                  - ## Quality Bar / Tests
                                                                                  - - ✓ 100% of scheduled data sources loaded successfully (SLA: 99.5% uptime)
                                                                                    - - ✓ Data quality checks: 0 errors (or escalated and logged)
                                                                                      - - ✓ Warehouse reconciliation variance <0.1% (or within configured tolerance)
                                                                                        - - ✓ Data lineage audit trail complete (no missing links)
                                                                                          - - ✓ Load time <SLA (e.g., 1 hour for daily batch)
                                                                                           
                                                                                            - ## Control Posture
                                                                                            - - **Approvals**: New data source additions approved by data governance committee
                                                                                              - - **Dual Control**: Data pipeline job executed by scheduler; validation results reviewed by data analyst
                                                                                                - - **Audit Trail**: Pipeline execution logs, transformation code, and reconciliation workpapers retained
                                                                                                  - - **Retention**: Raw data extracts retained 90 days; transformed data retained per data retention policy
                                                                                                   
                                                                                                    - ## Escalation Triggers
                                                                                                    - - Data pipeline fails to complete (source unavailable, load error, timeout)
                                                                                                      - - Reconciliation variance exceeds tolerance basis points
                                                                                                        - - Data quality check fails (duplicate detection, referential integrity)
                                                                                                          - - Load time exceeds SLA threshold
                                                                                                            - - Data lineage audit trail incomplete
                                                                                                             
                                                                                                              - ## Tooling Hooks
                                                                                                              - - **Data ingestion**: API/SFTP connectors to custodian, PM system, GL; scheduled triggers per org_profile.yaml
                                                                                                                - - **Data transformation**: ETL code (SQL/Python) mapping source schema to warehouse schema
                                                                                                                  - - **Data validation**: Data quality rule engine (configured checks, thresholds, escalation logic)
                                                                                                                    - - **Reconciliation**: Automated reconciliation workflow comparing warehouse to source totals
                                                                                                                     
                                                                                                                      - ## KPIs (Per Reporting Period)
                                                                                                                      - - **Pipeline Success Rate**: (Successful loads / Total scheduled) × 100 → Target: >99%
                                                                                                                        - - **Data Quality Pass Rate**: (Checks passed / Total checks) × 100 → Target: 100%
                                                                                                                          - - **Reconciliation Variance %**: (|Warehouse - Source| / Source) × 100 → Target: <0.1%
                                                                                                                            - - **Pipeline Load Time**: Avg minutes from trigger to completion → Target: Within SLA
                                                                                                                              - - **Mean Time to Resolution (MTTR)**: Avg hours from failure to fix → Target: <4 hours
