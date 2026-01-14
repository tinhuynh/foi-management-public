# Integration Test Plan – FOI System

## Scope
Validate asynchronous integrations and document handling across:
- Email intake → Dataverse FOI creation (Logic Apps)
- Webhook → Azure Function → Service Bus → Logic App (Jurisdiction validation and Downstream processing)
- Document handling → SharePoint (Canvas app upload + Model-driven initiated copy via Power Automate)
- Centralised observability via Integration Log (success/failure visibility)

## Preconditions
- DEV/TEST environment configured (env vars / connections set)
- SharePoint target site/library reachable
- Service Bus queue/topic accessible (connection healthy)
- Logic Apps enabled (Email intake and FOI event consumer)
- Azure Function endpoint reachable and configured (webhook calls succeed)
- Integration Log table exists and is writable by relevant flows/functions

---

## Test Scenarios (Summary)

| ID | Category | Description |
|----|---------|-------------|
| INT-01 | Trigger handling | Each trigger source starts processing correctly (email, webhook, app/flow) |
| INT-02 | Async behaviour | Core FOI operations are not blocked by integration latency |
| INT-03 | External dependency failure | SharePoint/service endpoint failure handled gracefully with logging |
| INT-04 | Idempotency & replay safety | Replayed triggers do not create duplicate records/files/messages |
| INT-05 | Observability & diagnostics | Failures are persisted to Integration Log with enough context to debug |
| INT-06 | Downstream processing | Successful validation publishes to Service Bus and downstream Logic App consumes |
| INT-07 | Document lifecycle | Upload/copy produces correct folder, file placement and retains metadata |

---

## Detailed Scenarios

### INT-01 — Trigger handling
**Goal**  
Confirm each supported trigger starts the integration pipeline correctly.

**Steps**
1. Send a valid FOI intake email (with and without attachment).
2. Trigger jurisdiction validation via webhook (valid postcode).
3. Upload a file via Canvas app.
4. Trigger the Model-driven initiated “copy/upload to SharePoint” Power Automate flow.

**Expected Result**
- Each trigger starts its respective integration path successfully.
- No manual intervention required to begin processing.
- Any failures are captured to Integration Log (see INT-05).

---

### INT-03 — External dependency failure (SharePoint unavailable)
**Goal**  
Validate graceful failure when SharePoint is unavailable.

**Steps**
1. Temporarily change the SharePoint site URL (environment variable) to an invalid value.
2. Execute the integration via one of the following methods:
   - Canvas upload
   - Model-driven app upload
   - Email intake
3. Confirm that:
   - No FOI record or folder is created.
   - A structured Integration Log entry is generated with Integration Status = Failed.
4. Restore the correct SharePoint URL in environment variables.
5. Manually retry the operation.
6. Confirm that the retry succeeds and the FOI record + folder are created normally.

**Expected Result**
- Operation fails without corrupting Dataverse state.
- Integration Log entry created with:
  - Operation name
  - Source
  - Failure status
  - Error message
  - Correlation Id
  - Created On (Timestamp)
  - Related FOI request
-  After restoring the correct configuration, the retry succeeds.  

---

### INT-04 — Replay / Duplicate Trigger Scenario (Documented Limitation)
**Test Objective**  
Acknowledge the behaviour when the same integration trigger is invoked more than once for the same FOI context. This is a known hardening area for enterprise environments.

**Current Behaviour**  
Due to the design of this demo solution, each email intake and FOI Request creation generates a new record and GUID. As a result, the current implementation does not include automatic deduplication or suppression of repeated triggers for the same FOI Request ID.

**Outcome**  
Replay of the same upstream signal would result in duplicate processing. This does not corrupt the system, but it does not currently prevent repeated executions.

**Future Hardening (Enterprise Approach)**  
In a production environment, idempotency controls could be implemented through:
- Idempotency keys based on message ID or FOI Request ID  
- Duplicate detection via staging table hash comparison  
- Middleware suppression at the Service Bus / Logic App layer  
- Correlation-based event locking  

**Expected Result (for this demo)**  
Duplicate prevention is not implemented. The behaviour is documented as a known limitation and a future enhancement area.

---

### INT-05 — Observability & diagnostics (Integration Log)
**Goal**  
Prove failures are diagnosable without relying on run history or inboxes.

**Steps**
1. Force one predictable failure in each area:
   - Email intake (e.g., invalid/missing required value in payload mapping)
   - Webhook validation (e.g., postcode missing / invalid)
   - SharePoint operation (see INT-03)
2. Check Integration Log entries for each failure.

**Expected Result**
- Each failure produces an Integration Log record with:
  - Operation name
  - Source of error
  - Status = Failed
  - Summary (business-level)
  - Technical detail
  - Correlation Id
  - Created On (Timestamp)
  - Related FOI request
- Logs are visible via the Model-driven app view.

---

### INT-06 — Downstream processing (Service Bus consumption)
**Goal**  
Confirm successful validation publishes a message and downstream Logic App consumes it.

**Steps**
1. Trigger jurisdiction validation with a postcode that should pass.
2. Confirm Azure Function returns success outcome.
3. Verify a message is published to Service Bus.
4. Verify the FOI event consumer Logic App processes the message.

**Expected Result**
- Service Bus message is created for successful validation.
- Downstream consumer picks it up and completes expected actions.
- Any failure at any stage is logged (Integration Log and platform diagnostics).

---

### INT-07 — Document lifecycle (placement + metadata)
**Goal**  
Validate correct SharePoint folder/file placement and key metadata outcomes.

**Steps**
1. Create FOI Request with known request number / identifier.
2. Upload a document (Canvas app) and trigger copy (Model-driven) for another document.
3. Inspect SharePoint:
   - folder naming convention
   - correct library/site target
   - file name + extension preserved
4. Confirm Dataverse references (file's relative path).

**Expected Result**
- Files land in the correct folder/library/site for the environment.
- File naming is preserved and predictable.
- Dataverse record references are correct.
- Errors are logged when file write fails.

---

## Verification Summary
- Triggers initiate processing reliably
- Integrations remain asynchronous and non-blocking
- Dependency failures fail safely and are logged
- Replays do not create unintended duplicates
- Service Bus downstream processing works end-to-end
- SharePoint document placement is correct and repeatable