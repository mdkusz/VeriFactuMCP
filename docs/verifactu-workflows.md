# VeriFactu Workflows (Submission, Retry, Cancellation, Query)

## Scope

This document defines the practical workflows a client system must implement to:

- submit invoice records (registration and cancellation)
- interpret AEAT responses
- retry safely
- query previously submitted records (when available)

Primary reference: AEAT "Computerised Invoicing Systems" specification. :contentReference[oaicite:0]{index=0}  
Secondary reference for query structures: AEAT consultation schema documentation. :contentReference[oaicite:1]{index=1}

---

## Workflow 1: Register an Invoicing Record (RegistroAlta)

### Goal
Submit an invoice record for registration.

### Preconditions
- The system has invoice data.
- The XML is generated according to the XSD.
- The request is signed/authenticated with a qualified client certificate (TLS client auth).

### Steps
1. Build submission XML:
   - Cabecera
   - One RegistroAlta
2. Send SOAP request to AEAT over HTTPS.
3. Receive synchronous response:
   - SOAP Fault OR normal XML response.
4. Persist:
   - outbound XML
   - inbound response XML
   - derived status per record

### Outcomes
A) SOAP Fault:
- Entire submission rejected → fix payload → resend. :contentReference[oaicite:2]{index=2}

B) Normal response:
- Global result: Correct / PartiallyCorrect / Incorrect. :contentReference[oaicite:3]{index=3}
- Per-record result: Correct / AcceptedWithErrors / Incorrect. :contentReference[oaicite:4]{index=4}

---

## Workflow 2: Retry After Transmission Failure

### Goal
Recover when no response was received or a transient server issue occurred.

### Trigger conditions
- Network timeout / connection failure
- No XML response received
- SOAP Fault with server-side indication

### Steps
1. Do not change the payload.
2. Resend the same submission (idempotent approach recommended).
3. Process the response normally.

### Notes
- AEAT describes that transmissions may need to be resent when no expected XML is received. :contentReference[oaicite:5]{index=5}

---

## Workflow 3: Handle Rejections (Record-level Incorrect)

### Goal
Resubmit only the records that were rejected, after fixing the cause.

### Trigger conditions
- Normal response received
- Some records have per-record status = Incorrect
- Global status is PartiallyCorrect or Incorrect (without SOAP Fault)

### Steps
1. Identify rejected records from response detail.
2. Apply corrections in source data (only for voluntary referral scenarios).
3. Create a new submission containing only corrected records.
4. Resend.

### Notes
- If all records are rejected due to structural issues, AEAT returns SOAP Fault and the whole submission must be corrected. :contentReference[oaicite:6]{index=6}
- If rejection is business validation, per-record error codes are returned. :contentReference[oaicite:7]{index=7}

---

## Workflow 4: Accept With Errors (AcceptedWithErrors)

### Goal
Track admissible errors (warnings) for operational and compliance reporting.

### Trigger conditions
- Normal response received
- Per-record status = AcceptedWithErrors

### Steps
1. Persist the record as "accepted".
2. Persist associated admissible error codes/messages.
3. Decide if a corrective invoice/business action is needed (outside the scope of transport protocol).

### Notes
- AEAT differentiates admissible vs non-admissible errors; admissible errors do not prevent acceptance. :contentReference[oaicite:8]{index=8}

---

## Workflow 5: Cancel an Invoicing Record (RegistroAnulacion)

### Goal
Cancel a previously submitted invoice record.

### Preconditions
- The record identity is known:
  - NIF + NumSerieFactura + FechaExpedicionFactura

### Steps
1. Build submission XML:
   - Cabecera
   - One RegistroAnulacion
2. Send SOAP request.
3. Process response:
   - SOAP Fault OR normal response with per-record result.
4. Persist outbound/inbound XML and resulting status.

### Possible outcomes
- Correct: cancellation accepted
- AcceptedWithErrors: cancellation accepted with admissible issues
- Incorrect: cancellation rejected (business rules, not found, etc.)

:contentReference[oaicite:9]{index=9}

---

## Workflow 6: Query Submitted Records (ConsultaFactuSistemaFacturacion)

### Availability
This query service is available for voluntary referral systems (VERI*FACTU) and can be performed by:
- issuer of the invoicing record
- recipient (customer) to query supplier-submitted records

:contentReference[oaicite:10]{index=10}

### Goal
Retrieve invoicing records previously submitted.

### Query dimensions (typical)
- Fiscal year and period (derived from transaction date or issue date)
- Counterparty NIF
- Invoice number/series
- Issue date range
- External reference (RefExterna)
- Software system (SIF) filters

:contentReference[oaicite:11]{index=11} :contentReference[oaicite:12]{index=12}

### Result limits
- Maximum returned records per query: 10,000
- If more exist: pagination is required

:contentReference[oaicite:13]{index=13}

---

## Workflow 7: Paginated Query

### Goal
Retrieve more than 10,000 records by performing multiple queries.

### Steps
1. Perform initial query (with year/period and optional filters).
2. If response indicates more data is pending:
   - store the last record identifier returned (submission-date ordered).
3. Perform a new query including the pagination key / last record identifier.
4. Repeat until no pending data remains.

:contentReference[oaicite:14]{index=14}

---

## Implementation Checklist (Runtime)

A runtime implementation should include:

- XML generation compliant with XSDs
- SOAP client (document/literal)
- TLS client certificate auth
- Persistent storage of:
  - request XML
  - response XML
  - derived per-record status
  - error codes/messages
- Deterministic retry policy:
  - retry on timeouts / server faults
  - do not retry blindly on client faults
- Optional query and pagination support (if enabled/available)

:contentReference[oaicite:15]{index=15}
