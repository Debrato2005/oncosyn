# OncoSyn Codebase Walkthrough

This living guide describes verified contents. Planned architecture is explicitly labelled and owned by [`README.md`](../README.md).

## Repository map

| Path | Responsibility | State |
| --- | --- | --- |
| `.gitignore` | Ignores Python/build/cache/editor/local-secret artifacts. | Initial tracked configuration; selects no runtime. |
| `AGENTS.md` | Compact project-specific operating rules for Codex. | Documentation. |
| `README.md` | Engineering source of truth: scope, contracts, stack, roadmap. | Documentation. |
| `oncosyn_problem_statement.md` | Active product brief. | Documentation. |
| `ENGINEERING_PLAYBOOK.md` | Workflow, review, release, secrets, operations. | Documentation. |
| `docs/backend-build-checklist.md` | Ordered, test-first MVP milestones. | Documentation; all implementation planned. |
| `docs/research-and-validation.md` | Computational-hit definition, baselines, validation ladder, open research questions, potential collaboration framing. | Documentation. |
| `docs/architecture/0001-research-data-boundary.md` | Sensitive data/input-tier boundary. | Accepted plan. |
| `docs/architecture/0002-hypothesis-certificate.md` | Falsifiable hypothesis certificate. | Accepted plan. |
| `docs/architecture/0003-analysis-run-persistence.md` | Immutable PostgreSQL records. | Accepted plan. |
| `docs/architecture/0004-asynchronous-analysis-execution.md` | Synchronous-first decision and queue triggers. | Accepted plan; async deferred. |
| `docs/architecture/0005-mvp-analysis-pipeline.md` | Stage handoffs and failures. | Accepted plan. |
| `docs/architecture/0006-pyclone-vi-integration.md` | Generic clonal provider, exact PyClone-VI adapter, precomputed path, uncertainty. | Accepted plan. |
| `docs/architecture/0007-immunotherapy-modality-boundary.md` | Vaccine/TCR peptide-HLA versus future CAR surface-antigen boundary. | Accepted plan. |
| `.git/` | Generated Git metadata. | Generated; not product source. |
| `.agents/`, `.codex/` | Local tool state. | Local; no product responsibility. |

No application source, manifest, dependency lock, tests, API, schema, migrations, CI, containers, provider runtime, frontend, cache, worker, object storage, or deployment exists.

## Reading and planned execution order

Read README → active brief → relevant ADR → checklist milestone → playbook. The planned runtime flow—not current code—is:

```text
tiered processed input + HLA
  -> schema/eligibility -> immutable run snapshot
  -> normalized mutations
  -> explicit ClonalInferenceProvider
     -> PyClone-VI adapter OR validated precomputed evidence
  -> uncertain clone assignments/prevalence
  -> peptide enumeration -> MHCflurry-first candidate evidence
  -> candidate -> mutation -> clone mapping
  -> named scenarios + baseline suite + bounded optimizer
  -> sensitivity/provenance -> hypothesis certificate -> PostgreSQL
```

- **Inputs:** targeted-panel Tier A or research-rich Tier B. Sparse/insufficient evidence is rejected or limited; VAF is never converted to pseudo-counts.
- **Clonal inference:** provider formats stay inside adapters. Cluster/prevalence outputs retain error/posterior and do not establish ancestry.
- **Candidates:** enumeration and prediction are separate; alternative predictors remain replaceable. Candidate evidence is not a clinical conclusion.
- **Mapping/optimization:** mapping uncertainty feeds declared scenarios. The optimizer returns coverage/residual uncovered mass and explicit partial/infeasible results.
- **Certificate:** tests OncoSyn versus named same-input baselines and records assumptions, failure modes, sensitivity, and next validation. A computational hit is only a promoted research hypothesis.
- **Persistence/delivery:** repositories isolate PostgreSQL; future API routes are thin. Sensitive values never enter logs/errors. Async/platform layers wait for measured triggers.

The MVP modality is neoantigen/T-cell immunotherapy. TCR-T and conventional CAR-T require the later distinct constraints in ADR 0007; small-molecule portfolios are deferred.

## Current file behaviour

`.gitignore` protects common generated/local files but installs and runs nothing. Markdown files define plans and constraints; they provide no executable route, dependency, schema, failure handling, or test result. `.git/` is generated; local tool directories and future caches/environments/build artifacts must not be treated as source.

## Maintenance rule

Update this guide in the same change whenever code, tests, configuration, migrations, Docker, CI, infrastructure, or frontend files change. Add real paths in execution order with dependencies, inputs, outputs, control flow, failure behaviour, tests, generated-file labels, and exact commands. Remove or revise planned language when implementation differs.
