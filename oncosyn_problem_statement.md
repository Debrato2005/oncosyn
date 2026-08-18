# OncoSyn — Evolution-Aware Immunotherapy Research Co-Design

## North-star problem

Given a heterogeneous tumour state and several plausible immunotherapy designs, which targets should researchers combine, which modality-specific design should they investigate, how could it fail, what evidence is missing, and which feasible experiment is most likely to change the decision?

OncoSyn aims to reduce the search and experimental prioritization space between processed tumour evidence and downstream immunotherapy research. It produces traceable computational research hypotheses—not treatment recommendations, efficacy predictions, safe therapeutic designs, or validated targets.

## Version 1 contract

Version 1 is the implementable MVP: the **Heterogeneity-Aware Neoantigen Target Portfolio Engine**.

**Processed tumour profile → uncertain clone evidence → peptide candidates → bounded portfolio across declared scenarios → stability and decision-sensitivity analysis → hypothesis certificate.**

- **Tier A — targeted-panel:** mutations; actual read counts where available (or VAF/depth for adequacy assessment); copy number; purity; sample metadata; HLA; processed mutant protein context or supplied peptides; optional expression.
- **Tier B — research-rich:** WES/WGS-derived processed calls, including a labelled normalized VEP-annotated-VCF subtype; matched normal, allele-specific copy number, HLA, RNA, and optional multiregion or longitudinal evidence.

Raw sequencing, variant calling, HLA typing, purity/copy-number estimation, and invented phylogeny are out of scope. Limited panels are labelled incomplete views of heterogeneity. A PyClone-family provider runs only with actual allele counts, copy-number evidence, explicit tumour content, and adequate sample/mutation evidence. VAF must never become fabricated pseudo-counts. Validated precomputed clone evidence is an explicit alternative provider, never a silent fallback.

```text
processed molecular data + HLA
  -> explicit NeoantigenCandidateProvider
     -> pVACseq all_epitopes for eligible VEP input
     -> native enumeration + MHCflurry for processed protein/peptides
  -> explicit ClonalInferenceProvider
     -> PyClone-VI first adapter OR validated precomputed evidence
  -> candidate -> mutation -> selected-cluster evidence + reported uncertainty
  -> bounded optimizer over named measurement/adverse scenarios
  -> baseline comparison + stability + decision-sensitivity ranking
  -> immutable hypothesis certificate
```

PyClone-VI is the first adapter, not a universal scientific default. Its variational approximation can underestimate posterior variance, so provider, configuration, seed, and input-adequacy sensitivity must be tested when they affect selection. A cluster is neither ancestry nor an additive tumour-mass component.

## Research contribution

Neoantigen ranking, clonality-aware prioritization, vaccine-construct generation, heterogeneity-aware portfolio optimization, CAR target-pair discovery, Boolean CAR targeting, evolutionary treatment modelling, and active experimental design all have material prior art. OncoSyn does not claim novelty for integrating those capabilities.

The research thesis is instead:

> Existing tools usually rank or optimize targets at a fixed stage. OncoSyn investigates whether jointly modelling heterogeneous tumour-state evidence, modality-specific design constraints, decision uncertainty, failure scenarios, and experiment value can improve sequential multi-target immunotherapy research decisions.

For Version 1, this becomes a falsifiable question: can uncertainty-aware portfolio selection plus decision-sensitivity analysis reduce the experiments required to identify a stable neoantigen portfolio compared with score ranking, heterogeneity-aware optimization, generic uncertainty sampling, and other declared baselines?

The certificate records selected and excluded candidates, source mutations, inferred clusters, evidence, uncertainty, scenarios, same-input baseline comparisons, per-cluster support, explicitly defined uncovered-support scores, assumptions, limitations, failure modes, portfolio stability, and the next evidence check or experiment predicted to have decision value. That prediction is itself an experimental hypothesis.

A threshold-passing candidate or portfolio is a **computational hit promoted for downstream experimental investigation**, not a biological or clinical hit.

## Exactly three versions

1. **Version 1 — Heterogeneity-Aware Neoantigen Target Portfolio Engine.** Observed-state neoantigen portfolios, propagated uncertainty, stability analysis, fair baselines, and decision-focused validation prioritization.
2. **Version 2 — Modality-Specific Immunotherapy Co-Design Engine.** Separate vaccine/TCR-T and CAR target-and-logic tracks sharing only evidence, provenance, uncertainty, optimization primitives, and experiment schemas.
3. **Version 3 — Adaptive Immunotherapy R&D Engine.** Evidence-gated stress tests, experiment selection, observation-driven updating, contingency hypotheses, and carefully bounded evolutionary scenarios.

Version 2 CAR research must model accessible surface antigens, quantitative density and its uncertainty, tumour and normal-cell states, logic feasibility, heterogeneity, and antigen-loss/downregulation scenarios. It cannot claim to design a safe or effective CAR-T product. Version 3 must not claim to predict patient tumour evolution; state-space or sequential-control models require identifiable transition and observation evidence first.

Small-molecule target optimization remains deferred.

## Claim boundary

OncoSyn may report a computational research hypothesis, modelled scenario, design hypothesis, stress test, validation recommendation, experiment-selection hypothesis, or contingency strategy. It does not claim diagnosis, treatment benefit, relapse prevention, clinical coverage, safety, efficacy, validated hits, partnerships, regulatory readiness, or patient-specific evolutionary prediction.

See [`README.md`](README.md), [`docs/research-and-validation.md`](docs/research-and-validation.md), and [`docs/architecture/`](docs/architecture/).
