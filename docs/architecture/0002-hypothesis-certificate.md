# ADR 0002: Research hypothesis and decision-sensitivity certificate

## Status

Accepted for planning; no certificate is implemented.

## Problem and constraints

A selected list or single score cannot show why a portfolio is preferable, how uncertain clonal evidence affected it, or what could falsify the conclusion. Output must remain research-only and distinguish observed evidence from model assumptions.

## Selected design

Each successful or partial run emits an immutable certificate expressing:

> Given tumour-state snapshot `S`, evidence `E`, constraints `C`, and scenarios `R`, portfolio `P` achieved the declared model objective relative to named baseline `B`; the result remains a research hypothesis.

The certificate contains:

1. selected `K` candidates and exclusions;
2. each candidate's source mutation;
3. candidate-to-source-mutation-to-selected-cluster evidence, including the uncertainty actually reported by the selected clonal provider;
4. candidate quality/evidence and complete provenance;
5. named measurement/adverse scenarios, per-cluster candidate support, and explicitly defined uncovered clonal-support scores that are not tumour-mass fractions;
6. comparison with score Top-`K`, clonality-weighted, cluster-support-only, reproducible minimax, pVACseq ranking, and prior-art optimization where comparable;
7. input tier, reference/VEP data where applicable, candidate/clonal provider, predictor/model/license, optimizer/solver/schema/source versions and settings;
8. assumptions, limitations, missing evidence, eligibility decisions, and failure modes;
9. portfolio stability and sensitivity: which evidence, provider/configuration choice, or declared perturbation changes selection or objective values;
10. feasible next experiments, public evidence checks, or patient-specific verification ranked by declared decision impact; and
11. the acquisition method and its limits—for Version 1, bounded re-solving/sensitivity rather than uncalibrated Bayesian experimental design.

Candidate generation, prediction, clonal mapping, optimization, baselines, experiment-definition assembly, and certificate assembly remain separate. A proposed experiment is not an observation. PyClone cluster prevalences are not summed as mutually exclusive tumour mass, and the certificate never calls a model score or risk a clinical probability.

## Status and interpretation

Output status is `complete`, `partial`, `infeasible`, or typed failure. Unknown evidence cannot default to favourable. Evidence is labelled directly observed, inferred, literature-prior, hypothetical, or currently unidentifiable. A threshold-passing result is labelled a **computational hit promoted for downstream investigation**, not a validated hit. Equivalent or inferior performance to a baseline must be reported.

## Data flow, errors, and security

```text
immutable inputs/provider results -> scenarios + optimizer + baselines
  -> sensitivity/provenance assembly -> immutable certificate -> repository
```

The certificate references sensitive inputs by controlled identifiers and never exposes raw molecular records, secrets, stack traces, or private artifact paths. Invalid inputs, unavailable providers, unsupported candidates, mapping gaps, and infeasible constraints remain typed and visible; no incomplete run receives a complete-looking certificate.

## Testing strategy

Golden synthetic tests cover all fields, versions, assumptions, evidence classifications, selection/exclusion rationale, uncertainty, equivalent/inferior baselines, partial/infeasible states, deterministic serialization, and safe redaction. Sensitivity tests include perturbations that change and do not change the portfolio and experiment rankings. End-to-end fixtures test the central hypothesis without clinical claims.

## Alternatives and deferred work

Rejected: selected-list-only output, one unexplained score, Top-`K` as the only baseline, hidden external lookups, clinical escape probability, and automatic clinical interpretation. Deferred: downloadable reports, signatures, outcome calibration, regulatory documents, and prospective validation.
