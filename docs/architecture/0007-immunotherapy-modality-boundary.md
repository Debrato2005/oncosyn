# ADR 0007: Immunotherapy modality boundary

## Status

Accepted for planning; nothing is implemented.

## Problem and constraints

Neoantigen vaccines, TCR-T, and conventional CAR-T do not share one target representation. Treating all as peptide/HLA selection would create invalid evidence and safety assumptions.

## Selected design and data flow

The MVP uses mutant peptide, HLA, presentation/binding, source mutation, expression, uncertain clone distribution, and provenance for vaccine/T-cell portfolios. TCR-T is a future extension with recognition, cross-reactivity, expression/processing, safety, and developability constraints.

Conventional CAR-T is a later separate model because it primarily recognizes accessible cell-surface antigens rather than peptide/HLA complexes. It requires surface abundance/accessibility, tumour specificity, normal-tissue/off-tumour risk, prevalence, spatial heterogeneity, stability, and antigen-loss evidence.

```text
shared tumour-state/provenance
  -> neoantigen contract -> vaccine/TCR-oriented portfolio (MVP)
  -> future surface-antigen contract -> CAR-T portfolio (separate validation)
```

Generic set-selection primitives may be shared, but each modality owns eligibility, evidence, scenarios, constraints, certificate wording, and validation.

## Error/security behaviour

Reject mismatched modality records; never coerce them. Certificates name their modality and cannot generalize safety or evidence. ADR 0001 governs sensitive molecular/HLA data.

## Alternatives rejected

- One universal immunotherapy score: mechanisms and safety constraints differ.
- CAR-T as peptide/HLA selection: scientifically misleading.
- All modalities in MVP: prevents a coherent verifiable product.

## Testing and deferred work

Contract tests must reject cross-modality records and prove modality-specific constraints, provenance, and terminology. TCR-T is V3; a dedicated surface-antigen/CAR-T ADR and validation are V4. Small-molecule portfolios remain deferred.

This boundary reflects the established mechanistic distinction that TCRs recognize peptide–MHC complexes while CARs recognize cell-surface antigens, summarized in this [authoritative review](https://pubmed.ncbi.nlm.nih.gov/39495525/). Implementation must verify and cite current evidence for detailed modality claims.
