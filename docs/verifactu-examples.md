# VeriFactu XML Examples

## Scope

This directory contains example XML messages used for development and testing.

The examples illustrate real VeriFactu message structures and are intended to help developers and automated agents understand how XML submissions are constructed.

These examples are useful for:

- validating XML generation
- testing SOAP submissions
- verifying schema compliance
- understanding the practical structure of invoicing records
- understanding XML signatures used in VeriFactu

Examples are derived from official AEAT documentation and test environments.

---

# Example Files

../examples/registro-alta.xml  
../examples/registro-alta-firmado.xml  

---

# Example 1: Invoice Record Registration (Unsigned)

File:

../examples/registro-alta.xml

Operation:

RegistroAlta

This example represents a **registration of an invoicing record without XML signature**.

This structure is useful for understanding the logical structure of the invoicing record before the signature is applied.

Typical structure:

RegistroAlta
  IDVersion
  IDFactura
    IDEmisorFactura
    NumSerieFactura
    FechaExpedicionFactura
  NombreRazonEmisor
  Subsanacion
  RechazoPrevio
  TipoFactura
  TipoRectificativa
  FacturasRectificadas
  FechaOperacion
  DescripcionOperacion
  Destinatarios
  Desglose

Important elements illustrated in this example:

Rectifying invoice  
TipoRectificativa

Reference to corrected invoices  
FacturasRectificadas

VAT breakdown  
Desglose

Multiple tax rates

This example is useful for:

- understanding the XML data model
- implementing XML builders
- validating schema compliance

---

# Example 2: Signed Invoice Record

File:

examples/registro-alta-firmado.xml

Operation:

RegistroAlta

This example represents a **fully signed invoicing record ready for submission to AEAT**.

The XML contains a digital signature compliant with:

XML Digital Signature (XMLDSIG)  
XAdES (XML Advanced Electronic Signatures)

Typical additional structure:

RegistroAlta
  ...
  Signature
    SignedInfo
    SignatureValue
    KeyInfo
    Object
      QualifyingProperties

The Signature element guarantees:

- integrity of the invoicing record
- authenticity of the sender
- non-repudiation

The XAdES extension includes additional metadata such as:

- signing certificate
- signing time
- signature policy

---

# Signing Workflow

The signed example illustrates the final stage of the submission pipeline.

Typical workflow:

1 Generate RegistroAlta XML  
2 Canonicalize XML (C14N)  
3 Calculate digest  
4 Generate XML Digital Signature  
5 Add XAdES properties  
6 Attach Signature node  

After signing, the XML is sent to AEAT inside the SOAP message.

---

# Purpose of These Examples

These examples help automated tools and developers to:

Understand real XML structure  
Compare unsigned vs signed records  
Implement XML builders and signers  
Validate generated XML during development
