# Data Vault Test Cases - Hubs & Links

## 1. Test Strategy Overview

### Scope
Testing of 10 raw vault tables comprising Hub and Link entities to ensure data integrity, consistency, relationships, and system performance in a Data Vault 2.0 architecture.

### Types of Testing
- **Functionality Testing:** Data loading, relationships, key generation
- **Data Integrity Testing:** Unique constraints, referential integrity
- **Performance Testing:** Load times, query performance under volume
- **Security Testing:** Data access controls, audit trail
- **Reliability Testing:** Error handling, recovery scenarios

### Pre-requisites
- Test database environment with sample data
- Staging tables populated with source data
- Data vault ETL jobs configured and executable
- Audit and logging infrastructure enabled
- Access credentials for data vault schemas

---

## 2. Functional Test Cases

### A. Hub Table Tests

#### TC-F-001: Load Hub with Valid Business Keys
- **Type:** Positive
- **Pre-conditions:** Staging table populated with unique business keys
- **Steps:**
  1. Execute ETL process to load Hub table
  2. Verify row count matches source data
  3. Confirm all business keys inserted
  4. Check Hub Load Date and Source ID populated
- **Expected Result:** All rows loaded successfully with unique business keys and metadata populated

#### TC-F-002: Prevent Duplicate Hub Records
- **Type:** Negative
- **Pre-conditions:** Hub table contains existing record with business key "BK001"
- **Steps:**
  1. Attempt to load duplicate record with same business key
  2. Check for constraint violation
  3. Verify existing record remains unchanged
- **Expected Result:** Duplicate rejected; existing record maintained; constraint error logged

#### TC-F-003: Hub Key Generation and Uniqueness
- **Type:** Positive
- **Pre-conditions:** Multiple business keys from different sources
- **Steps:**
  1. Load business keys into Hub table
  2. Retrieve generated Hub Keys
  3. Verify each business key has unique Hub Key
  4. Confirm no NULL values in Hub Key column
- **Expected Result:** Each business key assigned unique Hub Key; no duplicates or NULLs

#### TC-F-004: Handle NULL Business Keys
- **Type:** Boundary
- **Pre-conditions:** Staging data contains NULL values in business key column
- **Steps:**
  1. Execute load process with NULL business keys
  2. Check for error handling
  3. Review error logs
- **Expected Result:** NULL values rejected; error message logged; non-NULL records loaded successfully

#### TC-F-005: Multi-Source Hub Load
- **Type:** Positive
- **Pre-conditions:** 5 different source systems configured
- **Steps:**
  1. Load records from each source system
  2. Verify Source ID distinct for each source
  3. Check Load Date reflects each load cycle
- **Expected Result:** All records loaded with correct Source ID and Load Date timestamp

#### TC-F-006: Hub Load Idempotency
- **Type:** Positive
- **Pre-conditions:** Hub table has been populated
- **Steps:**
  1. Run ETL process first time
  2. Load 50 records
  3. Run same ETL process again
  4. Verify record count remains 50
- **Expected Result:** Idempotent behavior; no duplicate records created on re-run

---

### B. Link Table Tests

#### TC-F-007: Create Valid Link Between Hubs
- **Type:** Positive
- **Pre-conditions:** Parent Hub tables (Customer, Order) have loaded records
- **Steps:**
  1. Load Link table with valid Hub Key references
  2. Verify Link Key generated
  3. Confirm Hub Key foreign keys populated
  4. Check Load Date and Source ID assigned
- **Expected Result:** Link successfully created with valid relationships and metadata

#### TC-F-008: Enforce Referential Integrity - Invalid Hub Key
- **Type:** Negative
- **Pre-conditions:** Link table FK constraints enabled
- **Steps:**
  1. Attempt to insert Link with non-existent Hub Key reference
  2. Check for constraint violation
  3. Review error message
- **Expected Result:** Insert rejected; referential integrity error logged; transaction rolled back

#### TC-F-009: Link with Multiple Hub Keys
- **Type:** Positive
- **Pre-conditions:** Order-Product Link references both Order Hub and Product Hub
- **Steps:**
  1. Load Link with 2+ Hub Key foreign keys
  2. Verify all Hub Key relationships established
  3. Confirm Link Key uniqueness across combination
- **Expected Result:** Link created with valid composite relationships

#### TC-F-010: Link Effective Dating
- **Type:** Positive
- **Pre-conditions:** Link Load Date tracking enabled
- **Steps:**
  1. Load initial Link record with Load Date = "2026-01-15"
  2. Load same Link relationship with Load Date = "2026-02-01"
  3. Verify both records exist with different Load Dates
- **Expected Result:** Both records maintained; historical tracking preserved

#### TC-F-011: Detect Orphaned Hub References in Link
- **Type:** Edge Case
- **Pre-conditions:** Hub record deleted; Link still references deleted Hub Key
- **Steps:**
  1. Identify Links referencing non-existent Hub Keys
  2. Run integrity check query
  3. Log orphaned relationships
