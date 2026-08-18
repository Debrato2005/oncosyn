# OncoSyn — Codex Instructions

## Project truth
- OncoSyn is an evolution-aware immunotherapy research co-design engine, not clinical decision support. Never claim efficacy, diagnosis, treatment benefit, relapse prevention, safety, patient-evolution prediction, validated hits, partnerships, or regulatory timelines.
- Planned architecture is not implemented until verified. Never invent code, routes, dependencies, commands, schemas, test results, or infrastructure.
- `oncosyn_problem_statement.md` is the active product brief; **OncoSyn** is the active name.
- Keep `README.md`, `docs/codebase-walkthrough.md`, ADRs, and the build checklist consistent with actual implementation.

## MVP boundary
- Version 1 is the **Heterogeneity-Aware Neoantigen Target Portfolio Engine**. Version 2 has separate vaccine/TCR-T and CAR target-and-logic tracks. Version 3 is the Adaptive Immunotherapy R&D Engine. Do not reintroduce additional numbered roadmap stages.
- TCR-T needs recognition, cross-reactivity, processing, feasibility, safety-evidence, and developability constraints. Conventional CAR-T needs a separate accessible surface-antigen, quantitative-density, tumour/normal-state, logic-feasibility, heterogeneity, and antigen-loss model. Drug targets are deferred.
- Accept processed results in two labelled tiers: targeted-panel and research-rich. Never imply a sparse targeted panel captures complete heterogeneity.
- `ClonalInferenceProvider` is generic; PyClone-VI is the first adapter, not a universal scientific default. It requires actual `ref_counts`, `alt_counts`, `major_cn`, `minor_cn`, `normal_cn`, explicit `tumour_content`, and adequate sample/mutation evidence. VAF/depth may assess eligibility but must not become pseudo-counts. Reject inadequate inference input or explicitly select validated precomputed clone evidence; never silently fall back. Test provider/configuration/seed sensitivity when it affects portfolio decisions.
- Do not ingest raw sequencing, perform variant calling/HLA typing, reconstruct clone trees, or invent phylogenies in the MVP.
- Keep peptide enumeration, prediction/scoring, clonal inference, clonal mapping, portfolio optimization, baselines, and certificate assembly separate.
- Candidate generation uses an explicit `NeoantigenCandidateProvider`: pVACseq `all_epitopes` for eligible Tier B VEP inputs or native enumeration/MHCflurry for adequate processed protein/supplied-peptide inputs. Never consume pVACseq Top Score-reduced output as the optimization universe, mix providers, or silently fall back.
- Preserve cellular-prevalence standard error and the reported selected-cluster assignment probability through candidate-to-mutation-to-clone mapping; do not invent a full posterior or ancestry.
- Predictor outputs are evidence, not clinical truth.
- Do not sum cluster CCFs as tumour mass. Report per-cluster support and explicitly defined model scores under versioned assumptions/scenarios, never clinical escape probability.
- A “computational hit” is promoted for downstream investigation only. Compare on the same candidates/`K` against Top-K, clonality-weighted, coverage-only, reproducible minimax, pVACseq ranking, and prior-art optimization where comparable.
- NeoAgDT and other prior work already optimize bounded neoantigen vaccine composition over heterogeneity. Treat OncoSyn's uncertainty-aware robust-selection differentiation as a hypothesis requiring benchmarks, not established novelty.
- A ranked experiment is a decision-value hypothesis, not observed evidence. Version 1 uses bounded sensitivity/re-solving; call a method Bayesian experimental design only after outcome likelihoods are specified and calibrated.
- Classify mechanisms as directly observed, inferred, literature-prior, hypothetical, or currently unidentifiable. Never turn speculative escape/double-bind edges into biological rules.

## Architecture
- Keep routes thin: `API → service → repository → ORM`.
- API schemas own validation/serialization; services orchestrate; repositories own persistence. Domain/optimizer code must not depend on FastAPI or ORM/database sessions.
- Use PostgreSQL with explicit migrations for durable analysis runs; SQLite is only for isolated tests.
- Start synchronously. Do not add Redis, queues, object storage, microservices, or deployment infrastructure until the documented trigger is met.
- Record input tier/source, reference/VEP data when applicable, candidate and clonal providers/versions/settings, predictors/models/licenses, optimizer/solver, scenario, baseline, input-schema, and source-data versions.
- Keep modality target schemas, response assumptions, constraints, and validation endpoints separate. Share only evidence/provenance, uncertainty, optimization primitives, experiment schemas, and immutable research artifacts where their semantics truly match.

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
