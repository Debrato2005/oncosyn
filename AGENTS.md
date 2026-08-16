# OncoSyn — Codex Instructions

## Project truth
- OncoSyn is research decision support for neoantigen portfolio exploration, not clinical treatment recommendation. Never claim efficacy, diagnosis, or treatment benefit.
- Planned architecture is not implemented until verified. Never invent code, routes, dependencies, commands, schemas, test results, or infrastructure.
- `oncosyn_problem_statement.md` is the active product brief; **OncoSyn** is the active name.
- Keep `README.md`, `docs/codebase-walkthrough.md`, ADRs, and the build checklist consistent with actual implementation.

## MVP boundary
- First product track: **neoantigen portfolios only**; drug-target optimization is deferred.
- Inputs: preprocessed mutation/sample records with `ref_counts`, `alt_counts`, `major_cn`, `minor_cn`, `normal_cn`, and explicit `tumour_content`; HLA; optional expression evidence.
- PyClone-VI is the MVP clonal-inference provider. Reject missing read-count, copy-number, tumour-content, or mutation-by-selected-sample matrix evidence; never fill values with upstream defaults or silently replace the provider.
- Do not ingest raw sequencing, perform variant calling/HLA typing, reconstruct clone trees, or invent phylogenies in the MVP.
- Keep peptide enumeration, prediction/scoring, PyClone-VI inference, clonal mapping, portfolio optimization, and evidence/explanation separate.
- Preserve cellular-prevalence standard error and cluster-assignment posterior through candidate-to-mutation-to-clone mapping; a cluster is not an ancestry claim.
- Predictor outputs are evidence, not clinical truth.
- Optimize a bounded portfolio objective under explicit assumptions/scenarios. Report modelled coverage, residual clone mass, or escape risk; never call it a clinical escape probability.

## Architecture
- Keep routes thin: `API → service → repository → ORM`.
- API schemas own validation/serialization; services orchestrate; repositories own persistence. Domain/optimizer code must not depend on FastAPI or ORM/database sessions.
- Use PostgreSQL with explicit migrations for durable analysis runs; SQLite is only for isolated tests.
- Start synchronously. Do not add Redis, queues, object storage, microservices, or deployment infrastructure until the documented trigger is met.
- Record PyClone-VI provider/version/settings, predictor, optimizer, scenario, input-schema, and source-data versions for reproducibility.

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
