# Version 1 Backend Build Checklist

Legend: **implemented** = repository-verified; **in progress** = evidence exists but acceptance is incomplete; **planned** = agreed work; **deferred** = intentionally later. The repository is documentation-only, so every implementation item is **planned**.

For every milestone: write the listed deterministic synthetic tests first; implement only that boundary; run the real configured format/lint/type/test/migration commands; update README, walkthrough, affected ADR, and this checklist; then use the focused commit suggestion. Commands must be added only when tooling exists.

## Milestone 0 — Foundation — planned

- [ ] Tests first: one synthetic schema rejection test.
- [ ] Implement: Python 3.11/`uv`/`pyproject.toml`, pinned app dependencies, source/test layout, pytest and selected formatter/linter/type checker; provider executable/version checks and synchronous service boundary only. Do not force pVACtools and PyClone-VI into the app environment.
- [ ] Verify: record and run exact install/static/test commands.
- [ ] Docs/commit: update repository map and commands; `chore(platform): establish reproducible backend tooling`.

## Milestone 1 — Input and domain contracts — planned

- [ ] Tests first: Tier A/targeted, Tier B/research-rich, and Tier B VEP-subtype acceptance; HLA, normalized genomic/protein-only mutation identity, multiallelic decomposition, sample identity, actual counts, VAF/depth assessment, CN/purity, sequence context/supplied peptides, optional RNA, reference/VEP versions, and immutable snapshots.
- [ ] Implement: versioned schemas; input-tier/subtype and observed-scope labels; explicit candidate/clonal-provider eligibility; stable provider-independent mutation IDs; limited-panel status; PostgreSQL snapshot through repository/migration. No raw data, calling, pseudo-counts, HLA typing, CN/purity estimation, or tree inference.
- [ ] Verify: configured schema/static/migration tests.
- [ ] Docs/commit: document actual schema/errors; `feat(contract): define tiered analysis input`.

## Milestone 2 — Variant and clonal inference — planned

- [ ] Tests first: generic provider contract; PyClone-VI serialization/parser; every required actual count/CN/tumour-content field; matrix completeness; VAF-only rejection; sparse-panel limitation; unavailable/mis-versioned provider; validated precomputed evidence; uncertainty fields; provider/configuration/seed sensitivity hooks; no fallback/ancestry.
- [ ] Implement: normalized variant model and `ClonalInferenceProvider`; explicit `PyCloneVIProvider`, `PrecomputedClonalEvidenceProvider`, and labelled deterministic fixture provider. Pin real PyClone-VI externally if runtime compatibility requires it.
- [ ] Verify: unit/golden/fixture contracts; separately marked offline real-provider test when provisioned. Benchmark targeted-panel adequacy and provider/configuration/seed sensitivity rather than inventing thresholds or treating PyClone-VI as a universal default.
- [ ] Docs/commit: record actual version/runtime/contract; `feat(clonal-inference): add provider boundary`.

## Milestone 3 — Peptide enumeration and candidate scoring — planned

- [ ] Tests first: shared `NeoantigenCandidateProvider` contract; explicit eligibility/no-fallback/no-mixing; pVACseq `all_epitopes` golden parsing before Top Score; deterministic native enumeration/supplied-peptide validation; source mutation joins; per-HLA MHCflurry evidence; unsupported consequence/allele, missing model, malformed output, timeout, and licensing-disabled predictor.
- [ ] Implement: explicit `PVACSeqProvider` for eligible Tier B VEP input and `NativeOncoSynProvider` for processed protein/supplied peptides; unique-peptide optimization units with per-HLA evidence; private subprocess artifacts/version checks; broad normalized evidence vectors. No aggregate/filtered pVACseq universe, proprietary MHC model, or undocumented predictor license.
- [ ] Verify: contract/unit/golden tests plus separately marked pinned-runtime smoke tests; prove provider formats and paths do not cross the domain boundary.
- [ ] Docs/commit: record exact versions, VEP/reference/model data, filters, licenses, mappings, and runtime; `feat(candidates): add hybrid candidate providers`.

## Milestone 4 — Candidate-to-clone mapping — planned

- [ ] Tests first: candidate -> stable mutation -> selected inferred cluster, prevalence error, selected-cluster assignment probability, precomputed evidence, absent relationships, limited ascertainment, no invented posterior/tree/additive mass.
- [ ] Implement: uncertainty-bearing candidate/cluster support matrix; unsupported relationships never become support.
- [ ] Verify: mapping tests and synthetic pipeline through this stage.
- [ ] Docs/commit: document representation/failures; `feat(clonal-mapping): preserve mapping uncertainty`.

