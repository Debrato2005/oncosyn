# OncoSyn

OncoSyn is a research-stage portfolio engine for personalised cancer neoantigen exploration. Given tumour information, it is intended to generate candidate neoantigens, select a bounded team of targets that covers meaningful tumour subclones while accounting for plausible escape routes, and explain the selection with evidence provenance and uncertainty.

[`oncosyn_problem_statement.md`](oncosyn_problem_statement.md) is the active product brief. **OncoSyn is the active project name.**

## Current repository state

This repository is an initial scaffold. As of the initial commit, it contains no application source, dependency manifest, runtime configuration, test suite, CI workflow, container configuration, database schema, or migration tooling. The only tracked project file before this documentation was `.gitignore`; its Python-oriented ignores are not evidence of a selected runtime or framework.

The product is therefore a documented target, not an implemented system. The architecture is production-oriented from the start, even though no runtime infrastructure exists yet. See [`docs/backend-build-checklist.md`](docs/backend-build-checklist.md) for the ordered build plan and [`docs/codebase-walkthrough.md`](docs/codebase-walkthrough.md) for the live repository map.

## Vision and intended scope

The MVP flow is:

```text
tumour information -> candidate generation -> escape-aware portfolio selection -> explanation
```

Its intended inputs are preprocessed tumour molecular records with mutation/sample read counts, allele-specific copy-number and tumour-content evidence; HLA type; and optional expression evidence. PyClone-VI infers clone/subclone assignments, prevalence/CCF, and uncertainty before peptide selection. Intended outputs are a compact target portfolio, clone-coverage and modelled escape-risk summaries, candidate-level evidence, and the next evidence action most likely to reduce a material uncertainty.

OncoSyn is research decision support for investigation. It must not claim that a portfolio is clinically effective, prescribe treatment, or replace expert and experimental validation.

## Principles and non-goals

### Principles

- **Portfolio over ranking:** optimize the selected set collectively; do not merely return the highest individually scored candidates.
- **Traceability:** every output must retain the evidence inputs, source/version, assumptions, and optimizer contribution that produced it.
- **Uncertainty is an output:** preserve uncertainty in clonal assignment and candidate evidence; do not silently turn uncertain estimates into facts.
- **Biology-aware constraints:** model escape assumptions explicitly and distinguish them from observed tumour evidence.
- **Production discipline from the start:** reproducibility, access control, observability, migrations, and recoverability are product requirements, not demo polish.
- **Privacy by design:** treat genomic and HLA data as sensitive research data even when the MVP uses synthetic or public examples.

### Explicit non-goals for the MVP

- Clinical treatment recommendation, efficacy prediction, diagnosis, or regulatory-grade decision support.
- Raw sequencing ingestion, variant calling, HLA typing, purity/copy-number estimation, or phylogeny reconstruction. The MVP performs clonal inference only from sufficient preprocessed PyClone-VI inputs.
- A universal model for drug-target and neoantigen portfolios. The first product track is neoantigen vaccines; drug-target resistance modelling is deferred.
- A claim of being the first combinatorial neoantigen optimizer. The product differentiator is the evidence-traceable, escape-aware decision workflow.
- A fabricated phylogenetic tree. A clone tree is only displayed when a compatible reconstruction is supplied; otherwise show clone coverage without parent-child claims.

## Architecture decisions

| Decision | Rationale | Consequence |
| --- | --- | --- |
| Start with a neoantigen-vaccine portfolio, not dual therapy modes. | HLA-specific presentation and immune escape give one coherent MVP model. | Drug-target mode requires a separate design record before implementation. |
| Include PyClone-VI clonal inference as an upstream MVP component. | Portfolio selection needs clone/prevalence estimates, but clonal inference is not OncoSyn's novel algorithm. | Require a validated read-count, allele-specific copy-number, and tumour-content input contract; never fabricate missing values. |
| Keep candidate generation, clonal evidence, optimization, and explanation as separate modules. | Existing predictors and clonal tools can change independently of OncoSyn's selection logic. | Modules exchange versioned, normalized data; no module reaches into another's persistence. |
| Use a bounded robust portfolio objective rather than top-K ranking. | The project objective is clone coverage under plausible escape scenarios. | The optimizer must expose expected and worst-case results, assumptions, and alternatives. |
| Make evidence provenance and uncertainty first-class inputs. | A biologically plausible score is not proof; uncertain data can change a selected portfolio. | Every output needs source/version, confidence or interval, and an audit trail. |
| Separate public evidence from patient-specific verification. | Public datasets can calibrate plausibility but cannot confirm a particular patient's expression or HLA loss. | The system labels public lookups and patient-specific checks separately. |
| Start with de-identified, synthetic, or publicly usable cases while production controls are built. | The repository has no security or governance implementation today. | Customer/patient-derived data is blocked until the controls in ADR 0001 are verified. |
| Use managed PostgreSQL as the system of record. | The product needs concurrent users, durable backups, transactional integrity, and a clear managed-service operating model. | Use migrations and a persistence adapter from the start; SQLite is permitted only for isolated local tests. |
| Start analysis orchestration synchronously. | The core workflow and its actual runtime profile do not exist yet; a queue would add operational complexity before proving a need. | Keep an application-service boundary so a worker adapter can be added later without changing domain or optimizer code. |

