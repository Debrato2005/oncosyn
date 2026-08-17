# Backend MVP Build Checklist

Legend: **implemented** = repository-verified; **in progress** = evidence exists but acceptance is incomplete; **planned** = agreed work; **deferred** = intentionally later. The repository is documentation-only, so every implementation item is **planned**.

For every milestone: write the listed deterministic synthetic tests first; implement only that boundary; run the real configured format/lint/type/test/migration commands; update README, walkthrough, affected ADR, and this checklist; then use the focused commit suggestion. Commands must be added only when tooling exists.

## Milestone 0 — Foundation — planned

- [ ] Tests first: one synthetic schema rejection test.
- [ ] Implement: Python 3.11/`uv`/`pyproject.toml`, pinned dependencies, source/test layout, pytest and selected formatter/linter/type checker; synchronous service boundary only.
- [ ] Verify: record and run exact install/static/test commands.
- [ ] Docs/commit: update repository map and commands; `chore(platform): establish reproducible backend tooling`.

## Milestone 1 — Input and domain contracts — planned

- [ ] Tests first: Tier A/targeted and Tier B/research-rich acceptance; HLA, sample/mutation identity, actual counts, VAF/depth assessment, CN/purity, sequence context/supplied peptides, optional RNA, multiregion/longitudinal metadata; malformed, missing, and immutable snapshot cases.
- [ ] Implement: versioned schemas; input-tier and observed-scope labels; clonal-provider eligibility; explicit limited-panel status; PostgreSQL snapshot through repository/migration. No raw data, calling, pseudo-counts, HLA typing, CN/purity estimation, or tree inference.
- [ ] Verify: configured schema/static/migration tests.
- [ ] Docs/commit: document actual schema/errors; `feat(contract): define tiered analysis input`.

## Milestone 2 — Variant and clonal inference — planned

- [ ] Tests first: generic provider contract; PyClone-VI serialization/parser; every required actual count/CN/tumour-content field; matrix completeness; VAF-only rejection; sparse-panel limitation; unavailable/mis-versioned provider; validated precomputed evidence; uncertainty fields; no fallback/ancestry.
- [ ] Implement: normalized variant model and `ClonalInferenceProvider`; explicit `PyCloneVIProvider`, `PrecomputedClonalEvidenceProvider`, and labelled deterministic fixture provider. Pin real PyClone-VI externally if runtime compatibility requires it.
- [ ] Verify: unit/golden/fixture contracts; separately marked offline real-provider test when provisioned. Benchmark targeted-panel adequacy rather than inventing thresholds.
- [ ] Docs/commit: record actual version/runtime/contract; `feat(clonal-inference): add provider boundary`.

## Milestone 3 — Peptide enumeration and candidate scoring — planned

- [ ] Tests first: deterministic enumeration/supplied-peptide validation; source mutation; MHCflurry normalization/version; unsupported alleles/inputs; missing evidence; predictor disagreement representation.
- [ ] Implement: separate enumerator and predictor interfaces; MHCflurry first; replaceable NetMHCpan/pVACtools-derived providers. Candidate evidence may include presentation/binding, expression, clonality, tumour specificity, processing, immunogenicity, disagreement, and uncertainty only when sourced. No proprietary MHC model.
- [ ] Verify: unit and offline fixture tests.
- [ ] Docs/commit: record dependencies/licensing/mappings; `feat(candidates): enumerate and score candidates`.

## Milestone 4 — Candidate-to-clone mapping — planned

- [ ] Tests first: candidate -> mutation -> one/many inferred clones, prevalence error, assignment posterior, precomputed evidence, absent relationships, limited ascertainment, no invented tree.
- [ ] Implement: uncertainty-bearing mapping/coverage matrix; unsupported relationships never become coverage.
- [ ] Verify: mapping tests and synthetic pipeline through this stage.
- [ ] Docs/commit: document representation/failures; `feat(clonal-mapping): preserve mapping uncertainty`.

## Milestone 5 — Portfolio optimizer and baselines — planned

- [ ] Tests first: overlap, `K`, uncovered clones, ties, partial/infeasible results, uncertainty scenarios, deterministic diagnostics; same-set/`K` score Top-K, clonality-weighted, coverage-only, reproducible escape/minimax, and OncoSyn comparisons.
- [ ] Implement: separate candidate quality and clone coverage; bounded multi-objective selection over explicit scenarios; modelled coverage/residual uncovered mass and robustness—never clinical escape probability.
- [ ] Verify: solver/unit/end-to-end synthetic cases including OncoSyn-inferior/equivalent outcomes.
- [ ] Docs/commit: publish actual objective/constraints/tie rules; `feat(optimizer): select escape-aware portfolios`.

## Milestone 6 — Evidence, uncertainty, and certificate — planned

- [ ] Tests first: every certificate field in ADR 0002; complete provenance/versioning; missing sources; sensitivity change/no-change; ranked next action; all baselines; computational-hit label; immutable persistence; safe partial/failure status.
- [ ] Implement: falsifiable hypothesis certificate, evidence thresholds, sensitivity/actions, public-vs-patient verification distinction, PostgreSQL persistence. Report search reduction, retained high-confidence evidence, processing time, source support, coverage, and robustness without efficacy claims.
- [ ] Verify: unit/end-to-end/public-or-synthetic benchmark and migration/restore tests; report negative results.
- [ ] Docs/commit: add examples only from real outputs; `feat(evidence): certify portfolio hypothesis`.

## Release verification — planned

- [ ] `git status --short` shows only intended files; all configured format/lint/type/unit/integration/security/migration checks pass.
- [ ] Fresh synthetic run is reproducible; provider/predictor/solver/input/scenario/baseline provenance is complete.
- [ ] No secrets, identifiers, patient/raw data, private artifacts, or unintended generated files are tracked.
- [ ] README, walkthrough, ADRs, commands, and claims match implementation; `chore(release): prepare <verified-version>` only after versioning exists.

## Deferred delivery/platform work

- API/UI/auth/observability only after the analysis certificate works; keep routes thin.
- Queue/workers only after measured duration/retry/concurrency triggers; no default Celery/Redis.
- Object storage, containers, cloud, deployment, and scale work only after documented need.

## Future product phases

MVP neoantigen/T-cell portfolio; V1 robust clonal uncertainty/targeted-panel benchmarks; V2 richer presentation/immunogenicity; V3 TCR-T; V4 separate CAR-T; V5 preclinical feedback; V6 longitudinal/ctDNA; V7 closed-loop hypothesis engine. The evidence gates and open questions are in [`research-and-validation.md`](research-and-validation.md). Raw sequencing, small molecules, clinical recommendation, and efficacy claims remain deferred.
