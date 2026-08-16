# Backend MVP Build Checklist

Status legend: **implemented** = verified in this repository; **in progress** = active repository evidence exists but acceptance is incomplete; **planned** = agreed future work; **deferred** = intentionally outside the MVP.

The repository currently contains documentation only. All product milestones below are **planned**; no application implementation, test command, dependency manager, database schema, migration, object store, worker, or CI workflow has been verified.

## Milestone 0 — Foundation and executable tooling — planned

- [ ] Tests first: add a minimal test runner and one schema-validation test using a synthetic, non-sensitive fixture.
- [ ] Implementation: install and pin the selected core Python tooling (Python 3.11, `uv`, `pyproject.toml`, Pydantic, and `pytest` plus the chosen formatter/linter/type checker); establish source/test layout and a synchronous application-service boundary. Do not build the React delivery layer, authentication, queues, or deployment infrastructure in this milestone. Add non-secret environment configuration only when configuration is required.
- [ ] Commands: record the actual install, formatting, lint, type, and test commands in `README.md`; run each successfully before marking complete.
- [ ] Documentation: update the repository map, setup section, and this milestone with the exact selected tooling and versions.
- [ ] Commit: `chore(platform): establish reproducible backend tooling`.

## Milestone 1 — Input and domain contracts — planned

- [ ] Tests first: validate accepted and rejected synthetic input cases for HLA alleles; mutation/sample identity; `ref_counts`, `alt_counts`, `major_cn`, `minor_cn`, `normal_cn`, and explicit `tumour_content`; mutation sequence context or supplied mutant peptides; optional expression evidence; and declared scenarios. Test migration and creation of an immutable PostgreSQL input snapshot.
- [ ] Implementation: define versioned request/domain schemas; validate at the boundary; assign an immutable analysis-run identity and persist its normalized input snapshot in PostgreSQL through a persistence adapter and explicit migrations. Require a complete mutation-by-selected-sample molecular matrix for clonal inference; do not accept raw sequencing, perform variant calling/HLA typing, or infer a clone tree.
- [ ] Commands: run the configured schema/unit-test command and static checks from Milestone 0.
- [ ] Documentation: document the real schema and error categories in the README, walkthrough, and an architecture record if the public interface is introduced.
- [ ] Commit: `feat(contract): define normalized analysis input`.

## Milestone 2 — Variant representation and PyClone-VI clonal inference — planned

- [ ] Tests first: use a deterministic synthetic mutation/sample matrix to test PyClone-VI input serialization, missing count/copy-number/tumour-content rejection, incomplete matrix rejection, result parsing, provider-unavailable behaviour, cluster-assignment posterior, and cellular-prevalence standard error.
- [ ] Implementation: define a `ClonalInferenceProvider` boundary; implement `PyCloneVIProvider` as a version-pinned external environment/CLI adapter; translate its TSV/HDF5/result details into normalized mutation-to-inferred-clone assignments with documented CCF and assignment uncertainty. Never silently replace PyClone-VI with another tool; do not treat clusters as ancestry.
- [ ] Commands: run unit and deterministic offline fixture-contract tests; add a separately marked real-PyClone-VI synthetic integration command only after the environment is provisioned and verified.
- [ ] Documentation: update ADR 0006 and the walkthrough with the actual PyClone-VI version, environment command, input/output mappings, private-artifact behaviour, and runtime verification result.
- [ ] Commit: `feat(clonal-inference): add PyClone-VI provider boundary`.

## Milestone 3 — Peptide enumeration and MHCflurry scoring — planned

- [ ] Tests first: use fixed synthetic mutation/sequence-context and HLA fixtures to test deterministic peptide enumeration or supplied-peptide validation; separately use recorded MHCflurry fixtures to test scoring normalization, unsupported input behaviour, predictor-version capture, and failure isolation.
- [ ] Implementation: add a deterministic peptide-enumeration adapter only when the input contains sufficient sequence context, otherwise validate supplied mutant-peptide records; integrate one reproducible MHCflurry peptide/HLA prediction/scoring adapter behind a separate interface; emit normalized candidate evidence.
- [ ] Commands: run unit tests plus an offline fixture-based scoring integration test; add the exact commands when implemented.
- [ ] Documentation: add the actual adapters, dependencies, data sources, licensing/operational constraints, and input/output mappings to the walkthrough.
- [ ] Commit: `feat(candidates): enumerate and score neoantigen candidates`.

## Milestone 4 — Candidate-to-clone mapping — planned