Further rationale and constraints live in [`docs/architecture/0001-research-data-boundary.md`](docs/architecture/0001-research-data-boundary.md), [`docs/architecture/0002-escape-certificate.md`](docs/architecture/0002-escape-certificate.md), [`docs/architecture/0003-analysis-run-persistence.md`](docs/architecture/0003-analysis-run-persistence.md), [`docs/architecture/0004-asynchronous-analysis-execution.md`](docs/architecture/0004-asynchronous-analysis-execution.md), and [`docs/architecture/0006-pyclone-vi-integration.md`](docs/architecture/0006-pyclone-vi-integration.md).

The MVP stage contract is defined in [`docs/architecture/0005-mvp-analysis-pipeline.md`](docs/architecture/0005-mvp-analysis-pipeline.md). It is the source of truth for the handoff between clonal inference, candidate generation, clonal mapping, optimization, provenance, and uncertainty analysis.

## Selected technology stack

These are selected production targets; none has been implemented in the repository yet.

| Concern | Selected technology | Boundary |
| --- | --- | --- |
| Core runtime | Python 3.11, `uv`, `pyproject.toml`, committed lockfile | Python owns candidate generation, science, optimization, API, and any future workers. |
| API contract | FastAPI, Pydantic, Uvicorn | API validates/authorizes and serializes; it contains no biological or persistence rules. |
| Analysis execution | Synchronous application service initially | Domain orchestration is isolated so an asynchronous worker can be added after measured need. |
| Clonal inference | PyClone-VI in a version-pinned external provider environment | PyClone-VI TSV/CLI/HDF5 details stay behind an adapter; no substitute inference engine is silently used. |
| Peptide enumeration | A deterministic adapter or supplied, validated mutant-peptide records | Enumeration needs mutation/sequence context; it must be distinct from biological prediction. |
| Candidate prediction/scoring | MHCflurry behind a provider adapter | MHCflurry scores peptide/HLA pairs; predictor-specific formats cannot leak into domain/optimizer models. |
| Optimization | Google OR-Tools CP-SAT | Solver formulation belongs solely in the optimizer module. |
| Relational data | Managed PostgreSQL with SQLAlchemy and Alembic migrations | Repositories isolate PostgreSQL from the domain. |
| Artifacts | PostgreSQL-backed structured analysis records initially | Private object storage is deferred until permitted large artifacts or report exports are introduced. |
| Frontend | React, TypeScript, Vite, Tailwind CSS, Plotly.js | Frontend presents certificates and calls the API; it cannot calculate clinical/biological conclusions. |
| Authentication | Standards-based OIDC/OAuth provider with role-based workspace access | Identity provider is replaceable; authorization is enforced server-side. |
| Operations | Local development tooling and structured application logging | Cloud provider, containerization, queueing, and deployment infrastructure are deferred until the core workflow is verified. |

## Domain boundaries and ownership

| Area | Owns | Must not own |
| --- | --- | --- |
| Peptide enumeration | Candidate peptide records derived from adequate mutation/sequence context, or validation of supplied peptide records. | HLA prediction/scoring, clone inference, or portfolio choice. |
| Candidate prediction/scoring | Predictor output normalized into candidate evidence for peptide/HLA pairs. | Peptide enumeration, clone inference, portfolio choice, persistence policy, or user-facing clinical interpretation. |
| Clonal inference | Normalized mutation/sample read-count, allele-specific copy-number, tumour-content records and PyClone-VI-derived uncertainty-bearing clone assignments/prevalence estimates. | Variant calling, purity/copy-number invention, ancestry claims, peptide scoring, or portfolio choice. |
| Clonal evidence | Clone membership, prevalence/CCF, and uncertainty metadata. | Invented ancestry or candidate quality scores. |
| Portfolio optimization | Selection under K, coverage, robustness, and declared scenario assumptions. | Prediction-tool implementation or mutation-data parsing. |
| Evidence and explanation | Provenance, uncertainty, exclusions, and next-evidence-action ranking. | Mutating source evidence or silently changing optimization inputs. |
| Interface/API | Request validation, authentication when introduced, response serialization, and error mapping. | Biological rules, optimization formulation, or direct persistence logic. |
| Infrastructure | Configuration loading, logging, secret delivery, deployment, and backups when introduced. | Domain-specific policy. |

