# AI_PROMPT_MASTER — VeriFactu MCP Service (Architecture-First, Classic .NET)

You are an automated coding agent. Your job is to design and implement a classic, maintainable .NET 8 solution from scratch, in a PUBLIC GitHub repository, for a demo project.

The goal is NOT to build a “toy bot”. The goal is to assess whether AI can produce a **classic architecture** (clean separation of concerns) with **deterministic** behavior (no AI at runtime).

## 0) Non-negotiable rules

- Output MUST be a complete working repository state (code + scripts + docs) that compiles and passes tests.
- All code comments MUST be in English.
- Do NOT use any ORM or database framework:
  - NO Entity Framework
  - NO Dapper
  - NO micro-ORMs
  - Only ADO.NET base classes: `DbConnection`, `DbCommand`, `DbParameter`, `DbDataReader`, `DbProviderFactory`.
- Use Dependency Injection (Microsoft.Extensions.DependencyInjection). No Service Locator.
- Do NOT introduce unnecessary frameworks (CQRS/MediatR, generic repositories, code-gen beyond XSD).
- Do NOT invent tax/legal rules that are not strictly required for the MVP described below.
- Do NOT attempt to connect to AEAT test/prod endpoints in automated tests. Provide a send function as a stub with clear TODOs.

## 1) Repository constraints

The repository already contains these folders:

/src
/tests
/spec
/docs
/db

Under `/spec`, there will be AEAT XSD and PDF documentation, plus JSON samples. Your code must reference these paths.

## 2) MVP scope

### 2.1 What the service does
- The service acts as an **intranet MCP server**.
- It provides a **builder-style workflow**:
  1) start an invoice draft (F1 or R1)
  2) set header and party references (surrogates)
  3) add/update/remove items
  4) add rectified references for R1
  5) generate VeriFactu XML using **classes generated from AEAT XSD**
  6) validate XML against XSD
  7) record an immutable “facturation record” (RF) plus an append-only hash-chain ledger entry
  8) provide a “send to AEAT” function (implemented but NOT integration-tested)

### 2.2 Invoice types supported
- F1 (standard invoice)
- R1 (basic rectifying invoice) with minimal references to the original invoice(s)

No other types required.

### 2.3 Input data model
The builder commands use **surrogate references** (no personal data over the API by default):
- `sellerRef`
- `buyerRef`
- optional `productRef`

Master data is resolved by the service from a database using ADO.NET. For demo, provide a minimal schema and seed script.

Also provide an alternative “inline party data” path ONLY for tests/dev convenience (explicitly labeled as not recommended). The default/primary path is surrogates.

## 3) Architectural target (classic / clean)

Implement a classic layered architecture:

- **Domain**: invoice draft aggregate, value objects, minimal invariants. No DB, no XML.
- **Application**: use-cases (services) orchestrating repositories, XML generation, validation, ledger recording.
- **Infrastructure**:
  - Database (ADO.NET repositories, SQL scripts)
  - XSD-generated classes + XML serialization utilities
  - XSD validation service
  - Hash-chain ledger (append-only)
  - AEAT sender stub (interfaces + basic HTTP/SOAP client skeleton as appropriate)
- **Host**:
  - MCP server exposing tools

Keep it simple and readable.

## 4) Required solution structure

Create a .NET 8 solution in `/src`:

/src/VeriFactu.sln

Projects (names are mandatory):

- /src/VeriFactu.Domain
- /src/VeriFactu.Application
- /src/VeriFactu.Infrastructure.Database
- /src/VeriFactu.Infrastructure.VeriFactuXml
- /src/VeriFactu.Infrastructure.Validation
- /src/VeriFactu.Infrastructure.Ledger
- /src/VeriFactu.Host.Mcp

Tests:

- /tests/VeriFactu.Tests

Docs (write minimal but real docs):

- /docs/architecture.md
- /docs/decisions.md
- /docs/mapping.md

Database scripts:

- /db/schema.sql
- /db/seed.sql

## 5) Database requirements (ADO.NET only)

