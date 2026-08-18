# OncoSyn Engineering Playbook

This playbook applies to the research-stage OncoSyn repository. It complements the durable product and architecture contracts in [`README.md`](README.md); it does not redefine them.

## Daily workflow

1. Read the relevant README decision, architecture record, and build-checklist milestone before changing code.
2. Inspect the working tree and the nearest tests before editing: `git status --short`, then `rg --files` or targeted `rg` searches.
3. Write or update the smallest failing test that expresses the desired observable behavior.
4. Implement the smallest solution that satisfies the current milestone. Keep API delivery, biological/optimization rules, and persistence separate.
5. Run the exact checks configured by the repository. Do not report a command as run if it is not configured or was not run.
6. Update `README.md`, `docs/codebase-walkthrough.md`, the applicable checklist, and any affected architecture record in the same change.

There is currently no runtime, formatter, linter, type checker, test runner, migration tool, or CI workflow. Add their real commands to the README and checklist when they are adopted.

## Branches, commits, review, and release

- Branches use the `codex/` prefix by default, followed by a concise scope, for example `codex/portfolio-optimizer`.
- Keep commits focused and independently reviewable. Suggested format: `type(scope): imperative summary`, such as `feat(optimizer): add clone coverage objective`.
- A pull request explains the user-visible/domain behavior, decisions affected, tests and commands run, data/security impact, rollback path, and documentation updates.
- Reviewers verify that evidence provenance survives the complete flow, assumptions are visible, errors are safe, and a convenience shortcut has not become biological truth.
- Reviews distinguish computational promotion from biological/clinical validation, check modality terminology, and require sources for external scientific claims. Negative or baseline-equivalent results remain reportable.
- Reviews distinguish a proposed experiment from an observation, require negative/failed/ambiguous outcomes to remain representable, and reject Bayesian/evolutionary language when its likelihood or transition model is not identifiable.
- Release only from a clean, reviewed revision with a versioned changelog/release note, passing configured checks, documented migration/rollback status, and no unresolved sensitive-data exposure.

## Environment and secrets

- Do not commit secrets, private genomic data, access-controlled data, identifiers, or `.env` files.
- Document every required environment variable in a committed example/configuration reference once variables exist, without values or credentials.
- Keep secrets in the deployment environment or approved secret store; restrict access by service and environment.
- Use synthetic or public, non-sensitive fixtures in tests and demos. A real patient-data workflow requires an explicit architecture decision covering consent, retention, encryption, access logging, and deletion.

## Migrations and rollback

- Managed PostgreSQL is the selected production system of record for immutable analysis-run records; no database schema or migration framework has been implemented yet. SQLite is allowed only for isolated local tests.
- Introduce an explicit migration mechanism with the persistence implementation. Document forward migration, backward compatibility, backup, restore, and rollback ownership in the persistence architecture record. Rehearse upgrade and restore against a production-like non-sensitive environment before every production release.
- Every migration must be reviewable, deterministic, exercised on representative non-sensitive data, and paired with a rollback or an explicit irreversible-change approval.
- Analysis artifacts are immutable snapshots. Schema or algorithm changes create versioned outputs rather than rewriting historical conclusions.

## Observability and production readiness

- Add structured logs with the first API implementation. They must record safe correlation IDs, run IDs, component/version, outcome, latency, and error category—never raw genomic inputs, HLA types, tokens, or secrets. Add metrics, traces, and queue telemetry only when those components exist.
- Record input tier/subtype/source, reference/VEP data when applicable, candidate and clonal providers, predictor/model/license, optimizer/solver, scenario, baseline, and schema versions in every analysis result.
- Before customer data is enabled, verify authentication/authorization, rate limits, audit logging, data retention/deletion, encryption, backups, dependency scanning, incident response, and operational ownership.
- A successful HTTP response is insufficient: operational checks must cover infeasible portfolios, missing/incompatible inputs, external evidence failure, stale caches, and version mismatch.

## Decision-making rules

- Prefer the simplest solution that meets the current milestone and preserves the documented domain boundaries.
- Prefer the simplest implementation that meets the current milestone: a synchronous application service and domain modules first. Defer queues, Redis, workers, object storage, authentication, deployment infrastructure, microservices, multi-region deployment, generalized therapy modes, and speculative scale-out until measured demand requires them.
- Use replaceable, versioned providers. pVACseq `all_epitopes` and native enumeration/MHCflurry implement explicit candidate branches; PyClone-VI and validated precomputed evidence implement the separate clonal boundary. Reject inadequate input/provider combinations, never mix or silently fall back, never generate pseudo-counts/default biological values, and do not train an MVP MHC model.
- Treat PyClone-VI as the first clonal adapter, not a universal scientific default. Preserve its reported uncertainty limitations and test provider/configuration/seed sensitivity when conclusions depend on them.
- Treat a threshold-passing result only as a computational hypothesis for downstream investigation. Do not make efficacy, clinical, partnership, regulatory-timeline, or cross-modality claims from scores or plans.
- Preserve the exactly-three-version roadmap. Version 1 is neoantigen portfolio selection and bounded decision sensitivity; Version 2 introduces separate vaccine/TCR-T and CAR co-design models; Version 3 introduces evidence-gated adaptive experiment/update/evolution models. Planned later-version contracts are never reported as implemented.
- If a decision materially changes a README architecture contract, update the contract and an architecture record before or with the implementation—not afterward.
