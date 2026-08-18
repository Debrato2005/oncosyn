# ADR 0011: Adaptive experiment selection and evolution evidence gate

## Status

Accepted as a Version 3 research contract. No research-state engine, observation store, Bayesian acquisition model, escape graph, or evolution model is implemented.

## Problem

A static portfolio does not determine what evidence to collect next or how a design should change after positive, negative, failed, ambiguous, discordant, or longitudinal observations. Active learning and Bayesian experimental design already support closed-loop combination screens such as [BATCHIE](https://www.nature.com/articles/s41467-024-55287-7); tumour evolution, adaptive therapy, and evolutionary double binds also have material prior art. OncoSyn must contribute a modality-specific sequential research-decision hypothesis rather than relabel established methods.

## Selected design

Version 3 uses:

    ResearchState
      -> modality-specific DesignHypotheses
      -> evidence-labelled stress tests
      -> feasible ExperimentDefinitions
      -> selected experiment
      -> Observation
      -> explicit evidence/model update
      -> revised design or contingency hypothesis

A ResearchState is an immutable snapshot of tumour evidence, designs considered, assumptions, models, uncertainty, prior observations, and provenance. An ExperimentDefinition declares modality, biological question, assay context, admissible inputs, outcome vocabulary, feasibility/cost metadata when known, and the decision it could change. An Observation records execution context and a positive, negative, failed, ambiguous, or discordant result without overwriting the prior state.

Version 1 bounded sensitivity may rank actions by portfolio change under explicit perturbations. Version 3 may use expected value of information, expected decision improvement, regret reduction, Bayesian updating, hierarchical models, or state-space models only when their outcome likelihoods and calibration are documented. Otherwise the system must remain a bounded scenario analysis.

## Evolution and double-bind gate

Every mechanism is directly observed, inferred, literature-prior, hypothetical, or currently unidentifiable. Evolutionary scenarios are finite stress tests unless longitudinal evidence supports a calibrated transition/observation model. OncoSyn must not reconstruct an invented clone tree or claim to predict a patient's evolutionary path.

An escape-vulnerability/double-bind edge requires pressure, observed escape mechanism, consequence, measured vulnerability, tumour/model context, modality, evidence level, experimental validation, and replication status. Unsupported edges remain non-executable hypotheses.

MDP, POMDP, evolutionary-control, or patient-specific forecasting claims are prohibited until transition and observation models are identifiable, calibrated, and prospectively validated for the declared research context.

## Baselines and validation

Experiment selection is compared with fixed expert design, random selection, uncertainty sampling, generic information gain, and modality-appropriate active-learning baselines. Closed-loop claims require multiple predeclared prospective research rounds and report decision quality, calibration, experimental burden, failures, and negative outcomes. Preclinical results remain specific to their model and cannot establish human benefit.

## Failure conditions

Version 3 fails if observations cannot be normalized reproducibly, update rules are not calibrated, selected experiments do not improve research decisions over baselines, negative/ambiguous outcomes are discarded, or evolutionary models remain unidentifiable.

## Security and provenance

ADR 0001 governs sensitive data. Every state, experiment, observation, update, model, source, and decision change is immutable and provenance-bearing. A planned experiment and a predicted outcome can never be serialized as observed evidence.

## Alternatives rejected

- **Generic tumour simulator:** too detached from identifiable evidence and research decisions.
- **Information gain as the only objective:** may reduce uncertainty without changing the design decision.
- **Automatic Bayesian updating without calibrated likelihoods:** false precision.
- **Speculative escape graph:** converts plausible biology into unsupported rules.
- **Clinical adaptive-treatment policy:** outside OncoSyn's research-only scope.
