# OncoSyn — Codex Instructions

## Project truth
- OncoSyn is an evolution-aware immunotherapy hypothesis and target-portfolio engine, not clinical decision support. Never claim efficacy, diagnosis, treatment benefit, relapse prevention, validated hits, partnerships, or regulatory timelines.
- Planned architecture is not implemented until verified. Never invent code, routes, dependencies, commands, schemas, test results, or infrastructure.
- `oncosyn_problem_statement.md` is the active product brief; **OncoSyn** is the active name.
- Keep `README.md`, `docs/codebase-walkthrough.md`, ADRs, and the build checklist consistent with actual implementation.

## MVP boundary
- First product track: **neoantigen/T-cell immunotherapy portfolios**. TCR-T needs later modality constraints; conventional CAR-T is a separate future surface-antigen model; drug targets are deferred.
- Accept processed results in two labelled tiers: targeted-panel and research-rich. Never imply a sparse targeted panel captures complete heterogeneity.
- `ClonalInferenceProvider` is generic; PyClone-VI is first. It requires actual `ref_counts`, `alt_counts`, `major_cn`, `minor_cn`, `normal_cn`, explicit `tumour_content`, and adequate sample/mutation evidence. VAF/depth may assess eligibility but must not become pseudo-counts. Reject inadequate inference input or explicitly select validated precomputed clone evidence; never silently fall back.
- Do not ingest raw sequencing, perform variant calling/HLA typing, reconstruct clone trees, or invent phylogenies in the MVP.
- Keep peptide enumeration, prediction/scoring, clonal inference, clonal mapping, portfolio optimization, baselines, and certificate assembly separate.
- Use MHCflurry first behind the prediction/scoring boundary. Keep NetMHCpan, pVACtools-derived, and other providers replaceable; do not train a proprietary MHC model for the MVP.
- Preserve cellular-prevalence standard error and cluster-assignment posterior through candidate-to-mutation-to-clone mapping; a cluster is not an ancestry claim.
- Predictor outputs are evidence, not clinical truth.
- Optimize a bounded portfolio objective under explicit assumptions/scenarios. Report modelled coverage, residual clone mass, or escape risk; never call it a clinical escape probability.
- A “computational hit” is promoted for downstream investigation only. Compare on the same candidates/`K` against Top-K, clonality-weighted, coverage-only, and reproducible escape/minimax baselines.

## Architecture
- Keep routes thin: `API → service → repository → ORM`.
- API schemas own validation/serialization; services orchestrate; repositories own persistence. Domain/optimizer code must not depend on FastAPI or ORM/database sessions.
- Use PostgreSQL with explicit migrations for durable analysis runs; SQLite is only for isolated tests.
- Start synchronously. Do not add Redis, queues, object storage, microservices, or deployment infrastructure until the documented trigger is met.
- Record input tier/source, clonal provider/version/settings, predictor, optimizer/solver, scenario, baseline, input-schema, and source-data versions.

## Security
- Never commit, log, or expose secrets, `.env` values, tokens, identifiers, raw genomic data, or access-controlled datasets.
- Use only synthetic/public non-sensitive fixtures in tests and demos.
- Authentication/authorization is not implemented unless explicitly designed and tested.
- Use one configuration/settings boundary; tests must isolate settings and never load a developer's real `.env`.

## Testing
- Write the smallest failing test before implementation behavior.
- Prefer deterministic synthetic fixtures; mock external services.
- Run configured formatter, linter, type checker, unit/integration tests, and migration checks before claiming completion.
- Never claim a command was run unless it was actually run.

## Workflow
1. Read the relevant README/ADR/checklist before changing architecture.
2. Inspect `git status --short` and relevant files/tests.
3. Implement the smallest change satisfying the current milestone.
4. Test and verify.
5. Update affected documentation in the same change.
6. Keep changes focused and reviewable.
- Branches: `codex/<concise-scope>`.
- Commits: `type(scope): imperative summary`.
- Before completion, verify no secrets, patient data, or unintended generated artifacts are tracked.
