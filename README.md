# OncoSyn

**OncoSyn is an evolution-aware immunotherapy research co-design engine.** It asks: given heterogeneous tumour evidence and several plausible designs, which targets should researchers combine, which modality-specific design should they investigate, how could it fail, what evidence is missing, and which experiment is most likely to change the decision?

OncoSyn aims to reduce the search and prioritization space between tumour molecular profiling and downstream experimental therapeutic development. It produces research hypotheses—not treatment recommendations, efficacy predictions, or validated therapeutics. [`oncosyn_problem_statement.md`](oncosyn_problem_statement.md) is the active brief; **OncoSyn** is the only active name.

## Repository state

This is a documentation-only scaffold: no application, manifest, API, database, tests, CI, containers, or deployment exist. All technology and modules below are planned contracts. Follow the [`backend build checklist`](docs/backend-build-checklist.md) and [`codebase walkthrough`](docs/codebase-walkthrough.md).

## Version 1 boundary

Version 1 is the implementable MVP: one coherent **heterogeneity-aware neoantigen target-portfolio** track.

```text
processed tumour profile + HLA
  -> explicit candidate-provider eligibility
     -> Tier B VEP-annotated VCF -> pVACseq broad candidate evidence
     -> processed protein context/supplied peptides -> native enumeration + MHCflurry
  -> clonal-inference provider (PyClone-VI initially)
  -> uncertain clone assignments/prevalence
  -> candidate evidence -> candidate -> mutation -> clone mapping
  -> OncoSyn optimizer + declared uncertainty/adverse scenarios
  -> same-input baselines + stability + decision-sensitivity ranking
  -> evidence, per-cluster support, uncertainty, and hypothesis certificate
```

The output is a bounded `K`-peptide portfolio, per-cluster candidate support, a scenario-weighted uncovered clonal-support score, candidate evidence, robustness under named scenarios, provenance, portfolio stability, baseline comparisons, assumptions, failure modes, and ranked evidence checks or experiments. An experiment ranking is a decision-value hypothesis, not proof that the assay will validate a target. PyClone cluster prevalences are not additive tumour mass or ancestry. A promoted candidate is a **computational hit for downstream investigation**, not a biologically or clinically validated hit.

### Input tiers

OncoSyn consumes processed results, never raw sequencing files.

| Tier | Evidence | Interpretation |
| --- | --- | --- |
| **A — targeted-panel** | Mutations; actual reference/alternate counts where available (or reported VAF/depth for adequacy assessment); copy number; purity; sample metadata; separate HLA; processed mutant protein context or supplied peptides; optional expression. | Native candidate provider only when sequence/peptide evidence is adequate. Run clonal inference only when its separate exact evidence is adequate. Label sparse panels as limited. |
| **B — research-rich** | WES/WGS-derived processed calls, including an explicitly labelled normalized VEP-annotated-VCF subtype; matched normal, allele-specific copy number, RNA, HLA, and optional multiregion/longitudinal samples. | pVACseq may generate the broad candidate universe. A processed VCF is not raw sequencing, but its reference/VEP/source versions and retention remain explicit. |

PyClone-family providers require sequencing-derived reference/alternate counts and copy-number/purity evidence; never manufacture pseudo-counts from VAF. Insufficient evidence is rejected, or explicitly validated precomputed clone evidence is selected through a separate provider—never silently substituted.

## Scope and modality rules

- **Version 1:** mutant-peptide portfolios for neoantigen vaccine/T-cell research. It does not design a vaccine construct or TCR.
- **Version 2, Track A:** vaccine and TCR-T co-design hypotheses. TCR-T may reuse peptide/HLA evidence but requires recognition, cross-reactivity, processing, feasibility, safety-evidence, and developability constraints.
- **Version 2, Track B:** CAR target-and-logic hypotheses over accessible surface antigens, quantitative density, tumour and normal-cell states, logic feasibility, heterogeneity, and antigen-loss/downregulation scenarios. It is not the peptide/HLA workflow and cannot establish safety.
- **Version 3:** adaptive research-state updating, experiment selection, contingency hypotheses, and evidence-gated evolutionary stress tests. It does not predict patient tumour evolution.
- **Deferred:** small-molecule targets, raw sequencing/variant calling, HLA typing, purity/copy-number estimation, definitive phylogeny, and clinical outcome prediction.

See [ADR 0007](docs/architecture/0007-immunotherapy-modality-boundary.md), [ADR 0009](docs/architecture/0009-three-version-research-architecture.md), [ADR 0010](docs/architecture/0010-car-design-research-boundary.md), and [ADR 0011](docs/architecture/0011-adaptive-experiment-and-evolution-gate.md).