- **Expected Result:** Orphaned records detected and reported

#### TC-F-012: Link Idempotency with Same Timestamp
- **Type:** Positive
- **Pre-conditions:** Link table populated with records
- **Steps:**
  1. Load Link with Customer Hub Key = "HUB001", Order Hub Key = "HUB002", Load Date = "2026-03-01"
  2. Reload same link with identical values
  3. Verify record count unchanged
- **Expected Result:** No duplicate Link created; idempotent load confirmed

#### TC-F-013: Link with NULL Hub Key
- **Type:** Boundary
- **Pre-conditions:** Link staging data with NULL Hub Key value
- **Steps:**
  1. Attempt to load Link with NULL Hub Key
  2. Check constraint validation
- **Expected Result:** NULL values rejected; error logged; transaction rolled back

---

### C. Cross-Hub/Link Tests

#### TC-F-014: Hub-to-Link Consistency Check
- **Type:** Positive
- **Pre-conditions:** 3 Hub tables (A, B, C) and 2 Link tables (AB, BC) loaded
- **Steps:**
  1. Query all Hub Keys referenced in Links
  2. Verify each Link references valid Hub records
  3. Count distinct Hub Keys in each Link
- **Expected Result:** All Link-referenced Hub Keys exist; no orphaned references

#### TC-F-015: Complex Hub Relationship via Multiple Links
- **Type:** Positive
- **Pre-conditions:** Customer Hub, Product Hub, Order Hub with Links connecting them
- **Steps:**
  1. Query Customer → Order → Product path
  2. Verify chain of relationships intact
  3. Confirm all intermediate Links valid
- **Expected Result:** Multi-level relationship path resolves successfully; no broken links

#### TC-F-016: Circular Reference Prevention
- **Type:** Negative
- **Pre-conditions:** Attempt to create circular Link references (A→B→A)
- **Steps:**
  1. Create Link A-B
  2. Attempt to create Link B-A with circular intent
  3. Check for logic violation
- **Expected Result:** Business logic prevents circular references; appropriate error message

---

## 3. Non-Functional Test Cases

### A. Performance Tests

#### TC-NF-001: Hub Table Load Performance Under Volume
- **Category:** Performance
- **Description:** Load 1,000,000 records into Hub table and measure performance
- **Steps:**
  1. Prepare staging table with 1M rows
  2. Execute ETL load process
  3. Measure elapsed time
  4. Monitor CPU and memory usage
  5. Check disk I/O patterns
- **Success Criteria:** Load completes in < 5 minutes; CPU < 80%; Memory < 8GB

#### TC-NF-002: Link Query Performance with Index
- **Category:** Performance
- **Description:** Query Link table with 10M rows on Hub Key index
- **Steps:**
  1. Execute query: SELECT * FROM Link WHERE Hub_Key_1 = 'X'
  2. Capture execution plan
  3. Measure query time
- **Success Criteria:** Query executes in < 2 seconds; uses index seek (not scan)

#### TC-NF-003: Hub Lookup Performance Consistency
- **Category:** Performance
- **Description:** Retrieve Hub record by business key
- **Steps:**
  1. Execute lookup query 1000 times
  2. Measure average response time
  3. Check for performance degradation
- **Success Criteria:** Average < 100ms; no performance degradation over time

#### TC-NF-004: Concurrent Load on Hub and Link
- **Category:** Performance
- **Description:** Simulate concurrent ETL loads on Hub and Link
- **Steps:**
  1. Start Hub load process
  2. Start Link load process simultaneously
  3. Monitor resource contention
  4. Verify data consistency post-load
- **Success Criteria:** Both processes complete without lock timeout; data integrity maintained

#### TC-NF-005: Bulk Update Performance on Link
- **Category:** Performance
- **Description:** Update 100K Link records (e.g., hash value recalculation)
- **Steps:**
  1. Execute bulk update on Link table
  2. Measure elapsed time
  3. Verify no blocking on dependent queries
- **Success Criteria:** Bulk update completes in < 2 minutes; non-blocking for reads

---

### B. Security Tests

#### TC-NF-006: Hub Data Access Control - Role-Based
- **Category:** Security
- **Description:** Verify only authorized roles can access Hub data
- **Steps:**
  1. Create test users with different roles (Analyst, Developer, ETL Service)
  2. Attempt data access with each role
  3. Verify grants/denials align with role
- **Success Criteria:** Unauthorized roles cannot select/insert; audit log records all attempts

#### TC-NF-007: Sensitive Data in Hub - Encryption at Rest
- **Category:** Security
- **Description:** Verify sensitive Hub data (SSN, email) encrypted in database
- **Steps:**
  1. Query system catalogs for encryption status
  2. Verify Transparent Data Encryption (TDE) enabled
  3. Check backup encryption settings
