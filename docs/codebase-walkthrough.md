# OncoSyn Codebase Walkthrough

This living guide describes verified contents. Planned architecture is explicitly labelled and owned by [`README.md`](../README.md).

## Repository map

| Path | Responsibility | State |
| --- | --- | --- |
| `.gitignore` | Ignores Python/build/cache/editor/local-secret artifacts. | Initial tracked configuration; selects no runtime. |
| `AGENTS.md` | Compact project-specific operating rules for Codex. | Documentation. |
| `README.md` | Engineering source of truth: scope, contracts, stack, and three-version roadmap. | Documentation. |
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
| `docs/architecture/0008-neoantigen-candidate-generation.md` | Explicit pVACseq/native candidate providers, mutation identity, candidate handoff, runtime, licensing, and benchmarks. | Accepted plan. |
| `docs/architecture/0009-three-version-research-architecture.md` | Exactly three versions, shared research primitives, progression gates, and claim boundaries. | Accepted plan. |
| `docs/architecture/0010-car-design-research-boundary.md` | Version 2 CAR target-and-logic design variables, evidence, constraints, and non-claims. | Accepted research plan; future. |
| `docs/architecture/0011-adaptive-experiment-and-evolution-gate.md` | Version 3 research-state, experiment, observation, updating, and evolution evidence gates. | Accepted research plan; future. |
| `.git/` | Generated Git metadata. | Generated; not product source. |
| `.agents/`, `.codex/` | Local tool state. | Local; no product responsibility. |

No application source, manifest, dependency lock, tests, API, schema, migrations, CI, containers, provider runtime, frontend, cache, worker, object storage, or deployment exists.

## Reading and planned execution order

Read README → active brief → relevant ADR → checklist milestone → playbook. The planned runtime flow—not current code—is:

```text
tiered processed input + HLA
  -> schema/eligibility -> immutable run snapshot
  -> normalized mutations
  -> explicit candidate provider
     -> pVACseq all_epitopes for Tier B VEP input
     -> native enumerator + MHCflurry for processed protein/peptides
  -> explicit ClonalInferenceProvider
     -> PyClone-VI adapter OR validated precomputed evidence
  -> selected cluster assignments/prevalence + reported uncertainty
  -> candidate -> mutation -> cluster mapping
  -> named measurement/adverse scenarios + baseline suite + bounded optimizer
  -> stability + decision-sensitivity-ranked validation actions
  -> provenance -> hypothesis certificate -> PostgreSQL
```

- **Inputs:** targeted-panel Tier A or research-rich Tier B. Sparse/insufficient evidence is rejected or limited; VAF is never converted to pseudo-counts.
- **Clonal inference:** provider formats stay inside adapters. PyClone-VI's selected cluster, prevalence error, and assignment probability survive; no full posterior or ancestry is invented. PyClone-VI is the first adapter rather than a universal scientific default, so provider/configuration/seed sensitivity is part of the research contract.
- **Candidates:** pVACseq and native paths are explicit and never mixed/fallback. pVACseq contributes its broad pre-Top-Score universe; the native path keeps enumeration separate from prediction. Candidate evidence is not a clinical conclusion.
- **Mapping/optimization:** mapping uncertainty feeds declared scenarios. Cluster CCFs are not additive tumour mass; the optimizer returns per-cluster support, defined uncovered-support scores, and explicit partial/infeasible results.
- **Certificate:** tests OncoSyn versus named same-input baselines and records assumptions, failure modes, portfolio stability, sensitivity, and next validation actions ranked by declared decision impact. A computational hit and an experiment ranking are only research hypotheses.
- **Persistence/delivery:** repositories isolate PostgreSQL; future API routes are thin. Sensitive values never enter logs/errors. Async/platform layers wait for measured triggers.

Version 1 is the Heterogeneity-Aware Neoantigen Target Portfolio Engine. Its optimization unit is a unique mutant peptide with per-HLA evidence. Version 2 adds independent vaccine/TCR-T and CAR target-and-logic models; Version 3 adds observation-driven experimental updating and evidence-gated evolutionary stress tests. These future versions share infrastructure primitives but not one biological model. Small-molecule portfolios are deferred.

## Current file behaviour

`.gitignore` protects common generated/local files but installs and runs nothing. Markdown files define plans and constraints; they provide no executable route, dependency, schema, failure handling, or test result. `.git/` is generated; local tool directories and future caches/environments/build artifacts must not be treated as source.

## Maintenance rule

Update this guide in the same change whenever code, tests, configuration, migrations, Docker, CI, infrastructure, or frontend files change. Add real paths in execution order with dependencies, inputs, outputs, control flow, failure behaviour, tests, generated-file labels, and exact commands. Remove or revise planned language when implementation differs.
