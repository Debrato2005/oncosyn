# OncoSyn Codebase Walkthrough

This is a living learning guide. It describes verified repository contents only; planned architecture is identified as planned and is documented in [`README.md`](../README.md) and [`backend-build-checklist.md`](backend-build-checklist.md).

## Repository map

| Path | Responsibility | Current state |
| --- | --- | --- |
| `.gitignore` | Ignores common Python artifacts, environments, local secrets, caches, and editor files. | Implemented in the initial commit. It does not select a Python runtime. |
| `oncosyn_problem_statement.md` | Active MVP product brief. | Untracked documentation; implementation remains planned. |
| `README.md` | Engineering source of truth: scope, durable decisions, boundaries, roadmap, and verified commands. | Implemented documentation. |
| `ENGINEERING_PLAYBOOK.md` | Team workflow and release/operational expectations. | Implemented documentation. |
| `docs/backend-build-checklist.md` | Ordered, executable backend MVP milestones. | Implemented documentation; build work is planned. |
| `docs/architecture/` | Focused architecture records for data boundaries and escape-certificate behavior. | Implemented documentation. |
| `docs/architecture/0003-analysis-run-persistence.md` | PostgreSQL persistence decision for immutable analysis-run certificates. | Implemented documentation; database code is planned. |
| `docs/architecture/0004-asynchronous-analysis-execution.md` | Deferred API/worker/queue design, retained for the point at which measured workload requires it. | Implemented documentation; no worker is planned for the initial build. |
| `docs/architecture/0005-mvp-analysis-pipeline.md` | The MVP handoff contract for candidate generation, clonal mapping, optimization, provenance, and uncertainty. | Implemented documentation; no pipeline code exists. |
| `docs/architecture/0006-pyclone-vi-integration.md` | Exact PyClone-VI provider boundary, input/output contract, failure behaviour, and synthetic-fixture strategy. | Implemented documentation; no provider/runtime exists. |
| `.git/` | Git metadata and hooks. | Generated/local Git internals; do not edit as product source. |
| `.agents/`, `.codex/` | Local agent/application workspace directories. | Tooling directories; no repository product responsibility is asserted. |

## Execution order today

There is no executable application, package manifest, test suite, PyClone-VI environment, CI workflow, container configuration, database schema, migration configuration, worker, object store, cache, or deployment configuration. The first build is synchronous; PostgreSQL persistence and PyClone-VI clonal inference are selected but unimplemented, while asynchronous infrastructure is deferred until validated need. Consequently, no runtime control flow, API route, persistence flow, or automated failure behavior exists to explain.

The present order of work is documentary and architectural:

1. `README.md` defines what OncoSyn is and what it must not claim.
2. The architecture records define the sensitive research-data boundary and the intended escape-certificate contract.
3. `docs/backend-build-checklist.md` turns those constraints into verifiable implementation milestones and identifies the measurable triggers for deferred infrastructure.
4. `ENGINEERING_PLAYBOOK.md` governs how future changes are tested, reviewed, released, and documented.

## Existing configuration

### `.gitignore`

This file is the only file from the initial commit. It ignores compiled Python files, packaging outputs, virtual environments, test/coverage caches, local `.env` files, common editor state, and several tool-specific caches. Its relevant behavior is protective: local credentials and generated artifacts should not be added accidentally.

It has no runtime effect. It does not install dependencies, start a server, configure a database, select an API framework, or run tests.

## Existing product reference

### `oncosyn_problem_statement.md`

This untracked file is the aligned **OncoSyn MVP problem statement**. It defines the intended high-level flow—preprocessed tumour inputs, PyClone-VI clonal inference, peptide enumeration or supplied-peptide validation, prediction/scoring, inferred clone mapping, portfolio optimization, evidence, and an explanatory output.

No implementation detail in that brief is treated as already built. In particular, listed tools, datasets, API ideas, and example outputs are proposals until added to the repository with configuration and tests.

## Planned execution model

The following is a planned control flow, not current code:

```text
validated analysis input
  -> synchronous application service
  -> variant/mutation representation
  -> PyClone-VI provider -> inferred clone distribution, CCF, and uncertainty
  -> peptide enumeration or supplied-peptide validation
  -> peptide/HLA prediction and scoring + normalized candidate evidence
  -> candidate-to-mutation-to-inferred-clone coverage mapping
  -> baseline and declared uncertainty/escape scenarios
  -> bounded robust portfolio optimizer
  -> provenance, sensitivity analysis, and versioned certificate
  -> PostgreSQL analysis record -> API/UI result
```

- The input boundary will reject malformed or incompatible data before domain work begins.
- The application service coordinates the initial synchronous workflow; it is the seam at which persistence or background execution can be added later.
- Peptide enumeration, prediction/scoring, clonal mapping, and portfolio selection will remain separate so predictor failures cannot corrupt solver rules and MHCflurry is not treated as a peptide generator.
- The PyClone-VI provider will run only on a complete normalized read-count, copy-number, and explicit tumour-content matrix; it will return typed failure rather than fabricate missing values or use a different inference algorithm.
- Clonal mapping will join candidates to PyClone-VI's uncertainty-bearing clone assignments and prevalence estimates. Cluster assignments are not lineage; the system will not invent a phylogeny.
- The optimizer will minimize a declared scenario's residual uncovered clone mass, return an explicit infeasibility result when constraints cannot be met, and not present this measure as a clinical escape probability.
- Explanations will retain input, source, predictor, clone-mapping, optimizer, and scenario versions; sensitivity re-solves will rank next evidence actions by demonstrated decision impact.
- Any future delivery layer will map domain failures to safe external errors and must not leak sensitive inputs or stack traces.

Detailed decisions and rejected alternatives are in the architecture records. Tests for each stage are specified before implementation in the build checklist.

## Generated and local files

- `.git/` is generated Git state. Do not document or modify its internal files as application behavior.
- Python caches, virtual environments, coverage output, local environment files, and similar generated artifacts are intentionally covered by `.gitignore` when they are introduced.

## Maintenance rule

Update this walkthrough in the same change whenever code, tests, configuration, migrations, Docker files, CI workflows, infrastructure, or frontend files change. Add real paths, dependencies, inputs, outputs, control flow, failure behavior, tests, and exact commands as they are introduced. Do not leave planned statements behind once implementation diverges from them.
