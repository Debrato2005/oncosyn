# ADR 0005: MVP analysis pipeline and evidence boundary

## Status

Accepted for MVP planning. No application code exists yet.

## What exists and what is missing

The repository currently contains the product brief and architecture documentation only. It has no schemas, fixtures, predictor integration, PyClone-VI integration, optimizer, provenance store, API, database schema, migrations, tests, or runtime configuration.

Existing decisions already establish the product boundary: neoantigen-vaccine exploration only, immutable PostgreSQL-backed analysis runs, a synchronous first execution path, synthetic or openly usable preprocessed inputs, PyClone-VI as the upstream clonal-inference provider, and an auditable escape certificate. This record specifies how the scientific stages connect without treating a score as biological or clinical proof.

## Problem

The MVP must turn tumour information into a compact portfolio, but five concerns have different responsibilities:

1. candidate generation answers which peptide/HLA candidates are plausible under a declared predictor;
2. clonal inference estimates mutation-clone assignments and cellular prevalence from sufficient molecular evidence, while clonal mapping relates candidates to that uncertain inference output;
3. optimization chooses a set, not an individually ranked list;
4. provenance preserves the inputs and transformations behind that choice; and
5. uncertainty analysis identifies the evidence whose resolution could materially change the result.

Combining these concerns in a predictor adapter, solver, or UI would make the result impossible to audit or replace safely.

## Selected MVP design

Use one synchronous, deterministic analysis pipeline for **preprocessed tumour molecular information**. The first input contract accepts HLA alleles; annotated mutations with adequate sequence context for deterministic peptide enumeration or supplied mutant-peptide records; and a complete mutation-by-selected-sample matrix of reference/alternate read counts, allele-specific copy numbers, normal copy number, and explicit tumour content. It does not accept raw VCF/BAM files and does not perform variant calling, HLA typing, purity/copy-number estimation, or phylogeny reconstruction.

```text
validated analysis input
  -> variant/mutation representation
  -> PyClone-VI provider -> inferred clone distribution, CCF, and uncertainty
  -> peptide enumeration or supplied-peptide validation
  -> peptide/HLA prediction and scoring
  -> candidate-to-mutation-to-inferred-clone mapping
  -> named uncertainty/escape scenarios
  -> bounded portfolio optimizer
  -> sensitivity-based evidence-action ranking
  -> immutable escape certificate
  -> PostgreSQL analysis record
```

### 1. Peptide enumeration and prediction/scoring

Peptide enumeration and prediction/scoring are distinct adapters. The enumeration adapter derives mutant peptides from adequate annotated sequence context, while the input boundary validates supplied mutant-peptide records when the MVP fixture already contains them. The prediction/scoring adapter takes validated peptide/HLA pairs, invokes one versioned presentation/binding predictor, and emits normalized candidate records. A record retains the peptide, source mutation, HLA restriction, predictor name/version/settings, raw predictor output, normalized score or interval, and an explicit missing/unsupported status when scoring cannot be completed.

MHCflurry is a prediction/scoring adapter, not a peptide enumerator. Neither adapter selects a portfolio, infers clone membership, queries public evidence during a solve, or converts a predictor score into a probability of clinical benefit. Both are replaceable behind a stable normalized candidate contract.

### 2. PyClone-VI inference and clonal mapping

The clonal-inference provider transforms validated molecular records into PyClone-VI's upstream format and translates its documented output into normalized mutation-to-clone assignments with cellular-prevalence and assignment uncertainty. It requires reference/alternate read counts, major/minor allele copy number, normal copy number, and explicit tumour content. It rejects missing or incomplete values rather than fabricating them or accepting PyClone-VI's permissive default purity. Details belong in ADR 0006.

Clonal mapping joins each candidate to its source mutation and then to the inferred clone distribution. Each mapping retains cellular prevalence/CCF, its standard error, and cluster-assignment posterior. The MVP does not infer parent-child lineage from PyClone-VI clusters. A clone tree may be rendered only if a compatible reconstruction supplies lineage edges and provenance.

The output is a coverage matrix with an evidence status for every candidate/clone relationship, including absent, uncertain, and unsupported relationships. Missing clonal mapping is visible and cannot silently become coverage.

### 3. Portfolio optimization

The optimizer receives only normalized candidates, clone-coverage records, named scenarios, and a portfolio size limit `K`. It produces a bounded set that first minimizes the largest scenario-specific residual clone mass, then maximizes expected clone-weighted coverage, and finally resolves exact ties deterministically. The exact objective, constraints, solver version, tie-break rule, and solve status belong in the certificate.

A scenario is a declared, inspectable perturbation of observed or uncertain inputs, such as clone-assignment uncertainty or a supplied allele-specific presentation-loss assumption. Scenarios are not predictions of tumour evolution. Until calibrated data exists, the output is called **modelled residual uncovered mass** or a scenario-limited escape measure, not a clinical escape probability.

If no portfolio satisfies hard constraints, return a structured infeasible or explicitly partial result that names uncovered clones and violated constraints. Never force a complete-looking answer.

### 4. Evidence provenance

Provenance is data, not explanatory text added at the end. Every candidate, clone mapping, scenario input, and result must retain:

