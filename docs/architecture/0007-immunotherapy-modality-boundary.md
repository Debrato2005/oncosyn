# ADR 0007: Immunotherapy modality boundary

## Status

Accepted for planning; nothing is implemented.

## Problem and constraints

Neoantigen vaccines, TCR-T, and conventional CAR-T do not share one target representation. Treating all as peptide/HLA selection would create invalid evidence and safety assumptions.

## Selected design and data flow

Version 1 uses a unique mutant peptide as the portfolio unit, with per-HLA presentation/binding evidence, source mutation, expression, selected-cluster support and reported uncertainty, and provenance for vaccine/T-cell research. Version 2 Track A may promote `(peptide, HLA)` to target identity for TCR-T and needs recognition, cross-reactivity, expression/processing, safety-evidence, and developability constraints. Vaccine-construct design also has its own manufacturability and assay constraints.

Version 2 Track B is a separate CAR target-and-logic model because conventional CARs primarily recognize accessible cell-surface antigens rather than peptide/HLA complexes. It requires quantitative surface abundance/accessibility and uncertainty, tumour and normal-cell states, logic feasibility, prevalence, spatial heterogeneity, stability, shedding/internalization evidence where available, and antigen-loss/downregulation scenarios. ADR 0010 owns this future research boundary.

```text
shared tumour-state/provenance
  -> Version 1 neoantigen contract -> mutant-peptide portfolio
  -> Version 2 Track A -> vaccine/TCR-T design hypotheses
  -> Version 2 Track B -> CAR target-and-logic design hypotheses
```

Generic set-selection primitives may be shared, but each modality owns eligibility, evidence, scenarios, constraints, certificate wording, and validation.

## Error/security behaviour

Reject mismatched modality records; never coerce them. Certificates name their modality and cannot generalize safety or evidence. ADR 0001 governs sensitive molecular/HLA data.

## Alternatives rejected

- One universal immunotherapy score: mechanisms and safety constraints differ.
- CAR-T as peptide/HLA selection: scientifically misleading.
- All modalities in MVP: prevents a coherent verifiable product.

## Testing and deferred work

Contract tests must reject cross-modality records and prove modality-specific constraints, provenance, and terminology. Version 2 work begins only after its modality-specific evidence and validation gate; it must not be folded into Version 1. Small-molecule portfolios remain deferred.

This boundary reflects the established mechanistic distinction that TCRs recognize peptide–MHC complexes while CARs recognize cell-surface antigens, summarized in this [authoritative review](https://pubmed.ncbi.nlm.nih.gov/39495525/). Implementation must verify and cite current evidence for detailed modality claims.
