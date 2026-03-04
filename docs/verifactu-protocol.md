# VeriFactu Protocol (Transport, SOAP, Responses)

## Scope

This document describes the communication protocol used by AEAT VeriFactu services:

- Transport and message format
- Authentication requirements
- Response semantics (global vs per-record)
- SOAP Fault handling and retry guidance

Primary reference: AEAT "Computerised Invoicing Systems" technical specification. :contentReference[oaicite:0]{index=0}  
Secondary reference: AEAT "Servicio validación de registros no verificables (No VERI*FACTU)". :contentReference[oaicite:1]{index=1}

---

## Transport and Encoding

- Network: Internet
- Protocol: HTTPS
- Messaging: SOAP 1.1
- Style: document/literal
- Payload: XML
- Encoding: UTF-8

All messages must comply with the published XSDs. :contentReference[oaicite:2]{index=2}

---

## Authentication (Client Certificate)

Requests are authenticated using a **qualified electronic certificate** installed/available for the client making the call.

The caller may be:
- the taxpayer (issuer),
- an authorised representative,
- a social collaborator.

All NIFs are validated against AEAT’s central database. :contentReference[oaicite:3]{index=3}

---

## Message Pattern

### Request

A request is a SOAP envelope whose Body contains:
- Header (`Cabecera`)
- One or more invoicing records (registration and/or cancellation)

AEAT performs:
1) XML/XSD validation
2) business validation

Max records per submission: **1000**. :contentReference[oaicite:4]{index=4}

### Response

If the submission passes structural validation, AEAT returns an XML response with:
- a global outcome (submission-level)
- per-record outcomes (record-level), including error codes where applicable

If structural validation fails, AEAT returns a SOAP Fault and the **entire submission is rejected**. :contentReference[oaicite:5]{index=5}

---

## Outcomes and Semantics

### Global submission outcome

Possible values:
- `Correct` (fully accepted)
- `PartiallyCorrect` (partially accepted)
- `Incorrect` (fully rejected)

Meaning:
- `Correct`: all records accepted
- `PartiallyCorrect`: mix of accepted/rejected records OR accepted records with admissible errors
- `Incorrect`: all records rejected (or the header/structure is invalid and a SOAP Fault is returned)

:contentReference[oaicite:6]{index=6}

### Per-record outcome

Possible values:
- `Correct` (accepted)
- `AcceptedWithErrors` (accepted with admissible errors)
- `Incorrect` (rejected)

:contentReference[oaicite:7]{index=7}

---

## Error Categories

Two categories are used conceptually:

- **Non-admissible errors**: cause record rejection.
- **Admissible errors**: record accepted but flagged (AcceptedWithErrors).

:contentReference[oaicite:8]{index=8}

---

## SOAP Fault Handling

SOAP Fault is returned when:
- XML is not well-formed
- schema validation fails
- header-level syntactic/structural problems occur

SOAP Fault includes:
- `faultcode` (client vs server)
- `faultstring` describing the issue

Interpretation guidance:
- `env:Client`: the request is incorrect → fix and resend
- `env:Server`: transmission/server-side issue → resend (retry)

:contentReference[oaicite:9]{index=9}

---

## Retry Guidance (Implementation Rule)

Implement deterministic retry rules:

1) If HTTP/network timeout or no response:
   - retry with backoff

2) If SOAP Fault with `env:Server`:
   - retry with backoff (idempotent design recommended)

3) If SOAP Fault with `env:Client`:
   - do not retry blindly
   - fix payload, then resend

4) If normal response:
   - process per-record results
   - re-submit only rejected records if the business flow requires it (voluntary submission)

This is consistent with AEAT’s response semantics and fault guidance. :contentReference[oaicite:10]{index=10}

---

## No VERI*FACTU Validation Service (Test Utility)

AEAT provides a validation web service for **No VERI*FACTU** records that:
- validates XML format and business rules
- does NOT validate the XML signature
- does NOT store any information (validation-only)

Response values for the validated record:
- `Correcto` (Accepted)
- `AceptadoConErrores` (Accepted with errors)
- `Incorrecto` (Rejected)

:contentReference[oaicite:11]{index=11}

Note: This service is useful for automated pre-validation in development and CI pipelines.
