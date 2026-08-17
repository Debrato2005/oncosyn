# ADR 0002: Escape-aware hypothesis certificate

## Status

Accepted for planning; no certificate is implemented.

## Problem and constraints

A selected list or single score cannot show why a portfolio is preferable, how uncertain clonal evidence affected it, or what could falsify the conclusion. Output must remain research-only and distinguish observed evidence from model assumptions.

## Selected design

Each successful or partial run emits an immutable certificate expressing:

> Given tumour-state snapshot `S`, evidence `E`, constraints `C`, and scenarios `R`, portfolio `P` is predicted to be more robust under the declared objective than named baseline `B`.

The certificate contains:

1. selected `K` candidates and exclusions;
2. each candidate's source mutation;
3. candidate-to-inferred-clone distributions and uncertainty;
4. candidate quality/evidence and complete provenance;
5. named escape/uncertainty scenarios and modelled coverage/residual uncovered mass;
6. comparison with score Top-`K`, clonality-weighted, coverage-only, and reproducible escape/minimax baselines;
7. input tier, provider/predictor/optimizer/solver/schema/source versions and settings;
8. assumptions, limitations, missing evidence, eligibility decisions, and failure modes;
9. sensitivity: which evidence changes selection or objective values; and
10. next experiment, public evidence check, or patient-specific verification most likely to resolve a decision.

Candidate generation, prediction, clonal mapping, optimization, baselines, and certificate assembly remain separate. The certificate never calls modelled risk a clinical probability.

## Status and interpretation

Output status is `complete`, `partial`, `infeasible`, or typed failure. Unknown evidence cannot default to favourable. A threshold-passing result is labelled a **computational hit promoted for downstream investigation**, not a validated hit. Equivalent or inferior performance to a baseline must be reported.

## Data flow, errors, and security

```text
immutable inputs/provider results -> scenarios + optimizer + baselines
  -> sensitivity/provenance assembly -> immutable certificate -> repository
```

The certificate references sensitive inputs by controlled identifiers and never exposes raw molecular records, secrets, stack traces, or private artifact paths. Invalid inputs, unavailable providers, unsupported candidates, mapping gaps, and infeasible constraints remain typed and visible; no incomplete run receives a complete-looking certificate.

## Testing strategy

Golden synthetic tests cover all fields, versions, assumptions, selection/exclusion rationale, uncertainty, equivalent/inferior baselines, partial/infeasible states, deterministic serialization, and safe redaction. Sensitivity tests include perturbations that change and do not change the portfolio. End-to-end fixtures test the central hypothesis without clinical claims.

## Alternatives and deferred work

Rejected: selected-list-only output, one unexplained score, Top-`K` as the only baseline, hidden external lookups, clinical escape probability, and automatic clinical interpretation. Deferred: downloadable reports, signatures, outcome calibration, regulatory documents, and prospective validation.