### 5.1 Provider-agnostic DI
Implement `IDbConnectionFactory` using `DbProviderFactories` so the service can use any provider by configuration:

- `Database:ProviderInvariantName`
- `ConnectionStrings:Default`

No provider-specific connection types in the code.

### 5.2 Minimal tables
Implement minimal tables to support:
- master data (parties, optional products)
- invoice drafts (mutable)
- RF records (immutable append-only)
- ledger entries (append-only hash chain)
- optional event log (append-only)

Use GUID strings (`CHAR(36)` style) as primary keys to avoid autoincrement differences.

Keep SQL as portable as reasonably possible. Avoid engine-specific features.

### 5.3 Append-only enforcement
In code: never execute UPDATE/DELETE on RF and ledger tables.
Optionally provide SQL triggers to block UPDATE/DELETE (keep triggers simple and optional).

## 6) XSD classes + XML generation

### 6.1 XSD location
XSD files will be placed under:

/spec/aeat/xsd

### 6.2 Code generation
Generate C# classes from AEAT XSD and store the generated code under:

/src/VeriFactu.Infrastructure.VeriFactuXml/Generated

Do not manually edit generated files. If customizations are needed, use partial classes in a separate folder.

### 6.3 XML serialization
Implement XML generation using `XmlSerializer` on the generated classes.
Correct namespaces and root element must be produced so that XSD validation passes.

## 7) XSD validation

Implement `IXsdValidator` that validates generated XML against the XSD set from `/spec/aeat/xsd`.
Return structured validation errors (line/position/message).

## 8) Hash chain ledger (internal “blockchain”)

For each RF creation:
- compute `payloadHash` over the XML bytes (UTF-8, normalized line endings)
- lookup last `chainHash` for the (sellerRef / SIF) scope (keep scope simple for MVP)
- compute new `chainHash = SHA256(prevHash + payloadHash + createdAtUtc + rfId)`
- insert ledger entry append-only

Document this in `/docs/decisions.md`.

## 9) MCP tools (builder workflow)

Expose the following MCP tools from the MCP host (names are mandatory):

- `invoice_start`
- `invoice_set_header`
- `invoice_set_seller_ref`
- `invoice_set_buyer_ref`
- `invoice_add_item`
- `invoice_update_item`
- `invoice_remove_item`
- `invoice_add_rectified_reference` (R1 only)
- `invoice_get`
- `invoice_validate`
- `invoice_generate_verifactu_xml` (must create RF + ledger entry)
- `verifactu_validate_xml`
- `aeat_send_verifactu_xml` (implemented but no integration tests)

Tool inputs/outputs must be JSON serializable and documented in `/docs/architecture.md`.

## 10) Testing requirements

- Unit tests MUST run with `dotnet test`.
- Provide at least:
  1) F1 sample JSON → generate XML → XSD validate OK
  2) R1 sample JSON → generate XML → XSD validate OK
  3) Ledger chain increments correctly (prevHash chaining)
- Tests must use sample JSON files from `/spec/samples`.
- For database in tests: use a lightweight provider approach:
  - Prefer SQLite if available OR a file-backed stub repository for tests ONLY.
  - Do NOT introduce an ORM.
  - If you use SQLite for tests, keep it strictly as ADO.NET + provider factory.
Document the test DB approach in `/docs/decisions.md`.

## 11) Deliverables and completion criteria

You are done only when:
- The solution builds.
- All tests pass.
- `VeriFactu.Host.Mcp` can run and exposes the required tools.
- XML generated from the provided samples validates against AEAT XSD.
- The repository contains docs and SQL scripts as specified.

## 12) Execution plan

Work iteratively:
1) Create solution and projects.
2) Define Domain + Application interfaces.
3) Implement DB connection factory and minimal repositories.
4) Implement draft builder use-cases.
5) Generate XSD classes and implement serializer.
6) Implement XSD validator.
7) Implement RF recording + hash chain ledger.
8) Implement MCP host + tool wiring.
9) Implement tests and ensure green.
10) Write docs.

Do not skip docs. Keep them concise and factual.

END.