- [ ] Tests first: cover candidate-to-source-mutation identity, one-to-many inferred clone assignments, cellular-prevalence standard error, cluster-assignment posterior, absent mutation, and no invented tree.
- [ ] Implementation: map every normalized candidate through its source mutation to the PyClone-VI inferred clone assignment/prevalence estimate; preserve documented inference uncertainty as explicit mapping evidence rather than a point fact.
- [ ] Commands: run mapping unit tests and the synthetic candidate-to-certificate pipeline through this stage. Record the exact commands.
- [ ] Documentation: document the real mapping representation, uncertainty propagation, absent-mapping behaviour, and ancestry-display restriction.
- [ ] Commit: `feat(clonal-mapping): preserve candidate clone uncertainty`.

## Milestone 5 — Escape-aware portfolio optimizer — planned

- [ ] Tests first: cover a single clone, overlapping candidates, uncovered low-frequency clone, `K` limit, deterministic tie-breaking, infeasible portfolio, and PyClone-VI-derived mapping uncertainty under baseline and declared escape scenarios.
- [ ] Implementation: calculate clone coverage from inferred mapping evidence separately from candidate scoring; implement bounded selection and explicit robust escape scenarios; return expected coverage and modelled worst-case residual clone mass with solver diagnostics. Do not call the value a clinical escape probability.
- [ ] Commands: run unit tests, deterministic solver tests, and a synthetic end-to-end scenario. Record the exact commands.
- [ ] Documentation: document the actual objective, scenario assumptions, tie-breaking, infeasibility behaviour, and algorithm version in the architecture record and walkthrough.
- [ ] Commit: `feat(optimizer): select escape-aware target portfolios`.

## Milestone 6 — Evidence, uncertainty, certificate, and storage — planned

- [ ] Tests first: assert provenance retention for input, PyClone-VI, peptide, predictor, mapping, scenario, and solver versions; test missing-source behaviour, uncertainty perturbation/re-solve behaviour, selected-versus-excluded rationale, next-action ranking, immutable PostgreSQL certificate persistence, and incomplete-run status.
- [ ] Implementation: create an escape certificate for each portfolio; distinguish public evidence checks from patient-specific verification; rank actions by demonstrated decision impact, not generic evidence strength; persist immutable analysis snapshots and certificates in PostgreSQL through a persistence adapter and explicit migrations.
- [ ] Commands: run unit and end-to-end synthetic tests, including unavailable external-evidence behaviour.
- [ ] Documentation: add source/version policy, safe failure behaviour, and API/output examples based on real output, not invented payloads.
- [ ] Commit: `feat(evidence): explain portfolio uncertainty and next checks`.

## Delivery surface and observability — deferred until the analysis pipeline is verified

- [ ] Trigger: begin only after the synthetic end-to-end pipeline and certificate are verified.
- [ ] Tests first: test request validation, error mapping, authorization behaviour once introduced, safe logging, and end-to-end result serialization.
- [ ] Implementation: add a thin API/interface over domain services; expose clone coverage without inventing a phylogeny; add structured, non-sensitive observability.
- [ ] Documentation: add actual routes, authentication choices, operational commands, and deployment constraints to the walkthrough and playbook.
- [ ] Commit: `feat(api): expose auditable analysis runs`.

## Asynchronous execution — deferred until required

- [ ] Trigger: introduce this phase only after request-duration measurements, retries, or concurrent workload demonstrate the need.
- [ ] Tests first: cover job retry/idempotency and worker failure recovery for the components actually selected.
- [ ] Implementation: add a worker queue behind the synchronous application-service interface; do not move domain or optimizer rules into infrastructure code.
- [ ] Commands: document and run the actual worker integration commands only after the relevant components exist.
- [ ] Documentation: revise the asynchronous-execution ADR with the implemented queue, retry, and operational behavior.
- [ ] Commit: `feat(platform): add measured asynchronous execution`.

## Final release verification — planned

- [ ] Clean tree and reviewed changes: `git status --short` must contain only intended release files.
- [ ] Run every configured formatter, lint, type, unit, integration, and security check; record outcomes in the release/PR.
- [ ] Re-run the documented synthetic demo from a fresh environment and verify reproducible portfolio and provenance output.
- [ ] Verify no secrets, raw patient data, identifiers, or generated artifacts are tracked.
- [ ] Confirm README commands, walkthrough, architecture records, and roadmap match the release exactly.
- [ ] Commit: `chore(release): prepare <verified-version>` only after a versioning convention and release process exist.

## Future product phases — deferred

- Raw sequencing and production clonal-reconstruction pipelines.
- Secure patient-data ingestion, retention, consent, and access controls.
- Drug-target portfolio optimization and its separate resistance model.
- Clinical integration, treatment recommendation, or efficacy claims.
- Multi-region deployment, service decomposition, and speculative horizontal scale-out.