## Principles and non-goals

- **Portfolio over ranking:** optimize the combination, not independent scores.
- **Research differentiation, not assumed novelty:** prior work including NeoAgDT already optimizes neoantigen vaccine composition over tumour heterogeneity. OncoSyn tests whether explicit clonal-inference uncertainty, versioned scenarios, stability analysis, and auditable robust selection improve on ranking and existing optimization baselines.
- **Design and experiment over static targets:** the long-term research unit is a modality-specific design hypothesis plus its failure envelope, missing evidence, and validation plan—not a universal candidate score.
- **Uncertainty is data:** preserve prevalence error, assignment uncertainty, predictor disagreement, missing evidence, and assumptions.
- **Traceability:** retain every source, version, transformation, threshold, exclusion, and optimizer contribution.
- **Replaceable upstream science:** clonal inference, peptide generation, predictors, and evidence sources stay behind normalized boundaries; they are not OncoSyn's proprietary algorithm.
- **Research-only:** no diagnosis, prescription, relapse-prevention, clinical efficacy, or regulatory-timeline claims.
- **Production discipline:** reproducibility, migrations, access control, observability, backup, and recovery are required before sensitive production use; speculative scale infrastructure is not.

## Durable architecture decisions

| Decision | Rationale and consequence |
| --- | --- |
| Exactly three evidence-gated versions | Keeps the current MVP implementable while making modality co-design and adaptive research explicit without presenting them as implemented. |
| One Version 1 modality track | Keeps candidate semantics and validation coherent; TCR-T and CAR-T require separate Version 2 models. |
| Generic `ClonalInferenceProvider`; PyClone-VI first adapter | Inference is upstream evidence, not novelty. PyClone-VI is not a universal default; domain/optimizer never consume provider formats, and validated precomputed evidence is an explicit alternative. |
| Explicit hybrid candidate provider | Use pVACseq `all_epitopes` for eligible Tier B VEP inputs and a native enumerator/MHCflurry path for adequate processed protein or supplied-peptide inputs. Never mix, silently fall back, or optimize pVACseq's Top Score-reduced output. |
| Bounded robust optimization as a hypothesis | Compare the same `K` and candidate set against Top-`K`, clonality-weighted, coverage-only, reproducible minimax, pVACseq ranking, and prior-art optimization where comparable. |
| Hypothesis certificate | Every result must expose selected targets, source mutations/clones, evidence, uncertainty, scenarios, baseline, assumptions, failure modes, and next check. |
| PostgreSQL system of record | Immutable concurrent analysis records require transactions/migrations; use SQLAlchemy repositories/Alembic and SQLite only for isolated tests. |
| Synchronous execution first | No runtime evidence justifies Celery/Redis/object storage/deployment; preserve service/provider boundaries for later change. |

ADRs: [data boundary](docs/architecture/0001-research-data-boundary.md), [certificate](docs/architecture/0002-hypothesis-certificate.md), [persistence](docs/architecture/0003-analysis-run-persistence.md), [execution](docs/architecture/0004-asynchronous-analysis-execution.md), [Version 1 pipeline](docs/architecture/0005-mvp-analysis-pipeline.md), [clonal inference](docs/architecture/0006-pyclone-vi-integration.md), [modality boundary](docs/architecture/0007-immunotherapy-modality-boundary.md), [candidate generation](docs/architecture/0008-neoantigen-candidate-generation.md), [three-version architecture](docs/architecture/0009-three-version-research-architecture.md), [CAR research boundary](docs/architecture/0010-car-design-research-boundary.md), and [adaptive experiment/evolution gate](docs/architecture/0011-adaptive-experiment-and-evolution-gate.md).

## Module ownership

| Module | Owns | Must not own |
| --- | --- | --- |
| Input/API | Schemas, validation, serialization, error mapping. | Biology, solver, direct SQL, or unimplemented authentication. |
| Candidate provider | Eligibility, scientific-runtime invocation, broad candidate normalization, provenance. | Clonal inference, portfolio choice, silent fallback. |
| Clonal inference | Eligibility and normalized uncertain clone evidence. | Variant calling, ancestry, selection. |
| Peptide enumeration | Mutation/sequence-to-peptide records. | HLA scoring or selection. |
| Candidate evidence | Versioned predictor/evidence normalization. | Portfolio choice or clinical interpretation. |
| Clonal mapping | Candidate -> stable mutation -> selected cluster support with reported uncertainty. | Full-posterior, additive-mass, or lineage claims. |
| Optimizer | `K`, constraints, scenarios, objectives, baselines, diagnostics. | Provider formats, persistence, lookups. |
| Certificate | Provenance, comparisons, sensitivity, next actions. | Hidden reruns or source mutation. |
| Experiment schema | Feasible assay definition, outcome semantics, cost/context, and decision linkage. | Treat an unrun experiment or predicted outcome as observed evidence. |
| Repository | Immutable persistence and transactions. | Domain decisions. |

