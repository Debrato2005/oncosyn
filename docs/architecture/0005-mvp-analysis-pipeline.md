# ADR 0005: MVP analysis pipeline contract

## Status

Accepted for planning. No pipeline is implemented.

## Problem and constraints

OncoSyn must turn processed tumour evidence into a small, auditable neoantigen/T-cell portfolio without collapsing candidate generation, clonal inference, prediction, mapping, optimization, provenance, and uncertainty into one opaque ranker. Upstream tools are replaceable evidence providers; OncoSyn's contribution is portfolio selection and its falsifiable explanation.

The MVP accepts Tier A targeted-panel or Tier B research-rich processed results. It does not accept raw sequencing, call variants, type HLA, estimate purity/CN, reconstruct definitive ancestry, or infer clinical outcomes. A sparse panel must not be presented as complete heterogeneity.

## Selected synchronous flow

```text
validated tiered input + HLA -> immutable snapshot
  -> normalized mutations
  -> explicit ClonalInferenceProvider
     -> PyClone-VI (adequate exact input) OR validated precomputed evidence
  -> uncertain clone assignments/prevalence
  -> peptide enumeration or supplied-peptide validation
  -> presentation/binding prediction (MHCflurry first)
  -> normalized candidate evidence
  -> candidate -> mutation -> uncertain clone distribution
  -> named uncertainty/escape scenarios
  -> same-input baseline suite + bounded OncoSyn optimizer
  -> sensitivity and evidence-action ranking
  -> immutable hypothesis certificate -> PostgreSQL
```

### Input and clonal inference

The boundary records tier, observed scope, sample design, HLA, mutations, actual allele counts where required, CN, explicit tumour content, sequence context/supplied peptides, and optional expression. VAF/depth may support assessment but cannot become pseudo-counts.

`ClonalInferenceProvider` returns normalized, provenance-bearing mutation/cluster assignments, prevalence/CCF, uncertainty, exclusions, and limitations. `PyCloneVIProvider` is first; a validated precomputed-evidence provider is an explicit alternative, not fallback. Provider formats never reach domain/optimizer. Details and panel eligibility are in ADR 0006. Clusters are not ancestry; visualization requires separate compatible reconstruction.

### Candidate generation and evidence

Enumeration derives peptides from adequate mutation/sequence context or validates supplied peptides. A separate predictor scores peptide/HLA pairs. MHCflurry is initial; NetMHCpan, pVACtools-derived, and other providers remain replaceable. No proprietary MHC model is trained for MVP.

Normalized candidate evidence always retains peptide, source mutation, HLA, provider/version/settings, raw/normalized result, uncertainty/missing status, and provenance. Expression, clonality, tumour specificity, antigen processing, immunogenicity, multi-predictor disagreement, and other evidence may be added only with explicit sources and semantics. Evidence is not clinical truth.

### Mapping, scenarios, optimizer, and baselines

Clonal mapping joins every candidate to its source mutation and inferred clone distribution, retaining prevalence error, assignment posterior, evidence status, and panel limitations. Missing mapping never becomes coverage.

The optimizer consumes only normalized candidates/mappings, declared scenarios/constraints, and `K`. It minimizes declared worst-case residual uncovered clone mass, then applies documented secondary quality/coverage objectives and deterministic tie-breaking. Scenario perturbations are inspectable assumptions, not predicted evolution. Return complete, partial, or infeasible diagnostics; never force a portfolio or report clinical escape probability.

Run comparisons on identical eligible candidates, input, `K`, and constraints: score Top-K, clonality-weighted ranking, coverage-only selection, reproducible escape/minimax where comparable, and OncoSyn. Equivalent or inferior OncoSyn results remain visible.

### Certificate and uncertainty

The certificate states: given tumour state, evidence, assumptions, and scenarios, portfolio `P` is predicted computationally more robust than named baseline `B`. ADR 0002 owns required fields. Sensitivity re-solves rank public evidence checks, patient-specific verification, or experiments by demonstrated decision impact. A promoted computational hit remains an experimental hypothesis.

## Component boundaries

| Component | Owns | Must not do |
| --- | --- | --- |
| Schema/service | Validation, eligibility, orchestration, typed status. | Biology, SQL, solver formulation. |
| Clonal provider | Input conversion and normalized uncertain clone evidence. | Selection, ancestry, silent fallback. |
| Enumerator | Peptide derivation/validation. | Prediction or selection. |
| Predictor/evidence | Versioned candidate evidence. | Enumeration, selection, clinical interpretation. |
| Mapper | Candidate-mutation-clone matrix and uncertainty. | Invent lineage or certainty. |
| Scenario/optimizer/baselines | Declared perturbations, bounded selection, diagnostics/comparisons. | Lookups, provider formats, persistence. |
| Certificate/sensitivity | Provenance, hypothesis, next actions. | Hidden evidence mutation/reruns. |
| Repository | Immutable transactions/retrieval. | Domain decisions. |

## Failure/security behaviour

Reject malformed, insufficient, incompatible, or cross-modality input before execution. Preserve provider unavailable, enumeration/prediction failure, mapping gap, no-candidate/no-covered-clone, partial/infeasible solve, evidence unavailable, and internal failure as typed statuses. Unknown evidence is not neutral/favourable. Never expose molecular values, identifiers, secrets, private paths, or stack traces. Only completed/explicit partial certificates are publishable with their status.

## Test strategy

Use deterministic synthetic/non-sensitive fixtures. Test both input tiers and eligibility; provider serialization/precomputed validation/unavailability/uncertainty; enumeration and prediction separately; mapping and no-tree rules; each optimizer baseline and OncoSyn-inferior/equivalent cases; certificate provenance/claims; sensitivity change/no-change; safe failures; immutable PostgreSQL transaction/migration flow. A separately marked real PyClone-VI test runs offline only after its pinned environment exists.

## Alternatives rejected and deferred work

Rejected: one ranker, one modality-universal score, PyClone formats in domain, VAF pseudo-counts/default biology, Top-K-only evaluation, hidden lookups, forced portfolios, and clinical conclusions. Deferred: TCR-T-specific V3, separate CAR-T V4, preclinical/longitudinal feedback, raw sequencing, small-molecule targets, outcome calibration, async workers, object storage, and deployment.
