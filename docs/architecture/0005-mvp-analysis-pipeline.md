# ADR 0005: Version 1 analysis pipeline contract

## Status

Accepted for planning. No pipeline is implemented.

## Problem and constraints

OncoSyn must turn processed tumour evidence into a small, auditable neoantigen portfolio without collapsing candidate generation, clonal inference, prediction, mapping, optimization, provenance, uncertainty, or decision sensitivity into one opaque ranker. Upstream tools are replaceable evidence providers. Because prior work including NeoAgDT already optimizes bounded neoantigen vaccine compositions over tumour heterogeneity, OncoSyn's uncertainty-aware robust-selection and experiment-prioritization approach is a research hypothesis requiring fair benchmarks, not established novelty.

The MVP accepts Tier A targeted-panel or Tier B research-rich processed results. It does not accept raw sequencing, call variants, type HLA, estimate purity/CN, reconstruct definitive ancestry, or infer clinical outcomes. A sparse panel must not be presented as complete heterogeneity.

## Selected synchronous flow

```text
validated tiered input + HLA -> immutable snapshot
  -> normalized mutations
  -> explicit NeoantigenCandidateProvider
     -> pVACseq all_epitopes (eligible Tier B VEP input)
     -> native enumeration + MHCflurry (processed protein/supplied peptides)
  -> normalized broad candidate evidence
  -> explicit ClonalInferenceProvider
     -> PyClone-VI (adequate exact input) OR validated precomputed evidence
  -> selected cluster assignments/prevalence + reported uncertainty
  -> candidate -> mutation -> uncertain cluster evidence
  -> named measurement-uncertainty and sourced adverse scenarios
  -> same-input baseline suite + bounded OncoSyn optimizer
  -> stability + bounded decision sensitivity
  -> feasible evidence-check/experiment ranking by declared decision impact
  -> immutable hypothesis certificate -> PostgreSQL
```

### Input and clonal inference

The boundary records tier/subtype, observed scope, sample design, HLA, stable mutation identities, actual allele counts where required, CN, explicit tumour content, sequence context/supplied peptides or normalized VEP-annotated VCF evidence, optional expression, and all reference/source versions. VAF/depth may support assessment but cannot become pseudo-counts.

`ClonalInferenceProvider` returns normalized, provenance-bearing selected mutation/cluster assignments, prevalence/CCF, reported standard error and selected-cluster assignment probability, exclusions, and limitations. `PyCloneVIProvider` is first; validated precomputed evidence is an explicit alternative, not fallback. Provider formats never reach domain/optimizer. Details and panel eligibility are in ADR 0006. Clusters are not ancestry or mutually exclusive tumour fractions.

### Candidate generation and evidence

ADR 0008 owns candidate generation. Eligible Tier B VEP inputs use `PVACSeqProvider` and normalize `all_epitopes.tsv` before Top Score reduction. Adequate processed protein context or supplied peptides use `NativeOncoSynProvider`, which keeps enumeration separate from MHCflurry prediction. Selection is explicit; providers never mix or silently fall back. No proprietary MHC model is trained for MVP.

The MVP optimization unit is a unique mutant peptide with per-HLA evidence. Normalized evidence retains source mutation, transcript/consequence when present, provider/runtime/predictor/model/version/settings, raw and normalized results, wild-type comparison when present, uncertainty/missing status, filters, licenses, and provenance. Expression, tumour specificity, antigen processing, immunogenicity, disagreement, and other evidence may be added only with explicit sources and semantics. Eligibility, quality, clonal evidence, and objective terms remain distinct; evidence is not clinical truth.

### Mapping, scenarios, optimizer, and baselines

Clonal mapping joins every candidate to its stable source mutation and selected inferred cluster, retaining prevalence error, selected-cluster assignment probability, evidence status, and panel limitations. It does not invent a full assignment posterior. Missing mapping never becomes support.

The optimizer consumes only normalized candidates/mappings, declared scenarios/constraints, and `K`. Cluster CCFs are not summed. The initial formulation minimizes a declared worst-case uncovered clonal-support score across clusters/scenarios, then applies versioned candidate-quality/evidence-completeness objectives and deterministic tie-breaking. CP-SAT is the planned first solver because decisions are primarily Boolean; fixed-point scaling, seed, limits, and solver status are provenance. Scenario perturbations are sourced measurement/adverse assumptions, not predicted evolution. Return complete, partial, infeasible, or typed solver diagnostics; never force a portfolio or report clinical escape probability.

