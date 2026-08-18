# ADR 0009: Exactly three evidence-gated research versions

## Status

Accepted for planning on 2026-08-18. Version 1 remains unimplemented; Versions 2 and 3 are research directions, not delivered capabilities.

## Problem

The former multi-stage sequence fragmented one research programme into technology increments and made long-term capabilities look closer to implementation than their evidence allowed. It also placed TCR-T, CAR-T, preclinical feedback, longitudinal evidence, and closed-loop updating on one linear biological model even though their target representations and validation requirements differ.

## Selected design

OncoSyn has exactly three versions:

1. **Version 1 — Heterogeneity-Aware Neoantigen Target Portfolio Engine.** Observed-state mutant-peptide portfolios, propagated uncertainty, same-input baselines, stability analysis, and decision-focused validation prioritization.
2. **Version 2 — Modality-Specific Immunotherapy Co-Design Engine.** Track A covers vaccine/TCR-T hypotheses; Track B covers CAR target-and-logic hypotheses. The tracks share infrastructure primitives but retain different targets, evidence, response assumptions, constraints, failure modes, and validation endpoints.
3. **Version 3 — Adaptive Immunotherapy R&D Engine.** Research-state snapshots, evidence-gated stress tests, experiment selection, observations, updating, contingency hypotheses, and optional evolutionary scenarios.

The progression is evidence-gated:

- Version 1 must demonstrate mathematical correctness, fair retrospective benchmarking, stability, and prospectively testable decision-sensitivity hypotheses.
- Version 2 requires modality-specific datasets, feasible experimental definitions, and validation that its quantitative constraints materially improve research decisions over simpler baselines.
- Version 3 requires repeated observations, explicit outcome semantics, calibrated uncertainty/update models, and prospective closed-loop evaluation.

## Shared and separate primitives

Versions may share immutable EvidenceItem provenance, tumour-state snapshots, uncertainty representations, optimization primitives, ExperimentDefinition and Observation envelopes, artifact identity, and certificate conventions.

They must not share one universal target schema, score, response function, safety rule, escape model, or validation endpoint. A shared technical interface is acceptable only when its scientific semantics remain explicit.

## Claim boundary

The roadmap describes research hypotheses. It does not claim that OncoSyn currently designs vaccines, TCRs, CAR circuits, experiments, contingency therapies, or evolutionary strategies. It does not predict treatment benefit, safety, relapse, resistance prevention, or patient tumour evolution.

## Alternatives rejected

- **Retain the former multi-stage roadmap:** rejected because implementation increments obscured the three scientific questions.
- **One universal immunotherapy optimizer:** rejected because peptide–HLA, TCR, vaccine-construct, and surface-antigen CAR designs have different biology.
- **Make evolution the entire product:** rejected because transition models are often unidentifiable and experiment selection may be the stronger near-term research contribution.
- **Reduce OncoSyn to Version 1:** rejected because it abandons the approved research mission rather than placing ambition behind evidence gates.

## Documentation and testing consequences

README, active brief, validation contract, checklist, walkthrough, contributor instructions, and affected ADRs use only the three-version terminology. Tests introduced with each version must prove claim wording, modality separation, provenance, negative-result retention, and evidence-gate enforcement.