## Milestone 5 — Portfolio optimizer and baselines — planned

- [ ] Tests first: overlap, `K`, unsupported clusters, ties, partial/infeasible/unknown results, measurement scenarios, fixed-point scaling, deterministic diagnostics; same-set/`K` score Top-K, clonality-weighted, cluster-support-only, reproducible minimax, pVACseq ranking, prior-art MinSum/MinMax where possible, and OncoSyn comparisons.
- [ ] Implement: separate eligibility, evidence vector, cluster support, and objective; bounded lexicographic robust selection over explicit scenarios; per-cluster support and scenario-weighted uncovered-support scores—never summed tumour mass or clinical escape probability.
- [ ] Verify: CP-SAT against brute force on small cases and solver/end-to-end synthetic cases including OncoSyn-inferior/equivalent/non-comparable outcomes.
- [ ] Docs/commit: publish actual objective/constraints/tie rules; `feat(optimizer): select escape-aware portfolios`.

## Milestone 6 — Evidence, uncertainty, and certificate — planned

- [ ] Tests first: every certificate field in ADR 0002; complete provenance/versioning; missing sources; sensitivity change/no-change; portfolio stability; typed feasible experiment/evidence-check definitions; ranked next action; all baselines; computational-hit label; immutable persistence; safe partial/failure status.
- [ ] Implement: falsifiable hypothesis certificate, evidence thresholds, stability/sensitivity actions, public-vs-patient verification distinction, and PostgreSQL persistence. Rank next actions by demonstrated portfolio/objective impact under declared perturbations; label this bounded decision sensitivity, not Bayesian experimental design. Report candidate provider, per-cluster support, uncovered-support score, stability, search reduction, retained evidence, runtime, and provenance without efficacy or unproven-superiority claims.
- [ ] Verify: unit/end-to-end/public-or-synthetic benchmark and migration/restore tests; report negative results.
- [ ] Docs/commit: add examples only from real outputs; `feat(evidence): certify portfolio hypothesis`.

## Milestone 7 — Thin analysis API — planned

- [ ] Tests first: liveness without dependencies; readiness with PostgreSQL/provider/model checks; capabilities; validation; synchronous run creation; history/summary/certificate/artifact retrieval; pagination; safe typed errors; idempotency same-key/same-content reuse and same-key/different-content conflict.
- [ ] Implement: only the planned `/health`, `/ready`, `/api/v1/capabilities`, and `/api/v1/analysis-runs` resource routes; schemas validate/serialize, services orchestrate, repositories persist. Do not expose provider-stage execution routes or add authentication before its own accepted design.
- [ ] Verify: API/integration tests against synthetic fixtures and production-like PostgreSQL; measure full pVACseq/native/PyClone run durations before reconsidering synchronous execution.
- [ ] Docs/commit: record the actual wire/error/idempotency contract and measured timing; `feat(api): expose immutable analysis runs`.

## Release verification — planned

- [ ] `git status --short` shows only intended files; all configured format/lint/type/unit/integration/security/migration checks pass.
- [ ] Fresh synthetic run is reproducible; candidate/clonal provider, reference/VEP/model, predictor/license, solver/input/scenario/baseline provenance is complete.
- [ ] No secrets, identifiers, patient/raw data, private artifacts, or unintended generated files are tracked.
- [ ] README, walkthrough, ADRs, commands, and claims match implementation; `chore(release): prepare <verified-version>` only after versioning exists.

## Deferred delivery/platform work

- UI/auth and production observability only after the analysis certificate and thin API work; keep routes thin.
- Queue/workers only after measured duration/retry/concurrency triggers; no default Celery/Redis.
- Object storage, containers, cloud, deployment, and scale work only after documented need.

## Three-version product roadmap

- **Version 1 — this checklist:** hybrid neoantigen candidate providers, explicit clonal evidence, bounded uncertainty-aware peptide portfolios, fair baselines, portfolio stability, and decision-sensitivity-ranked validation actions.
- **Version 2 — separate future checklist after evidence gate:** modality-specific co-design with independent vaccine/TCR-T and CAR target-and-logic tracks. Do not add either biological model to Version 1 opportunistically.
- **Version 3 — separate future checklist after repeated-observation gate:** adaptive research-state updating, decision-focused experiment selection, contingency hypotheses, and evidence-gated evolutionary stress tests.

The validation levels, falsification criteria, data needs, and collaboration requirements are in [`research-and-validation.md`](research-and-validation.md). Raw sequencing, small molecules, clinical recommendation, safety/efficacy claims, and patient-evolution prediction remain deferred.