- source input or external source identity;
- source/predictor/schema/optimizer version and retrieval or execution time when applicable;
- transformation that produced the normalized value;
- evidence status, uncertainty representation, and missing-data reason; and
- its contribution to selection, exclusion, or residual risk.

The optimizer reads provenance-bearing normalized records but does not fetch databases. A public-source lookup may add external context; it cannot confirm patient-specific expression, clonality, or HLA loss. Such confirmation remains a separately labelled patient-specific verification action.

### 5. Uncertainty and next evidence actions

After baseline and declared-scenario solves, the uncertainty service perturbs one uncertain input or scenario assumption at a time within its declared bounds and re-solves. It ranks an evidence action by demonstrated decision impact: change in selected portfolio, worst-case residual clone mass, or ambiguity between equivalent portfolios. It also records feasibility and whether the action is a public external lookup or patient-specific verification.

This is a sensitivity analysis, not a claim that additional evidence is available, cheap, or sufficient. If a public source is unavailable, the action remains unresolved with a failure status rather than being fabricated or silently dropped.

## Component boundaries

| Component | Owns | Must not do |
| --- | --- | --- |
| Input schema and application service | Validation, run orchestration, typed errors. | Biological scoring, SQL, or solver formulation. |
| Peptide enumerator | Deterministic mutation/sequence-context transformation or validation of supplied peptides. | HLA prediction, clone inference, or portfolio choice. |
| Prediction/scoring adapter | Predictor invocation and normalized peptide/HLA candidate evidence. | Peptide enumeration, clone inference, portfolio choice, or clinical interpretation. |
| PyClone-VI provider | Conversion between normalized molecular records and PyClone-VI, plus normalized uncertainty-bearing assignment/prevalence output. | Domain logic, portfolio choice, ancestry claims, or silent fallback to another tool. |
| Clonal mapper | Candidate-to-mutation-to-inferred-clone coverage matrix and mapping uncertainty. | Phylogeny invention, prediction scoring, or replacement of inference uncertainty with a point fact. |
| Scenario builder and optimizer | Declared scenarios, bounded selection, solver diagnostics. | External evidence queries or persistence coupling. |
| Certificate and uncertainty service | Immutable result, provenance assembly, sensitivity/action ranking. | Mutation of source evidence or hidden reruns. |
| Repository | Immutable run/certificate persistence and retrieval. | Domain decisions or modelled biology. |

## Failure behavior

- Reject malformed HLA, peptide, mutation, read-count, copy-number, tumour-content, scenario, or `K` inputs before analysis.
- Return a typed insufficient-input or provider-unavailable result when PyClone-VI cannot receive or execute a valid inference request; do not substitute another inference algorithm.
- Return a typed unsupported-input, enumeration-failure, or predictor-failure result when the candidate path cannot produce normalized candidates.
- Return a typed mapping failure when a candidate cannot be related to inferred clone evidence; do not presume coverage.
- Preserve no-candidate, no-covered-clone, infeasible, and partial-coverage outcomes as valid, explainable results.
- Preserve optional evidence failures and unknown values explicitly. A missing source never becomes a neutral or favourable score by default.
- Persist only completed certificates as successful analysis results; incomplete runs retain a safe status/error category under ADR 0003.

## Test plan before implementation

All tests use deterministic synthetic/non-sensitive fixtures.

| Stage | Tests to write first |
| --- | --- |
| Input | Accepted/rejected schemas, immutable run identity, missing mandatory molecular evidence. |
| Clonal inference | Synthetic complete mutation/sample matrix, rejected absent counts/copy number/tumour content, exact provider serialization/parsing, unavailable provider, CCF standard error, and assignment posterior. |
| Peptide enumeration | Deterministic sequence-context fixture, supplied-peptide validation, unsupported annotation, and source-mutation retention. |
| Prediction/scoring | Fixed peptide/HLA fixture, unsupported allele/peptide, predictor version capture, and normalized missing status. |
| Clonal mapping | Candidate-to-mutation-to-inferred-clone mapping, uncertain assignment, absent mutation, no invented tree. |
| Optimizer | Overlap, `K` limit, low-prevalence uncovered clone, scenario worst case, deterministic tie, infeasible/partial result. |
| Certificate | Selected/excluded rationale, all versions retained, no hidden source lookup, safe typed failures. |
| Uncertainty | A perturbation that changes the portfolio, one that does not, action ranking, unavailable external evidence. |
| End-to-end | Synthetic input to persisted certificate; compare with naive top-K only as a baseline, never as a clinical benchmark. |

## Deferred work

- Raw sequencing ingestion, HLA typing, purity/copy-number estimation, and peptide enumeration from raw calls. The MVP permits peptide enumeration only from sufficient preprocessed sequence context.
- Probability calibration from longitudinal or clinical data, and any clinical efficacy or escape-probability claim.
- Drug-target mode, phylogeny reconstruction, asynchronous workers, Redis, object storage, and deployment work.
- Patient-derived data until ADR 0001 controls are implemented and verified.

## Consequences

The first implementation can deliver the core product claim with five replaceable modules and one synchronous path. It cannot claim that it has reconstructed evolution, predicted clinical escape, or validated a therapeutic portfolio. New inputs, predictors, scenario types, or outcome claims must revise this record and the relevant README decision before implementation.
