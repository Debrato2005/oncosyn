# ADR 0010: Version 2 CAR target-and-logic research boundary

## Status

Accepted as a future research contract. No CAR schema, model, optimizer, dataset integration, or validation is implemented.

## Problem and prior-art boundary

Conventional CARs recognize accessible cell-surface antigens, not peptide–HLA targets. Surface-target discovery, target-pair discovery, Boolean logic design, multi-antigen targeting, and density-sensitive circuits already have substantial prior art, including [SCAN-ACT](https://pubmed.ncbi.nlm.nih.gov/40814001/) and single-cell logic-design methods. OncoSyn must not claim novelty for finding a target pair or enumerating AND/OR/NOT expressions.

The open research question is whether patient/cohort-specific tumour and normal-cell antigen states, quantitative density and uncertainty, logic feasibility, heterogeneity, escape scenarios, and experiment value can jointly improve which CAR design hypotheses researchers advance.

## Selected research design

Version 2 Track B compares research designs such as single-target, dual/tandem/bicistronic, OR-gated, AND-gated, NOT-gated, synNotch, iCAR, Tmod, adaptor/universal, and higher-order multi-antigen hypotheses only when the selected architecture has an explicit feasible contract.

The design variable is not merely an antigen set. It is a versioned hypothesis containing:

- target antigen identities and accessible surface evidence;
- tumour cell-state and lesion prevalence;
- normal-cell-state evidence and uncertainty;
- quantitative antigen-density distributions and assay context;
- logic architecture and implementability constraints;
- accessibility, shedding, internalization, spatial heterogeneity, and stability evidence where available;
- antigen-loss/downregulation and measurement scenarios;
- modelled tumour-state inclusion and normal-state exclusion;
- missing safety and feasibility evidence; and
- feasible next experiments and decision linkage.

RNA expression and public atlases may supply contextual priors but cannot establish patient-specific surface density, accessibility, normal-tissue exclusion, or CAR safety. Unknown evidence is never favourable.

## Boundaries

Track B may model target/logic research decisions. It does not model trafficking, exhaustion, persistence, cytokine toxicity, full tumour-microenvironment biology, manufacturing, receptor sequence design, or clinical response unless a later accepted contract and validation support a narrowly defined research endpoint.

It cannot output “safe CAR,” “effective CAR,” “prevents escape,” or a clinical recommendation. Permitted outputs are design hypothesis, modelled inclusion/exclusion under declared states, stress test, missing-evidence statement, and validation recommendation.

## Baselines and evidence gate

Compare with single-target ranking, expression-only pairs, SCAN-ACT-like pair discovery where reproducible, Boolean-coverage baselines, and density-agnostic robust selection on the same admissible evidence. Version 2 requires quantitative tumour and normal controls plus collaborator-confirmed construct/assay feasibility before prospective design-utility claims.

## Testing strategy

Future tests must reject peptide–HLA records; distinguish RNA, protein, accessible surface, and quantitative density evidence; preserve assay units/context; represent normal states and unknowns; validate logic truth tables and feasibility; stress antigen loss/downregulation; show design changes and no-changes; and prohibit safety/efficacy wording.

## Alternatives rejected

- **Reuse the Version 1 peptide model:** mechanistically invalid.
- **Expression-only CAR scoring:** insufficient for density, accessibility, normal-state, and circuit questions.
- **Target-pair discovery as novelty:** contradicted by prior art.
- **Model all CAR biology initially:** scientifically and experimentally underdetermined.