Run selection-only comparisons on identical eligible candidates, input, `K`, and constraints: score Top-K, clonality-weighted ranking, cluster-support-only selection, reproducible minimax, pVACseq aggregate ranking mapped back to the universe, and OncoSyn. Compare against NeoAgDT-style MinSum/MinMax where reproducible and legally usable. Full-pipeline comparisons are allowed only where branches consume equivalent source evidence. Equivalent, inferior, excluded, and non-comparable results remain visible.

### Certificate and uncertainty

The certificate states: given tumour state, evidence, assumptions, and scenarios, portfolio `P` achieved the declared computational objective relative to named baseline `B`. It does not assert superiority unless the recorded benchmark supports it. ADR 0002 owns required fields. Sensitivity re-solves rank public evidence checks, patient-specific verification, or feasible experiments by demonstrated decision impact. This is bounded decision sensitivity, not Bayesian optimal experimental design. A promoted computational hit and a ranked experiment remain experimental hypotheses.

## Component boundaries

| Component | Owns | Must not do |
| --- | --- | --- |
| Schema/service | Validation, eligibility, orchestration, typed status. | Biology, SQL, solver formulation. |
| Candidate provider | Runtime/input eligibility, broad candidate normalization, version/filter/license provenance. | Clonal inference, selection, silent fallback/mixing. |
| Clonal provider | Input conversion and normalized uncertain clone evidence. | Selection, ancestry, silent fallback. |
| Enumerator | Peptide derivation/validation. | Prediction or selection. |
| Predictor/evidence | Versioned candidate evidence. | Enumeration, selection, clinical interpretation. |
| Mapper | Candidate-mutation-cluster support and reported uncertainty. | Invent a posterior, lineage, or additive tumour mass. |
| Scenario/optimizer/baselines | Declared perturbations, bounded selection, diagnostics/comparisons. | Lookups, provider formats, persistence. |
| Certificate/sensitivity | Provenance, hypothesis, next actions. | Hidden evidence mutation/reruns. |
| Experiment definition | Feasible assay context, outcome vocabulary, evidence dependency, and decision linkage. | Treat a planned or predicted outcome as observed. |
| Repository | Immutable transactions/retrieval. | Domain decisions. |

## Failure/security behaviour

Reject malformed, insufficient, incompatible, cross-modality, cross-provider, or licensing-disabled input before execution. Preserve candidate/clonal provider unavailable or mis-versioned, enumeration/prediction failure, malformed output, unmatched mutation identity, mapping gap, no-candidate/no-supported-cluster, partial/infeasible/unknown solve, evidence unavailable, and internal failure as typed statuses. Unknown evidence is not neutral/favourable. Never expose molecular values, identifiers, secrets, private paths, or stack traces. Only completed/explicit partial certificates are publishable with their status.

## Test strategy

Use deterministic synthetic/non-sensitive fixtures. Test both input tiers/subtypes and explicit provider eligibility; pVACseq and PyClone serialization/parsing/version/unavailability; native enumeration and MHCflurry separately; stable mutation identity; mapping/no-tree/no-additive-CCF rules; fixed-point scaling; every baseline and OncoSyn-inferior/equivalent/non-comparable cases; certificate provenance/claims; sensitivity change/no-change; safe failures; immutable PostgreSQL transaction/migration flow. Tool-specific golden tests remain offline and separately marked after pinned environments exist.

## Alternatives rejected and deferred work

Rejected: one ranker, native genomic-consequence reimplementation, pVACseq-only, pVACseq Top Score as the candidate universe, one modality-universal score, provider formats in domain, VAF pseudo-counts/default biology, additive CCF mass, Top-K-only evaluation, hidden lookups, forced portfolios, uncalibrated Bayesian language, and clinical conclusions. Deferred: mixed-provider harmonization, additional pVACtools modes/predictors, Version 2 modality-specific vaccine/TCR-T and CAR models, Version 3 observation-driven updating/evolution, raw sequencing, small-molecule targets, outcome calibration, async workers, object storage, and deployment.