Backend layering is `API -> service -> repository -> ORM`; schemas own API boundaries, services orchestrate, repositories persist, and domain/optimizer code stays independent of FastAPI and SQLAlchemy.

## Planned stack and operating contracts

None is implemented: Python 3.11/`uv`; FastAPI/Pydantic/Uvicorn; SQLAlchemy async/Alembic/PostgreSQL; OR-Tools CP-SAT; pinned external pVACtools 7.1.2 and PyClone-VI 0.2.0 provider environments; stable MHCflurry 2.2.1 behind the native adapter; later React/TypeScript/Vite/Tailwind/Plotly. Versions are research snapshots from 2026-08-18 and must be reverified and locked during implementation. Add cloud, containers, queues, caches, or object storage only after documented measured triggers.

- An immutable analysis run owns its input snapshot/tier, providers, candidates, scenarios, baselines, portfolio, certificate, and provenance. Changes create a new run.
- Do not store raw sequencing. Patient/customer data is blocked until ADR 0001 governance and security controls are verified.
- Never commit/log secrets, identifiers, genomic records, tokens, or `.env` values. Tests use deterministic synthetic/public non-sensitive fixtures and isolated settings.
- Use explicit versioned schemas and typed errors for invalid/insufficient input, unavailable providers, unsupported prediction, mapping failure, partial/infeasible optimization, and internal failure.
- Record input/source, reference/VEP data where applicable, candidate and clonal providers, predictor/model, optimizer/solver, scenario, baseline, schema, settings, licenses, and software versions.

The smallest planned API is `GET /health`, `GET /ready`, `GET /api/v1/capabilities`, `POST /api/v1/analysis-runs/validate`, `POST /api/v1/analysis-runs`, `GET /api/v1/analysis-runs`, `GET /api/v1/analysis-runs/{run_id}`, `GET /api/v1/analysis-runs/{run_id}/certificate`, and `GET /api/v1/analysis-runs/{run_id}/artifacts/{artifact_type}`. Provider stages are not public endpoints. Run creation starts synchronously. Reusing an idempotency key with the same request returns the existing run; reusing it with different content is a conflict. The exact HTTP schema is implemented and tested with the API milestone, not implied by these planned routes.

## Validation and three-version roadmap

MVP metrics are computational: baseline-relative per-cluster support/robustness, portfolio stability, search-space reduction, retained evidence, profile-to-certificate time, and provenance completeness. They are not efficacy measurements. The validation ladder, prior-art challenge, open research questions, and potential—not current—collaboration asks are owned by [`docs/research-and-validation.md`](docs/research-and-validation.md).

1. **Version 1 — Heterogeneity-Aware Neoantigen Target Portfolio Engine:** observed-state portfolios, propagated uncertainty, stability analysis, fair baselines, and decision-focused validation prioritization.
2. **Version 2 — Modality-Specific Immunotherapy Co-Design Engine:** separate vaccine/TCR-T and CAR target-and-logic tracks sharing evidence, provenance, uncertainty, optimization primitives, and experiment schemas rather than one biological model.
3. **Version 3 — Adaptive Immunotherapy R&D Engine:** evidence-gated stress tests, experiment selection, observation-driven updating, contingency hypotheses, and carefully bounded evolutionary scenarios.

Progression is evidence-gated rather than calendar-based. Version 2 requires modality-specific retrospective and experimental validation. Version 3 requires repeated observations capable of calibrating update and experiment-selection models; evolutionary transition or control models remain prohibited until identifiable. Longitudinal samples, immunopeptidomics, and experiments may test associations between model outputs and research endpoints, but do not establish clinical efficacy.

Personalization is conditioned on observed mutations, inferred heterogeneity, patient HLA, available expression, and—only in later validated versions—longitudinal response evidence.

## Workflow and documentation contract

No setup, test, migration, or release commands exist; do not invent them. Currently verified inspection commands are `git status --short`, `git log --oneline -8`, and `rg --files`. Add exact commands with tooling. See [`ENGINEERING_PLAYBOOK.md`](ENGINEERING_PLAYBOOK.md).

Future changes must preserve these decisions or deliberately revise the README, affected ADRs, checklist, and walkthrough in the same change. Never report planned architecture as implemented.
