# OncoSyn — Evolution-Aware Immunotherapy Hypothesis and Target-Portfolio Engine

## Problem

Given many plausible tumour-specific immunotherapy targets, which small combination should investigators take forward? OncoSyn aims to reduce the search and prioritization space between tumour molecular profiling and downstream experimental therapeutic development.

The MVP accepts processed tumour evidence, infers uncertain clonal structure through a replaceable provider, generates and scores peptide/HLA candidates, and selects a bounded portfolio across inferred heterogeneity and declared escape scenarios. It returns a traceable, falsifiable research hypothesis—not a treatment recommendation or validated therapeutic.

## MVP contract

**Processed tumour profile → uncertain clone evidence → neoantigen/T-cell candidates → evolution-aware `K`-candidate portfolio → hypothesis certificate.**

- **Tier A:** targeted-panel mutations; actual read counts where available (or VAF/depth for adequacy assessment); copy number; purity; sample metadata; HLA; optional expression.
- **Tier B:** WES/WGS-derived processed calls, matched normal, allele-specific copy number, HLA, RNA, and optional multiregion/longitudinal evidence.

Raw sequencing, calling, HLA typing, purity/copy-number estimation, and invented phylogeny are out of scope. Limited panels are labelled incomplete views of heterogeneity. PyClone-VI runs only with its exact actual read-count, copy-number, and tumour-content evidence; no defaults or pseudo-counts. Explicit validated precomputed clone evidence is a separate provider option.

```text
processed molecular data + HLA -> mutations
  -> ClonalInferenceProvider (PyClone-VI first, or explicit precomputed evidence)
  -> uncertain clone assignments/prevalence
  -> peptide enumeration -> MHCflurry-first prediction -> candidate evidence
  -> candidate -> mutation -> inferred clone distribution
  -> bounded optimizer over named scenarios -> hypothesis certificate
```

## Contribution and output

OncoSyn investigates **uncertainty-aware, multi-objective therapeutic portfolio selection across inferred tumour heterogeneity and defined evolutionary/escape scenarios**. PyClone-VI, MHCflurry, NetMHCpan, pVACtools-derived workflows, and other upstream tools remain replaceable and are not the novel algorithm.

The certificate tests whether portfolio `P`, under supplied tumour state and assumptions, is predicted more robust than named baseline `B`. It records selected candidates, source mutations, inferred clones, evidence, uncertainty, escape scenarios, baseline comparisons, coverage/residual uncovered mass, assumptions, failure modes, and next experiment or data check. Baselines include individual Top-`K`, clonality-weighted, coverage-only, and reproducible escape/minimax formulations.

A threshold-passing candidate/portfolio is a **computational hit promoted for downstream experimental investigation**, not a validated hit. Evaluation measures computational reduction, retained evidence, robustness, runtime, and source support; it must not invent clinical efficacy.

## Modality and claims

The MVP is a neoantigen/T-cell immunotherapy track. Peptide/HLA evidence applies to vaccine and TCR-T research. Conventional CAR-T is a separate future surface-antigen modality requiring tumour specificity, normal-tissue/off-tumour, prevalence, stability, heterogeneity, and antigen-loss constraints. Small-molecule target optimization is deferred.

OncoSyn does not claim a better vaccine, relapse prevention, clinical benefit, diagnosis, validation, or regulatory readiness. See [`README.md`](README.md), [`docs/architecture/`](docs/architecture/), and [`docs/research-and-validation.md`](docs/research-and-validation.md).
