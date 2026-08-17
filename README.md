# OncoSyn

**OncoSyn is an evolution-aware immunotherapy hypothesis and target-portfolio engine.** It asks: given many plausible tumour-specific targets, which small combination should investigators take forward?

OncoSyn aims to reduce the search and prioritization space between tumour molecular profiling and downstream experimental therapeutic development. It produces research hypotheses—not treatment recommendations, efficacy predictions, or validated therapeutics. [`oncosyn_problem_statement.md`](oncosyn_problem_statement.md) is the active brief; **OncoSyn** is the only active name.

## Repository state

This is a documentation-only scaffold: no application, manifest, API, database, tests, CI, containers, or deployment exist. All technology and modules below are planned contracts. Follow the [`backend build checklist`](docs/backend-build-checklist.md) and [`codebase walkthrough`](docs/codebase-walkthrough.md).

## MVP boundary

The MVP is one coherent **neoantigen/T-cell immunotherapy portfolio** track:

```text
processed tumour profile + HLA
  -> variant representation
  -> clonal-inference provider (PyClone-VI initially)
  -> uncertain clone assignments/prevalence
  -> peptide enumeration
  -> presentation/binding prediction (MHCflurry initially)
  -> candidate evidence -> candidate -> mutation -> clone mapping
  -> OncoSyn optimizer + declared escape scenarios
  -> evidence, coverage, uncertainty, and modelled escape-risk certificate
```

The output is a bounded `K`-candidate portfolio, coverage, candidate evidence, modelled residual uncovered clone mass/robustness under named scenarios, provenance, sensitivity, baseline comparisons, assumptions, failure modes, and next validation action. A promoted candidate is a **computational hit for downstream investigation**, not a biologically or clinically validated hit.

### Input tiers

OncoSyn consumes processed results, never raw sequencing files.

| Tier | Evidence | Interpretation |
| --- | --- | --- |
| **A — targeted-panel** | Mutations; actual reference/alternate counts where available (or reported VAF/depth for adequacy assessment); copy number; purity; sample metadata; separate HLA; optional expression. | Run clonal inference only when the selected provider's exact evidence is adequate. Label sparse panels as limited; never imply complete heterogeneity. |
| **B — research-rich** | WES/WGS-derived processed calls, matched normal, allele-specific copy number, RNA, HLA, and optional multiregion/longitudinal samples. | Enables richer research analysis but is not clinical truth. |

PyClone-family providers require sequencing-derived reference/alternate counts and copy-number/purity evidence; never manufacture pseudo-counts from VAF. Insufficient evidence is rejected, or explicitly validated precomputed clone evidence is selected through a separate provider—never silently substituted.

## Scope and modality rules

- **Current:** neoantigen/T-cell portfolios. Peptide/HLA evidence supports vaccine and TCR-T investigation.
- **V3:** TCR-T may reuse peptide/HLA evidence but needs its own feasibility and safety constraints.
- **V4:** conventional CAR-T primarily targets cell-surface antigens and requires surface expression, tumour specificity, normal-tissue/off-tumour risk, prevalence, stability, heterogeneity, and antigen-loss evidence. It is not the peptide/HLA workflow.
- **Deferred:** small-molecule targets, raw sequencing/variant calling, HLA typing, purity/copy-number estimation, definitive phylogeny, and clinical outcome prediction.

See [ADR 0007](docs/architecture/0007-immunotherapy-modality-boundary.md).

## Principles and non-goals

- **Portfolio over ranking:** optimize the combination, not independent scores.
- **Careful novelty:** OncoSyn investigates uncertainty-aware, multi-objective therapeutic portfolio selection across inferred tumour heterogeneity and defined evolutionary/escape scenarios. It does not claim to be unprecedented.
- **Uncertainty is data:** preserve prevalence error, assignment uncertainty, predictor disagreement, missing evidence, and assumptions.
- **Traceability:** retain every source, version, transformation, threshold, exclusion, and optimizer contribution.
- **Replaceable upstream science:** clonal inference, peptide generation, predictors, and evidence sources stay behind normalized boundaries; they are not OncoSyn's proprietary algorithm.
- **Research-only:** no diagnosis, prescription, relapse-prevention, clinical efficacy, or regulatory-timeline claims.
- **Production discipline:** reproducibility, migrations, access control, observability, backup, and recovery are required before sensitive production use; speculative scale infrastructure is not.

## Durable architecture decisions

| Decision | Rationale and consequence |
| --- | --- |
| One immunotherapy MVP track | Keeps candidate semantics and validation coherent; drug targets and CAR-T require separate models. |
| Generic `ClonalInferenceProvider`; PyClone-VI first | Inference is upstream evidence, not novelty. Domain/optimizer never consume provider formats; validated precomputed evidence is an explicit alternative. |
| Enumeration separate from prediction | Peptide production and presentation estimation differ. Use MHCflurry first; keep NetMHCpan/pVACtools-derived evidence replaceable; do not train an MVP MHC model. |
| Bounded robust optimization | Compare the same `K` and candidate set against Top-`K`, clonality-weighted, coverage-only, and reproducible escape/minimax baselines. |
| Hypothesis certificate | Every result must expose selected targets, source mutations/clones, evidence, uncertainty, scenarios, baseline, assumptions, failure modes, and next check. |
| PostgreSQL system of record | Immutable concurrent analysis records require transactions/migrations; use SQLAlchemy repositories/Alembic and SQLite only for isolated tests. |
| Synchronous execution first | No runtime evidence justifies Celery/Redis/object storage/deployment; preserve service/provider boundaries for later change. |