Routes/controllers must remain thin. Business rules belong in domain services; persistence adapters must be replaceable and cannot become the source of biological decisions.

## Data, API, security, and versioning conventions

These are contracts for future implementation; no API or datastore exists yet.

- **Data ownership:** an analysis run owns its normalized molecular input snapshot, PyClone-VI provider configuration/result, generated candidates, assumptions, selection result, and provenance. Store these as immutable, versioned records in PostgreSQL; create a new run to revise an input or algorithm configuration.
- **Persistence:** use managed PostgreSQL through a persistence/repository adapter and explicit migrations. The domain and optimizer cannot depend on database models. The initial schema stores analysis-run metadata, input snapshots, candidate evidence, scenario configuration, portfolio results, and provenance; it does not store raw sequencing files.
- **Object storage:** do not add object storage until the product accepts permitted large artifacts or generates downloadable reports. When introduced, use private encrypted S3-compatible storage; PostgreSQL will hold ownership, checksum, classification, retention, and object-reference metadata. Direct public object access will be prohibited.
- **Caching:** introduce no cache initially. If measurements demonstrate a need, cache only reproducible, version-keyed derived artifacts; cache keys must include input content identity, PyClone-VI provider/version/settings, predictor version, candidate-generation settings, optimizer version, and scenario configuration. Cached results never substitute for evidence provenance or PostgreSQL's durable analysis record.
- **Security:** do not commit secrets, sample identifiers, raw patient data, access tokens, or local environment files. Use OIDC/OAuth-based authentication, role-based workspace authorization, encrypted transport, managed secret delivery, audit events, and least-privilege service identities. Customer/patient-derived data is disabled until a documented governance, retention, deletion, and incident-response process is implemented and verified.
- **API:** when introduced, use explicit request/response schemas, stable resource identifiers, validation before domain execution, and machine-readable errors. Do not expose stack traces or sensitive input values in responses.
- **Errors:** distinguish invalid input, unavailable PyClone-VI provider, unavailable external evidence, incompatible predictor/inference output, infeasible optimization, and internal failure. Preserve a safe diagnostic correlation ID in logs once logging exists.
- **Versioning:** version the public API and every analysis artifact. Record the schema, PyClone-VI provider/version/settings, predictor, optimizer, source-dataset, and scenario versions in each result; use semantic versioning for released software.

## Roadmap

1. **Core foundation:** install and pin tooling; establish schemas, tests, and non-sensitive test fixtures.
2. **Input/domain contracts:** validate and persist immutable preprocessed variant, copy-number, tumour-content, HLA, and scenario input snapshots.
3. **Variant and clonal inference:** implement the PyClone-VI provider boundary, synthetic fixtures, strict input completeness checks, and uncertainty-bearing normalized results.
4. **Peptide enumeration and scoring:** implement supplied-peptide validation or deterministic enumeration, then one reproducible MHCflurry prediction/scoring adapter.
5. **Candidate-to-clone mapping:** map candidates through source mutations to inferred clone assignments/prevalence estimates while preserving documented assignment uncertainty.
6. **Portfolio optimizer:** implement deterministic bounded selection against baseline and declared escape scenarios, returning a structured partial/infeasible result when needed.
7. **Evidence/uncertainty certificate:** assemble provenance and sensitivity-ranked next actions, then persist the completed certificate on the immutable PostgreSQL analysis run.
8. **Hardening when required:** add delivery/API expansion, authentication, observability, object storage, background execution, and deployment only when the validated analysis pipeline needs them. The technology table is an architecture target, not a mandate to build these layers first.
9. **Deferred product:** raw sequencing pipelines, clinical treatment recommendation, drug-target mode, and phylogeny reconstruction only after their own validated models and governance decisions.

## Local development and verification

No application setup, test, migration, or build commands are configured yet. Do not invent commands or imply a runtime exists.

The currently verified repository checks are:

```bash
git status --short
git log --oneline -8
rg --files
```

When tooling is introduced, add the exact install, development, test, migration, lint, type-check, and release-verification commands here in the same change. The required implementation sequence and command placeholders are maintained in [`docs/backend-build-checklist.md`](docs/backend-build-checklist.md).

## Documentation is a contract

Future changes must preserve the decisions in this README or deliberately revise them in the same pull request, including rationale, consequences, tests, and affected architecture records. Update the codebase walkthrough whenever code, tests, configuration, migrations, Docker, CI, or frontend files change. See [`ENGINEERING_PLAYBOOK.md`](ENGINEERING_PLAYBOOK.md) for the working agreement.
