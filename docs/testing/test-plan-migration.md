# ETL & Data Migration Test Plan – FOI System

## Scope
Validate the end-to-end ETL and migration pipeline from CSV ingestion to FOI Request creation, including:

- Dataflow (CSV → FOI Import Request staging)
- Alternate key and upsert behaviour
- Migration Power Automate flow
- Data Migration Run logging
- Row-level error handling

---

## Preconditions
- FOI Import Request staging table exists
- FOI Request production table exists
- Alternate key configured for idempotent imports
- Data Migration Run table enabled
- Migration Power Automate flow deployed and enabled

---

## Test Scenarios (Summary)

| ID | Scenario | Description |
|----|---------|-------------|
| ETL-01 | Happy path | End-to-end CSV → FOI Request migration |
| ETL-02 | Idempotency | Re-running same file does not create duplicates |
| ETL-03 | Incremental import | New rows inserted, existing rows updated |
| ETL-04 | Row-level failure | One invalid record handled without blocking batch |
| ETL-05 | Partial batch | Mixed valid and invalid rows |
| ETL-06 | Data integrity | Field mapping and transformation validation |
| ETL-07 | Performance sanity | Medium batch completes successfully |
| ETL-08 | Re-run safety | Re-running completed migration does not duplicate data |

---

## Detailed Scenarios

### ETL-01 — Happy Path (Baseline Functional)
**Goal**  
Confirm the full pipeline works end-to-end.

**Steps**
1. Upload a valid CSV containing:
   - 10 rows
   - At least one record with `Sensitive = Yes`
   - Multiple Requested Info Types
2. Run the Dataflow.
3. Validate FOI Import Request (staging):
   - Records exist
   - Fields populated correctly
   - Processing Status = `Pending`
4. Trigger the Migration Power Automate flow.
5. Validate FOI Request (production):
   - Same number of records created
   - Required fields populated
   - Owner assigned correctly
6. Validate FOI Import Request:
   - Processing Status = `Imported`
   - Linked FOI Request populated
   - Associated Data Migration Run populated
7. Validate Data Migration Run:
   - Status = `Completed`
   - Total Count = 10
   - Success Count = 10
   - Error Count = 0

**Expected Result**
- Clean end-to-end migration
- No duplicates
- Counters accurate
- Logs complete and readable

---

### ETL-02 — Idempotency (Re-run Without Duplicates)
**Goal**  
Verify alternate key + upsert prevent duplicates.

**Steps**
1. Re-run the same Dataflow using the same CSV.
2. Re-run the Migration flow.

**Expected Result**
- No duplicate FOI Import Request records
- No duplicate FOI Requests
- New Data Migration Run logged for the re-run

---

### ETL-04 — Row-Level Failure Handling
**Goal**  
Ensure one invalid record does not block the batch.

**Steps**
1. Prepare a CSV with:
   - 9 valid rows
   - 1 invalid row (e.g. missing Title, or maximum column's length exceeded)
2. Run the Dataflow.
3. Run the Migration flow.
4. Validate staging:
   - Invalid row Processing Status = `Error`
   - Error Message populated
5. Validate valid rows:
   - Processing Status = `Imported`
6. Validate Data Migration Run:
   - Success and error counts reflect mixed outcome
   - Status indicates partial success

**Expected Result**
- Valid records imported successfully
- Invalid record isolated and logged
- Batch completes without failure

---

## Verification Summary
- Staging loads correctly via Dataflow
- Alternate key prevents duplicate imports
- Incremental updates behave as expected
- Partial failures do not block batch processing
- Errors are logged with row-level detail
- Migration runs are repeatable and stable