ADRs: [data boundary](docs/architecture/0001-research-data-boundary.md), [certificate](docs/architecture/0002-hypothesis-certificate.md), [persistence](docs/architecture/0003-analysis-run-persistence.md), [execution](docs/architecture/0004-asynchronous-analysis-execution.md), [pipeline](docs/architecture/0005-mvp-analysis-pipeline.md), [clonal inference](docs/architecture/0006-pyclone-vi-integration.md), and [modality](docs/architecture/0007-immunotherapy-modality-boundary.md).

## Module ownership

| Module | Owns | Must not own |
| --- | --- | --- |
| Input/API | Schemas, auth, validation, serialization, error mapping. | Biology, solver, direct SQL. |
| Clonal inference | Eligibility and normalized uncertain clone evidence. | Variant calling, ancestry, selection. |
| Peptide enumeration | Mutation/sequence-to-peptide records. | HLA scoring or selection. |
| Candidate evidence | Versioned predictor/evidence normalization. | Portfolio choice or clinical interpretation. |
| Clonal mapping | Candidate -> mutation -> clone distribution with uncertainty. | Deterministic lineage claims. |
| Optimizer | `K`, constraints, scenarios, objectives, baselines, diagnostics. | Provider formats, persistence, lookups. |
| Certificate | Provenance, comparisons, sensitivity, next actions. | Hidden reruns or source mutation. |
| Repository | Immutable persistence and transactions. | Domain decisions. |

Backend layering is `API -> service -> repository -> ORM`; schemas own API boundaries, services orchestrate, repositories persist, and domain/optimizer code stays independent of FastAPI and SQLAlchemy.

## Planned stack and operating contracts

None is implemented: Python 3.11/`uv`; FastAPI/Pydantic/Uvicorn; SQLAlchemy async/Alembic/PostgreSQL; OR-Tools CP-SAT; pinned PyClone-VI provider; MHCflurry predictor adapter; later React/TypeScript/Vite/Tailwind/Plotly. Add cloud, containers, queues, caches, or object storage only after documented measured triggers.

- An immutable analysis run owns its input snapshot/tier, providers, candidates, scenarios, baselines, portfolio, certificate, and provenance. Changes create a new run.
- Do not store raw sequencing. Patient/customer data is blocked until ADR 0001 governance and security controls are verified.
- Never commit/log secrets, identifiers, genomic records, tokens, or `.env` values. Tests use deterministic synthetic/public non-sensitive fixtures and isolated settings.
- Use explicit versioned schemas and typed errors for invalid/insufficient input, unavailable providers, unsupported prediction, mapping failure, partial/infeasible optimization, and internal failure.
- Record input/source, provider/predictor, optimizer/solver, scenario, baseline, schema, settings, and software versions.

## Validation and roadmap

MVP metrics are computational: baseline-relative coverage/robustness, search-space reduction, retained high-confidence evidence, profile-to-certificate time, and multi-source support. They are not efficacy measurements. The validation ladder, ten open research questions, and potential—not current—collaboration asks are owned by [`docs/research-and-validation.md`](docs/research-and-validation.md).

1. **MVP:** neoantigen/T-cell portfolio and falsifiable hypothesis certificate.
2. **V1:** robust uncertainty-aware clonal analysis and targeted-panel adequacy benchmarks.
3. **V2:** richer antigen-presentation/immunogenicity evidence.
4. **V3:** TCR-T-specific constraints.
5. **V4:** separate CAR-T target/safety model.
6. **V5:** preclinical feedback loop; no animal-to-human efficacy inference.
7. **V6:** longitudinal/ctDNA tumour state.
8. **V7:** closed-loop hypothesis engine linking selection to accumulated experiments.

Long term, longitudinal response/relapse, immunopeptidomics, and experiments may test correlations between scores and outcomes. Those claims require separate evidence and validation.

Personalization is conditioned on observed mutations, inferred heterogeneity, patient HLA, available expression, and—only in later validated versions—longitudinal response evidence.

## Workflow and documentation contract

No setup, test, migration, or release commands exist; do not invent them. Currently verified inspection commands are `git status --short`, `git log --oneline -8`, and `rg --files`. Add exact commands with tooling. See [`ENGINEERING_PLAYBOOK.md`](ENGINEERING_PLAYBOOK.md).

Future changes must preserve these decisions or deliberately revise the README, affected ADRs, checklist, and walkthrough in the same change. Never report planned architecture as implemented.
