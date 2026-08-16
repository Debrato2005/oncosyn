# ADR 0001: Research-data boundary and controlled customer-data enablement

## Status

Accepted for the production-oriented product. No application or production-control implementation exists yet.

## Problem

OncoSyn is intended to process tumour and HLA-related information, which can be sensitive. The repository has no authentication, authorization, encryption, retention policy, secret delivery, audit logging, data-processing agreement, or protected-data infrastructure. The product needs reproducible examples to exercise candidate generation and portfolio selection before customer data is enabled.

## Constraints

- OncoSyn is research decision support, not a clinical decision system.
- The repository currently has no runtime or data store.
- Public evidence can inform a biological prior but cannot verify a patient-specific observation.
- The project must be demonstrable and reproducible without access-controlled data.

## Selected design

Use synthetic or openly usable, preprocessed and non-sensitive analysis fixtures until the production controls below are implemented and verified. Treat each analysis input and output as a versioned immutable snapshot. Keep external/public evidence sources separate from patient-specific verification actions in both the model and the presentation.

Before customer/patient-derived data can be enabled, implement and approve: workspace authorization; audit logging; encrypted transport and storage; secret management; retention/deletion and backup/restore processes; incident response; data classification; and access review. Record verification evidence for those controls before enabling the feature.

No raw patient sequencing, identifiers, access-controlled datasets, tokens, or credentials may be committed. A patient-data workflow is not enabled merely by adding an upload endpoint.

## Alternatives rejected

- **Use public cohort evidence as patient confirmation:** rejected because cohort-level evidence does not establish a particular tumour's expression, clonality, or HLA status.
- **Accept raw sequencing in the MVP:** rejected because variant calling, quality control, consent, security, storage, and retention controls are absent.
- **Store mutable “latest” analysis results:** rejected because it undermines provenance and reproducibility.

## Intended data flow

```text
synthetic/public preprocessed fixture
  -> validated immutable analysis input
  -> versioned candidate and clone evidence
  -> versioned portfolio result and escape certificate
```

If public evidence is queried in a future implementation, record source identity, retrieval time, source version when available, query parameters, and result status. If a source is unavailable, preserve the failure as missing evidence; do not substitute a success or fabricate a value.

## Error and security behavior

- Reject inputs outside the declared schema before domain processing.
- Never put raw inputs, HLA types, identifiers, credentials, or tokens in logs or error responses.
- Return safe, typed failures for unavailable evidence and incompatible data.
- Do not cache sensitive inputs or results until a cache security/retention design is accepted.

## Testing strategy

- Use only synthetic/non-sensitive fixtures.
- Test schema rejection, immutable input snapshots, source-provenance capture, unavailable-source handling, and absence of sensitive values in structured logs once logging exists.
- Add security, authorization, retention, encryption, backup, deletion, and audit tests before any patient-data feature is proposed for release.

## Deferred work

Raw sequencing ingestion, consent/governance, clinical integration, and any regulated clinical use each require a follow-up ADR and implementation-specific tests.