- **Success Criteria:** All sensitive columns encrypted; encryption keys securely managed

#### TC-NF-008: Audit Trail on Hub Insert/Update
- **Category:** Security
- **Description:** Verify all Hub modifications captured in audit log
- **Steps:**
  1. Insert new Hub record
  2. Query audit table
  3. Verify Insert timestamp, User ID, Source ID logged
- **Success Criteria:** Every DML operation captured with user, timestamp, source; no gaps

#### TC-NF-009: SQL Injection Prevention in ETL Load
- **Category:** Security
- **Description:** Test Link ETL process against SQL injection attempts
- **Steps:**
  1. Pass business key containing SQL injection payload (e.g., "'; DROP TABLE--")
  2. Monitor for injection execution
  3. Check parameter binding
- **Success Criteria:** Payload treated as literal string; no SQL executed; validation logged

#### TC-NF-010: Link Data Masking for Non-Privileged Users
- **Category:** Security
- **Description:** Verify Link table implements column-level security masking
- **Steps:**
  1. Query Link table as privileged user
  2. Query same Link as non-privileged user
  3. Compare results
- **Success Criteria:** Non-privileged users see masked/redacted sensitive FK values

---

### C. Reliability & Consistency Tests

#### TC-NF-011: Hub Recovery After Load Failure
- **Category:** Reliability
- **Description:** Verify Hub table consistency after mid-load failure
- **Steps:**
  1. Start Hub load of 100K records
  2. Simulate failure (network interruption) at 50% completion
  3. Inspect Hub table state
  4. Verify transaction rolled back
- **Success Criteria:** Either all 100K loaded or 0 loaded; no partial/corrupt data

#### TC-NF-012: Link Referential Integrity Check Across Database
- **Category:** Reliability
- **Description:** Run comprehensive referential integrity validation
- **Steps:**
  1. Execute integrity check: FOR EACH Link, verify referenced Hub Keys exist
  2. Identify orphaned records
  3. Generate reconciliation report
- **Success Criteria:** 100% of Link Hub Key references valid; 0 orphaned records

#### TC-NF-013: Hub/Link Data Consistency After Rollback
- **Category:** Reliability
- **Description:** Verify consistency maintained if Link load rolls back while Hub committed
- **Steps:**
  1. Load Hub successfully
  2. Start Link load
  3. Force rollback mid-transaction
  4. Verify Hub unchanged; Link unchanged
- **Success Criteria:** Atomicity maintained; no partial updates across tables

#### TC-NF-014: Backup Restore Integrity
- **Category:** Reliability
- **Description:** Restore Hub/Link from backup and verify consistency
- **Steps:**
  1. Take backup of vault tables
  2. Corrupt test data in live tables
  3. Restore from backup
  4. Verify Hub/Link integrity post-restore
- **Success Criteria:** Restored data matches pre-corruption state; relationships intact

---

### D. Scalability Tests

#### TC-NF-015: Hub Scalability - 100M+ Rows
- **Category:** Scalability
- **Description:** Evaluate Hub performance with enterprise-scale data volume
- **Steps:**
  1. Load 100M+ records into Hub
  2. Run standard queries (lookup by business key)
  3. Measure query time and resource usage
- **Success Criteria:** Query < 3 seconds; no memory exhaustion; index effective

#### TC-NF-016: Link Scalability - Multi-Hub Cross-References
- **Category:** Scalability
- **Description:** Test Link scalability with complex relationships across 10+ Hubs
- **Steps:**
  1. Create Links between all Hub combinations (C(10,2) = 45 potential Links)
  2. Load large dataset across all Links
  3. Execute complex multi-join queries
- **Success Criteria:** Queries complete in < 5 seconds; execution plans optimal

---

### E. Usability & Documentation Tests

#### TC-NF-017: Schema Documentation Completeness
- **Category:** Usability
- **Description:** Verify Hub/Link schema properly documented
- **Steps:**
  1. Review data dictionary for each table
  2. Verify column definitions, data types, constraints documented
  3. Check relationships documented
- **Success Criteria:** Every table/column documented; purpose and constraints clear

#### TC-NF-018: ETL Job Logging and Error Messages
- **Category:** Usability
- **Description:** Verify Hub/Link load logs are clear and actionable
- **Steps:**
  1. Review ETL job logs from successful load
  2. Trigger error scenario and capture logs
  3. Evaluate log message clarity
- **Success Criteria:** Logs contain sufficient detail to diagnose issues; timestamps accurate

---

## Summary

**Total Test Cases:** 46
- **Functional Tests:** 16 (Hubs: 6, Links: 7, Cross-table: 3)
- **Non-Functional Tests:** 30 (Performance: 5, Security: 5, Reliability: 4, Scalability: 2, Usability: 2)

**Recommended Execution Order:**
1. Functional tests (validate correctness)
2. Non-Functional tests (validate quality attributes)
3. Integration tests (validate end-to-end data vault behavior)
