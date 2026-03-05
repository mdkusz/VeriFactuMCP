# AGENTS.md

This repository is a sample MCP service to manage invoices in AEAT VeriFactu context.

## Repo layout (do not invent paths)
- `docs/` : Official AEAT PDFs (requirements, QR, hash footprint, signatures, validations).
- `spec/xsd/` : Official XSD schemas (Suministro/Consulta/Respuestas/Eventos).
- `spec/wsdl/` : Official WSDL.
- `spec/json/` : This repo JSON contracts (client <-> MCP). Agents MUST follow these schemas.
- `examples/` : Sample XML files.
- `src/` : .NET source code.
- `tests/` : Tests.

Agents must not create new top-level directories without explicit instruction.

## Core design
### Incremental invoice building
The MCP accepts invoices incrementally:
1) Create a Draft
2) Apply Ops (add/update/remove/set/unset)
3) Finalize => Snapshot (immutable)
4) Snapshot => Generate VeriFactu XML (XSD-compliant)
5) Optionally send/consult AEAT

### Immutable Snapshot
`InvoiceSnapshot` is the canonical record:
- Persist it.
- Use it to generate VeriFactu XML.
- Return it in queries (consultation) so the client can rectify using the same shape.

Drafts can be incomplete. Snapshots must be complete enough to generate XML and hash/QR.

## JSON contracts (authoritative)
Located in `spec/json/`:
- `common-types.schema.json`
- `invoice-draft-create.schema.json`
- `invoice-draft-ops.schema.json`
- `invoice-snapshot.schema.json`
- `aeat-consulta-request.schema.json`
- `aeat-consulta-response.schema.json`

Any API payload exchanged with the client MUST validate against these schemas.

## Operations model
Draft mutation happens via `InvoiceDraftOps`:
- Supports add/remove/update/set/unset operations.
- Operations are atomic, auditable, and deterministic.
- For repeated entities (lines), use stable IDs (e.g., `lineId`).

## AEAT mapping rules
- XML generation must validate against XSD in `spec/xsd/`.
- Hash footprint and chaining must follow AEAT specification.
- Signature (XAdES enveloped) is only used when applicable (e.g., non-VeriFactu validation flow), following AEAT signature specs.
- QR information must be derived following AEAT QR spec.

## Testing expectations
At minimum:
- JSON schema validation tests.
- XML output validates against XSDs.
- Hash fixtures tests (given a snapshot, hash matches expected).
- (Optional) signature verification tests if implemented.

## Style and PR hygiene
- Keep changes minimal and consistent with current repo layout.
- Do not refactor unrelated files.
- Prefer additive commits, keep examples under `examples/` and schemas under `spec/json/`.
