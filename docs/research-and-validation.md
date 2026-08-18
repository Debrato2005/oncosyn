# Research and Validation Contract

This file owns the research thesis, prior-art boundary, validation ladder, falsification criteria, experiment-value language, data strategy, and collaboration framing. Other documents link here rather than making independent research claims.

## Research thesis and claim boundary

Existing tools rank neoantigens, integrate multi-omics evidence, optimize heterogeneous vaccine compositions, discover CAR targets and logic pairs, predict TCR interactions, model tumour evolution, and select experiments. OncoSyn therefore does not claim novelty for any one of those capabilities or for combining familiar tools.

The falsifiable programme-level thesis is:

> Existing tools usually rank or optimize targets at a fixed stage. OncoSyn investigates whether jointly modelling heterogeneous tumour-state evidence, modality-specific design constraints, decision uncertainty, failure scenarios, and experiment value can improve sequential multi-target immunotherapy research decisions.

For a declared tumour-state snapshot, evidence, constraints, and scenarios, Version 1 tests whether a bounded portfolio achieves a better declared computational objective than named baselines and whether decision-focused validation can identify a stable portfolio with fewer experiments. A **computational hit** is promoted for downstream experimental investigation only; it is not a biological or clinical hit. A recommended experiment is an experiment-selection hypothesis, not a validated assay result.

Permitted terms include computational research hypothesis, modelled scenario, design hypothesis, stress test, validation recommendation, experiment-selection hypothesis, and contingency strategy. Prohibited claims include treatment recommendation, clinical coverage, safe CAR design, efficacy, relapse prevention, patient tumour-evolution prediction, and resistance prevention.

## Prior-art boundary and baselines

