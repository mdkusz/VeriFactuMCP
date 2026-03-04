# VeriFactu XML Signature

## Scope

VeriFactu submissions require the XML invoicing record to be digitally signed.

The signature ensures:

- integrity of the invoicing record
- authenticity of the sender
- non-repudiation

The signature format used by AEAT is:

XMLDSIG + XAdES

---

# Signature Standard

The XML signature uses the W3C XML Digital Signature standard.

Namespace:

http://www.w3.org/2000/09/xmldsig#

The signature node is added to the XML document:

Signature

Typical structure:

Signature
  SignedInfo
  SignatureValue
  KeyInfo
  Object

---

# XAdES Extension

VeriFactu uses XAdES properties to extend the standard XMLDSIG signature.

The XAdES profile used is typically:

XAdES-EPES

This profile includes:

- signing certificate
- policy identifier
- signing time

The properties are included in:

QualifyingProperties

---

# Signing Workflow

The signature must be generated after the XML invoicing record is created.

Workflow:

1 generate RegistroAlta XML
2 canonicalize XML
3 compute digest
4 generate signature
5 attach Signature node

The signed XML is then sent in the SOAP message.

---

# Signing Libraries

Typical libraries used:

Java

xades4j

.NET

System.Security.Cryptography.Xml
custom XAdES implementation

---

# Repository Examples

Example files:

examples/registro-alta.xml
examples/registro-alta-firmado.xml

These examples illustrate the difference between:

- unsigned record
- signed record ready for submission
