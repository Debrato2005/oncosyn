# OncoSyn MVP Problem Statement

> This is the active **OncoSyn** MVP problem statement. It is aligned with [`README.md`](README.md), [`docs/architecture/0005-mvp-analysis-pipeline.md`](docs/architecture/0005-mvp-analysis-pipeline.md), and [`docs/architecture/0006-pyclone-vi-integration.md`](docs/architecture/0006-pyclone-vi-integration.md).

## Problem

Develop a modular research engine that accepts preprocessed tumour molecular evidence and HLA alleles; uses PyClone-VI to infer uncertainty-bearing clone/subclone assignments and cellular prevalence; generates and scores neoantigen candidates; and selects a bounded portfolio that minimizes **modelled worst-case residual clone mass** under declared assumptions. Every selected or excluded candidate must be traceable to its evidence, and each material uncertainty must be paired with the next evidence action most likely to change the decision.

In one sentence: do not return the highest-scoring peptides independently; infer clone evidence from sufficient molecular inputs, then select the peptide set that best covers it under declared uncertainty and show why.

## MVP scope

- **Product track:** neoantigen-vaccine portfolio exploration only. Drug-target optimization is a future product track with a separate model and design decision.
- **Accepted inputs:** preprocessed mutation/sample read counts; allele-specific major/minor and normal copy numbers; explicit tumour content; annotated mutations with sequence context needed to enumerate mutant peptides or supplied mutant-peptide records; HLA alleles; and optional expression observations.
- **Not accepted:** raw VCF/BAM ingestion, variant calling, HLA typing, purity/copy-number estimation, or invented lineage. Insufficient PyClone-VI evidence is rejected rather than completed with defaults.
- **Output:** a size-bounded peptide portfolio, expected coverage and modelled escape-risk measures, clone-coverage summary, evidence provenance, uncertainty sensitivity, and ranked next evidence actions.
- **Research boundary:** this is investigation support. It does not establish biological efficacy, prevent tumour escape, prescribe treatment, or replace experimental and expert validation.

## MVP architecture

```text
preprocessed tumour molecular data + HLA
  -> variant/mutation representation
  -> PyClone-VI inference -> clone assignments + CCF + uncertainty
  -> peptide enumeration
  -> presentation/binding prediction and scoring (MHCflurry adapter)
  -> normalized candidate evidence
  -> candidate-to-mutation-to-inferred-clone mapping
  -> declared uncertainty / escape scenarios
  -> bounded OncoSyn portfolio optimizer
  -> portfolio certificate
      ├-> selected peptides and exclusions
      ├-> expected coverage and modelled worst-case residual clone mass
      └-> evidence provenance and next evidence actions
```

### Candidate enumeration is not prediction

Peptide enumeration derives possible mutant peptides from sufficiently annotated mutation/sequence context, or validates supplied peptide records. Presentation/binding prediction scores those peptides for declared HLA alleles. MHCflurry belongs only to the second step: it is a predictor/scorer, not the source of the complete candidate universe.

The MVP can begin with supplied mutant-peptide records in its synthetic fixtures. A deterministic enumeration adapter is added only when the input contract contains enough sequence context to test it reproducibly.

### PyClone-VI clonal inference is not phylogeny reconstruction

PyClone-VI consumes validated reference/alternate read counts, allele-specific copy-number, normal copy-number, and explicit tumour-content records to estimate mutation cluster assignments and cellular prevalence/CCF. Candidate-to-clone mapping joins a candidate's source mutation to this uncertainty-bearing distribution. PyClone-VI does not derive a definitive tumour tree. The MVP displays clone coverage; a lineage tree is allowed only when compatible parent-child edges are supplied by an explicit reconstruction and retained as evidence.

### Portfolio objective

The optimizer selects at most `K` candidates from normalized candidate and clone-coverage records. It minimizes the largest residual clone mass across named scenarios, then maximizes expected clone-weighted coverage, with deterministic tie-breaking. Scenarios are explicit perturbations of input uncertainty or supported presentation-loss assumptions; they are not a prediction of clinical evolution.

The certificate reports the mathematical objective and its assumptions. It may call the result a **modelled escape-risk score** in a presentation layer, provided the technical definition remains visible. It must not call it an escape probability unless that quantity is separately calibrated and validated.

## What is new

PyClone-VI is an upstream biological inference component, not OncoSyn's novel algorithm. Existing tools infer clone structure or enumerate/score candidates. OncoSyn's contribution is the collective selection and explanation layer: it decides which candidates belong together for the inferred clonal distribution, exposes the residual uncovered clone mass in named scenarios, and records the evidence and uncertainty behind the decision.

The optimizer guarantees only its stated mathematical objective under its declared inputs and assumptions. It does not establish biological efficacy, prevention of tumour escape, or a clinical outcome.

## Validation plan

- Compare the bounded portfolio with naive individual-score top-`K` selection on deterministic synthetic clone scenarios.
- Test overlap, low-prevalence uncovered clones, `K` limits, deterministic ties, incomplete evidence, and infeasible constraints.
- Verify that every result retains input, PyClone-VI provider, enumeration, predictor, clone-mapping, scenario, and optimizer versions.
- Verify that uncertainty perturbations change the ranked evidence actions only when they demonstrate a decision impact.

## Build order

1. Input/domain contracts and immutable PostgreSQL run snapshot.
2. Variant representation and PyClone-VI clonal inference.
3. Peptide enumeration or supplied-peptide validation, followed by MHCflurry scoring.
4. Candidate-to-mutation-to-inferred-clone mapping.
5. Bounded scenario portfolio optimization.
6. Certificate, provenance, uncertainty sensitivity, and persisted result.

FastAPI, PostgreSQL, React, authentication, and observability remain selected architecture targets. The first implementation must not build the delivery, deployment, or identity layers before the analysis pipeline is verified; see [`docs/backend-build-checklist.md`](docs/backend-build-checklist.md).
