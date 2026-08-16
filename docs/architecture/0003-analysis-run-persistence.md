# ADR 0003: Persist immutable analysis runs in PostgreSQL

## Status

Accepted for the production-oriented product. No database schema, ORM, repository implementation, migration tooling, or storage infrastructure exists yet.

## Problem

OncoSyn needs to make a result reproducible and reviewable after it has been generated. An in-memory-only application loses the input, predictor versions, assumptions, chosen portfolio, and provenance that constitute the escape certificate. A startup product also needs concurrent workspace access, managed backups, and transactional integrity.

## Constraints

- Customer/patient-derived data remains disabled until security and governance controls are implemented and verified.
- Analysis runs are immutable snapshots; a revised input or configuration is a new run.
- The optimizer and domain rules cannot become coupled to storage models or SQL.
- Raw sequencing pipelines, secrets, and access-controlled data are out of scope until separately approved.
- Schema changes must be reversible or explicitly approved as irreversible.

## Selected design

Use managed PostgreSQL as the system of record behind persistence/repository adapters and an explicit migration mechanism. Persist in PostgreSQL:

- analysis-run identity, creation metadata, and schema version;
- normalized input snapshot and its content identity;
- PyClone-VI provider identity/settings, normalized inference result, and its uncertainty-bearing mutation-to-clone distribution;
- generated candidate evidence and provenance;
- clonal evidence and declared escape scenarios;
- portfolio result, solver status, and algorithm versions; and
- the completed escape certificate.

The domain service creates immutable records through the adapter. The API/delivery layer reads a certificate through the same boundary. The database model is not the domain model.

## Alternatives rejected

- **No persistence:** rejected because it prevents auditability and repeatable demonstration of the core product value.
- **SQLite as the production system of record:** rejected because it does not meet the intended concurrent, managed, backup, and operational requirements. It is allowed only for isolated local tests.
- **Persist raw VCF/BAM or patient data:** rejected because the MVP does not have the required governance or security controls.
- **Store mutable “latest result” rows:** rejected because it destroys the link between a result and the exact evidence/configuration that produced it.

## Intended data flow

```text
validated molecular input -> immutable run snapshot -> PostgreSQL
  -> PyClone-VI inference records -> candidate/clone/scenario records -> optimizer result
  -> immutable escape certificate -> PostgreSQL -> API/UI read model
```

## Error and security behavior

- Fail the analysis before persistence when input validation fails.
- Make incomplete runs observable with a safe status/error category; do not publish a partial certificate as a successful result.
- Enforce foreign-key integrity and transaction boundaries once the schema exists.
- Enforce tenant/workspace ownership, foreign-key integrity, transaction boundaries, and least-privilege database roles once the schema exists.
- Keep databases private, encrypted in transit and at rest, excluded from version control, and inaccessible without authorization. Do not log sensitive values.
- Do not interpret production infrastructure as approval for patient-data storage before the governance controls in ADR 0001 are verified.

## Testing strategy

- Repository tests for creating and reading complete immutable certificates, including PyClone-VI input/output provenance and uncertainty records.
- Migration upgrade, rollback, and restore tests against a production-like non-sensitive PostgreSQL environment.
- Transaction/failure tests ensuring an error cannot leave a successful-but-incomplete certificate.
- Contract tests proving domain/optimizer code does not require a concrete database model.

## Deferred work

Private object storage for permitted large artifacts/report exports, multi-region data replication, sharding, and storage of raw sequencing files are deferred. Any addition of customer/patient data requires a separately approved governance and security design.
