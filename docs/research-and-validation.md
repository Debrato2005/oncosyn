# Research and Validation Contract

This file owns validation, open research questions, computational-hit language, and collaboration framing; other docs link here.

## Hypothesis, hit, and metrics

For a declared tumour-state snapshot, evidence, constraints, and scenarios, OncoSyn tests whether a bounded portfolio is computationally more robust than a named baseline. A **computational hit** is promoted for downstream experimental investigation after predefined evidence/portfolio thresholds; it is not a biological or clinical hit.

MVP metrics: search-space reduction; retained high-confidence evidence; clone coverage and modelled residual uncovered mass; baseline trade-offs; processed-profile-to-certificate time; and breadth/agreement/provenance of evidence. The long-term metric—proportion surviving experimental validation—cannot be estimated before experiments exist.

## Baselines

Use the same eligible candidates, `K`, constraints, and input for: (1) score Top-`K`; (2) clonality-weighted ranking; (3) coverage-only selection; (4) reproducible escape/minimax where comparable; and (5) OncoSyn. Report negative and equivalent results.

## Validation ladder

1. Deterministic synthetic and permitted public computational evaluation.
2. Independently governed retrospective datasets with predefined endpoints.
3. In-vitro/cell/organoid experiments appropriate to the hypothesis.
4. Animal/preclinical studies; these do not establish human efficacy.
5. Much-later prospective clinical study after independent ethical, regulatory, safety, manufacturing, and governance requirements. No fixed FDA timeline.

Preclinical feedback may recalibrate models only with assay context, protocol, failures, uncertainty, and domain shift preserved.

## Open research questions

1. What minimum mutation count, depth, purity, copy-number quality, and breadth make targeted-panel clonal inference informative?
2. How should incomplete panel ascertainment limit coverage/robustness claims?
3. Which clonal uncertainty representation is faithful and tractable?
4. How sensitive are portfolios to provider, predictor, allele, threshold, and source disagreement?
5. Which presentation, expression, tumour-specificity, processing, and immunogenicity evidence adds reproducible value?
6. Which escape scenarios are defensible, identifiable, and useful rather than speculative?
7. Which formulation beats simple baselines without overfitting synthetic assumptions?
8. How stable are portfolios across regions, timepoints, sampling noise, and tumour state?
9. What endpoint defines experimental survival for vaccine, TCR-T, and later modalities?
10. How should negative experiments update weights without assay/modality bias?

## Potential collaboration asks

There are no claimed partnerships. Potential collaborators may include ICMR, NCG/Tata and other NCG centres, ICMR-NIPCR, and MAHE/KMC where appropriate. Outreach must request a specific contribution—review, permitted processed data, assay expertise, retrospective evaluation, or preclinical design—and never imply endorsement, access, or an existing relationship.

## Claims needing verification

Before external use, source claims about predictor performance, PyClone-family suitability, panel adequacy thresholds, immunogenicity, escape mechanisms, comparative novelty, time/search reduction, and translation. Mentor suggestions and planned benchmarks are not established facts. Record internal feedback as constraints/questions without personal attribution unless authorized.
