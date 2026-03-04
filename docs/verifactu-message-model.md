# VeriFactu Message Model

## Scope

This document describes the logical message model used by AEAT VeriFactu services.

The structure is formally defined by XML Schema (XSD) files published by AEAT.

Main schemas:

SuministroInformacion.xsd  
SuministroLR.xsd  
ConsultaLR.xsd  
RespuestaSuministro.xsd  
RespuestaConsultaLR.xsd  
RespuestaValRegistNoVeriFactu.xsd  
EventosSIF.xsd  

These schemas define the structure of:

- submission messages
- consultation messages
- response messages

---

# Message Categories

Three message categories exist:

1. Submission of invoicing records
2. Cancellation of invoicing records
3. Query of submitted records

Each message is transported inside a SOAP envelope.

---

# General SOAP Structure

All messages follow this structure:

Envelope  
  Body  
    Request  

The request node depends on the operation.

---

# Submission Message (Registration / Cancellation)

Root message type:

SuministroLRFacturasEmitidas

Logical structure:

Cabecera  
RegistrosFacturacion

The message may contain multiple records.

Maximum records per submission:

1000

---

# Cabecera (Header)

The header identifies the sender and the context of the submission.

Typical fields:

IDVersion  
Version of the message schema.

ObligadoEmision  
Issuer of the invoice records.

ObligadoEmision.NombreRazon  
Name or company name.

ObligadoEmision.NIF  
Tax identifier.

Representante (optional)  
Representative acting on behalf of the taxpayer.

IndicadorRepresentante  
Flag indicating whether the sender acts as representative.

---

# RegistrosFacturacion

Container for invoicing records.

Each element represents an operation on an invoicing record.

Two record types exist.

RegistroAlta  
RegistroAnulacion

---

# RegistroAlta (Invoice Record Registration)

Represents the creation or correction of an invoice record.

Typical logical structure:

RegistroAlta  
  IDFactura  
  DatosFactura  
  SistemaInformatico  
  Huella

---

## IDFactura

Uniquely identifies the invoice record.

Fields:

NIF  
Issuer tax identifier.

NumSerieFactura  
Invoice number or number with series.

FechaExpedicionFactura  
Invoice issue date.

Composite identifier:

NIF + NumSerieFactura + FechaExpedicionFactura

This identifier is used for:

- duplicate detection
- corrections
- cancellations
- queries

---

## DatosFactura

Contains the main invoice data.

Typical fields:

TipoFactura  
Invoice type.

ImporteTotal  
Total invoice amount.

DescripcionOperacion  
Description of the transaction.

Contraparte  
Customer information.

---

## Contraparte

Customer or counterparty of the invoice.

Fields:

NombreRazon  
Customer name.

NIF  
Customer tax identifier.

---

## SistemaInformatico

Information about the invoicing software that generated the record.

Fields include identifiers for the software system.

This block is required for traceability of the invoicing system.

---

## Huella (Fingerprint)

Cryptographic hash used to guarantee the integrity of invoicing records.

The hash chain ensures that records cannot be altered without detection.

The hash is calculated according to the VeriFactu hash specification.

---

# RegistroAnulacion (Invoice Record Cancellation)

Represents the cancellation of a previously submitted invoicing record.

Logical structure:

RegistroAnulacion  
  IDFactura  
  MotivoAnulacion

The IDFactura block identifies the record being cancelled.

---

# Consultation Message

Consultation allows retrieving previously submitted invoicing records.

Root element:

ConsultaFactuSistemaFacturacion

Structure:

Cabecera  
FiltroConsulta  
DatosAdicionalesRespuesta (optional)

This structure is defined in the consultation schema. :contentReference[oaicite:0]{index=0}

---

# FiltroConsulta

Defines the search criteria for the query.

Typical fields:

PeriodoImputacion  
Fiscal year and period.

NumSerieFactura  
Invoice identifier.

Contraparte  
Counterparty identifier.

FechaExpedicionFactura  
Invoice issue date.

SistemaInformatico  
Software system identifier.

RefExterna  
External reference.

ClavePaginacion  
Pagination key.

---

# Response Message

AEAT returns a response message containing:

Cabecera  
ResultadoGlobal  
DetalleRegistros

The response includes both global and per-record validation results.

---

# ResultadoGlobal

Possible values:

Correct  
PartiallyCorrect  
Incorrect

These values represent the overall result of the submission.

---

# DetalleRegistros

Each submitted record has its own result.

Fields typically include:

EstadoRegistro  
CodigoError  
DescripcionError

---

# SOAP Fault Response

If the XML fails schema validation, AEAT returns a SOAP Fault instead of a normal response.

Structure:

Envelope  
  Body  
    Fault  
      faultcode  
      faultstring  
      detail

SOAP Fault means the entire submission failed and must be corrected before resending.
