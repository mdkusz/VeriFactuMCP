# VeriFactu Error Codes

## Scope

This document describes the error codes returned by AEAT during validation of VeriFactu submissions.

Errors may appear in two contexts:

1. SOAP Fault (structural or header validation errors)
2. Record validation errors in a normal response

Error codes allow the client system to determine:

- whether the submission must be corrected
- whether only specific records must be resent
- whether the record was accepted with warnings

Primary reference: AEAT validation specification and error catalogue.

---

# Error Categories

Errors fall into two categories.

## Non-admissible errors

These errors cause rejection of the invoicing record.

Characteristics:

- The record is not registered by AEAT.
- The status returned is:

Incorrect

The record must be corrected and resent.

Typical causes:

- invalid NIF
- invalid invoice identifier
- invalid dates
- schema validation errors
- business rule violations

---

## Admissible errors

These errors do not prevent acceptance of the invoicing record.

Characteristics:

- The record is registered by AEAT.
- The status returned is:

AcceptedWithErrors

The system should store the error but does not need to resend the record.

Typical causes:

- minor inconsistencies
- optional data validation issues

---

# SOAP Fault Errors

SOAP Fault indicates that the submission itself is invalid.

In this case:

- the entire submission is rejected
- no record-level validation occurs

Typical causes:

- malformed XML
- schema validation errors
- invalid header data

SOAP Fault structure:

Envelope  
  Body  
    Fault  
      faultcode  
      faultstring  
      detail  

Example:

faultcode: env:Client  
faultstring: Codigo[4104]. The NIF of the holder in the header is not identified.

When SOAP Fault is returned:

- correct the request
- resend the entire submission

---

# Error Code Structure

AEAT error codes usually contain:

ErrorCode  
Numeric identifier of the validation rule.

Description  
Human-readable explanation of the validation failure.

Example:

4104  
The NIF of the holder in the header is not identified.

---

# Error Handling Strategy

A client implementation should handle errors as follows.

SOAP Fault received

Action:

- inspect faultcode
- correct request
- resend submission

Normal response received

Check per-record status:

Correct  
Record accepted.

AcceptedWithErrors  
Record accepted with warnings. Store warning.

Incorrect  
Record rejected. Correct data and resend record.

---

# Error Mapping Strategy

The implementation should normalize AEAT error codes into internal categories.

Suggested structure:

ErrorCode  
Original AEAT code.

Severity  

Possible values:

Fatal  
Rejects record.

Warning  
Record accepted with errors.

Scope  

Possible values:

Submission  
Whole message invalid.

Record  
Specific record invalid.

Description  
Original AEAT message.

---

# Example Error Mapping

Example:

ErrorCode: 4104

Severity: Fatal

Scope: Submission

Description:

The NIF of the holder in the header is not identified.

Result:

Submission rejected.