[pVACtools](https://pvactools.readthedocs.io/) and [Vaxrank](https://github.com/openvax/vaxrank) provide mature neoantigen prioritization and therapy-design functions. [NeoDisc](https://www.nature.com/articles/s41587-024-02420-y) integrates proteogenomic and immunopeptidomic evidence. [NeoAgDT](https://pmc.ncbi.nlm.nih.gov/articles/PMC11076149/) already optimizes bounded vaccine compositions over simulated heterogeneous tumour populations. [SCAN-ACT](https://pubmed.ncbi.nlm.nih.gov/40814001/) discovers single and paired CAR/TCR targets from single-cell evidence. TCR specificity and structure models, Boolean CAR design methods, tumour-evolution models, and Bayesian experimental-design systems are established research areas.

Version 1 selection-only benchmarks hold eligible candidates, input, K, and constraints constant:

1. score Top-K;
2. clonality-weighted ranking;
3. cluster-support-only selection;
4. reproducible minimax;
5. pVACseq aggregate ranking mapped back to the candidate universe;
6. NeoAgDT-style MinSum/MinMax where reproducible and legally usable;
7. generic uncertainty sampling or information gain for experiment ranking, once implemented; and
8. OncoSyn portfolio and decision-focused experiment ranking.

Full-pipeline comparisons are valid only where provider branches consume equivalent source evidence. Report negative, equivalent, excluded, and non-comparable outcomes.

## Exactly three research versions

### Version 1 — Heterogeneity-Aware Neoantigen Target Portfolio Engine

**Question:** can propagated clonal/presentation uncertainty, robust portfolio selection, stability analysis, and decision-focused validation reduce experimental burden relative to static and heterogeneity-aware baselines?

**Required evidence:** deterministic synthetic fixtures; permitted retrospective neoantigen data; candidate/clonal-provider sensitivity; eventually presentation or T-cell-recognition results with negative outcomes retained.

**Failure:** the approach does not improve held-out decision utility, selection stability, or experiments-to-stable-portfolio; or gains disappear under provider/configuration sensitivity.

### Version 2 — Modality-Specific Immunotherapy Co-Design Engine

Track A covers vaccine and TCR-T design hypotheses. Track B covers CAR target-and-logic hypotheses. They share evidence, provenance, uncertainty, optimization primitives, and experiment schemas—not target identities, response models, safety assumptions, or validation endpoints.

**Question:** do quantitative modality-specific constraints change which research designs advance, and do those changes survive retrospective and experimental comparison with target-ranking or expression-only logic baselines?

**Failure:** modality constraints do not change or improve design decisions; required surface-density, normal-state, presentation, recognition, or cross-reactivity evidence is unavailable or unreliable; or designs cannot be translated into feasible experiments.

### Version 3 — Adaptive Immunotherapy R&D Engine

Version 3 represents research states, design hypotheses, evidence-gated stress tests, feasible experiments, observations, updates, and contingency hypotheses. Evolution is one optional scenario source, not the product centre.

**Question:** does structured observation-driven updating and decision-focused experiment selection improve successive design decisions over fixed, generic-active-learning, or expert-only strategies?

**Failure:** observations cannot be mapped reproducibly into updates; the closed loop does not outperform baselines; uncertainty remains uncalibrated; or evolutionary transition/observation models remain unidentifiable.

## Experiment Engine contract

An experiment has a modality, biological question, admissible inputs, assay context, possible outcomes, cost/turnaround metadata when known, evidence dependencies, and the design decision it could change. Selection should prefer expected decision improvement, expected regret reduction, or portfolio-change probability over uncertainty reduction alone.

Initial Version 1 sensitivity analysis may rank evidence checks by re-solving explicit perturbations. It must not be labelled Bayesian optimal experimental design. Expected-value-of-information or Bayesian acquisition functions enter only after outcome likelihoods are specified and calibrated. Failed, negative, ambiguous, and discordant observations remain first-class evidence.

Example:

    presentation uncertainty for peptide P7 controls the selected portfolio
      -> a feasible presentation/recognition assay could change the decision
      -> high decision-value hypothesis

    uncertainty for peptide P20 never changes any admissible portfolio
      -> low decision-value hypothesis

## Evidence and evolution classification

Every mechanism or state is labelled:

- **directly observed:** measured in the declared sample/assay context;
- **inferred:** estimated from an explicit model with provenance and uncertainty;
- **literature prior:** externally supported but not established for the declared tumour state;
- **hypothetical:** a stress-test assumption without sufficient direct evidence; or
- **currently unidentifiable:** not estimable from available data.

Escape-vulnerability or evolutionary double-bind edges require an initial pressure, observed escape mechanism, molecular/phenotypic consequence, measured vulnerability, tumour/model context, modality, evidence level, experimental validation, and replication status. No speculative edge becomes an executable biological rule. Double-bind discovery is a long-term Version 3 research opportunity.

## Validation ladder

| Level | Evidence | Claims permitted |
| --- | --- | --- |
| **L0 — mathematical correctness** | Objective, constraints, solver and serialization tests. | The implementation matches its declared mathematics. |
| **L1 — synthetic stress tests** | Deterministic perturbations, missingness, calibration and failure cases. | Behaviour under declared synthetic assumptions. |
| **L2 — retrospective public data** | Legally usable public processed datasets. | Retrospective computational performance in that dataset. |
| **L3 — retrospective experimentally validated targets** | Historical positive and negative assay endpoints. | Association with those declared experimental endpoints. |
| **L4 — collaborator-led in-vitro experiments** | Prospective presentation, recognition, density, co-culture or perturbation assays. | Prospective experimental utility in the tested context. |
| **L5 — organoid/co-culture** | Context-rich functional experiments. | Performance in the tested ex-vivo model. |
| **L6 — preclinical research** | Governed animal/preclinical studies. | Performance in that model; no human efficacy inference. |
| **L7 — prospective closed-loop research** | Multiple predeclared design–experiment–update rounds. | Prospective research-decision utility, not clinical benefit. |

Minimum validation: Version 1 computational correctness requires L2 and its experiment-reduction claim requires L4; Version 2 design-utility claims require L4 and stronger context-dependent claims may require L5/L6; Version 3 closed-loop claims require L7. Clinical claims are outside this roadmap.

## Ranked publishable hypotheses

1. **Decision-focused neoantigen validation:** portfolio-change acquisition reduces assays-to-stable-portfolio relative to ranking, uncertainty sampling, and generic information gain.
2. **Quantitative heterogeneous CAR logic co-design:** tumour/normal cell states, density uncertainty, logic feasibility, and antigen-loss scenarios improve experimental design prioritization over expression-only pair discovery.
3. **Assay-calibrated uncertainty propagation:** retaining clonal, presentation, and mapping uncertainty reduces unstable target promotion relative to point-estimate pipelines.
4. **Closed-loop design updating:** structured positive, negative, ambiguous, and discordant observations improve successive design decisions over one-shot reranking.
5. **Escape-vulnerability contingency selection:** evidence-backed escape graphs identify reproducible sequential vulnerabilities better than independent target selection. This is the highest-risk and latest hypothesis.

## Collaborators and experiments

There are no claimed partnerships. Potential collaborators may provide expert review, permitted processed data, retrospective interpretation, assay design, or experimental evaluation. Outreach must request a specific contribution and never imply endorsement or access.

| Research feature | Computational requirement | Scientific collaborator requirement | Experimental validation |
| --- | --- | --- | --- |
| Neoantigen portfolio sensitivity | Calibrated scenarios and decision analysis | Review assumptions and candidate plausibility | Presentation and T-cell recognition assays |
| Vaccine/TCR-T hypothesis | Presentation, specificity and cross-reactivity evidence | Select targets, TCRs and controls | Multimer, cytokine, killing and cross-reactivity assays |
| CAR logic co-design | Cell-state, density, logic and escape model | Confirm construct and assay feasibility | Quantitative flow, normal controls, co-culture and killing |
| Escape stress testing | Evidence-labelled mechanism graph | Judge biological context and perturbations | Selected perturbation or escape models |
| Experiment selection | Outcome, feasibility and decision models | Define feasible assays and endpoints | Prospective comparison with fixed/expert selection |
| Closed-loop updating | Observation and recalibration model | Interpret discordant outcomes | Multiple predeclared experimental rounds |

The smallest useful collaboration is a predeclared comparison on a fixed candidate/design set: OncoSyn and experts prioritize feasible assays; selected and control assays are run where resources permit; positive and negative outcomes are retained; and decision changes and experimental burden are measured.

## Data acquisition and research moat

Public inputs may include permitted genomic cohorts, immunopeptidomic datasets, single-cell tumour atlases, quantitative surface-expression studies, and experimentally annotated TCR/neoantigen collections. Every source needs license, cohort, assay, version, ascertainment, and missingness provenance. Public expression is contextual evidence, not patient-specific surface density or CAR safety.

Collaborator-generated data should prioritize negative results, quantitative surface-density measurements, normal-cell controls, complete assay context, tested CAR logic configurations, and repeated/post-perturbation observations. Exact sample sizes must follow endpoint-specific power and simulation studies rather than an invented global minimum.

The plausible moat is not code. It is a governed longitudinal graph connecting tumour evidence, designs considered, uncertainty at decision time, experiment selected, complete outcome, and resulting decision change—especially negative/discordant outcomes and assay-calibrated density, presentation, recognition, and escape evidence.

## Open research questions

1. What evidence makes targeted-panel clonal inference informative, and how should incomplete ascertainment constrain claims?
2. How sensitive are portfolios to clonal provider, configuration, seed, predictor, allele, threshold, and source disagreement?
3. Which presentation, expression, processing, tumour-specificity, immunogenicity, density, accessibility, shedding, and internalization evidence adds reproducible decision value?
4. Which Version 1 acquisition function best predicts portfolio-changing experiments?
5. What endpoints and controls define experimental survival for vaccine, TCR-T, and CAR design hypotheses?
6. How should negative, failed, ambiguous, and discordant experiments update evidence without assay or modality bias?
7. Which CAR antigen-density and normal-cell-state measurements are sufficiently quantitative for logic-design comparison?
8. Which escape mechanisms are directly observable, inferable, prior-only, hypothetical, or unidentifiable in each dataset?
9. When do repeated observations support calibrated Bayesian updating rather than bounded sensitivity analysis?
10. Can any escape-vulnerability relationship replicate strongly enough to support a contingency hypothesis?

## Claims needing verification

Before external use, verify and source claims about predictor performance, clonal-provider suitability, panel adequacy, antigen presentation, immunogenicity, surface density, normal-tissue exclusion, logic feasibility, cross-reactivity, evolutionary mechanisms, experiment value, comparative novelty, time/search reduction, and translation. Package availability is not licensing approval. Planned benchmarks, collaborator asks, and research hypotheses are not established facts.
