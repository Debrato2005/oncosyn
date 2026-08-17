# ADR 0006: Clonal-inference provider and PyClone-VI adapter

## Status

Accepted for MVP planning. No provider/runtime/fixture is implemented.

## Problem and verified distinction

The optimizer needs uncertain mutation clusters and cellular prevalence, but clonal inference is upstream biology—not OncoSyn novelty. The original PyClone family and PyClone-VI both use sequencing-derived allele counts and copy-number/purity evidence. PyClone-VI adds scalable variational inference suitable for larger mutation sets; it is not the “WGS version,” and a number such as “6.1” may be a software/workflow version rather than a data-modality distinction. The [PyClone-VI repository](https://github.com/Roth-Lab/pyclone-vi), [PyClone-VI publication](https://pmc.ncbi.nlm.nih.gov/articles/PMC7730797/), and [original PyClone repository](https://github.com/Roth-Lab/pyclone) are the upstream sources; versions must be checked again at implementation.

Neither provider reconstructs a definitive phylogenetic tree. Cluster assignments/prevalence are uncertain estimates.

## Exact PyClone-VI contract

The official tabular input requires one mutation/sample row with `mutation_id`, `sample_id`, `ref_counts`, `alt_counts`, `major_cn`, `minor_cn`, and `normal_cn`. Upstream `tumour_content` and `error_rate` may have defaults; OncoSyn requires explicit `tumour_content` and declared settings. It never fabricates counts, copy number, or purity. Reported VAF/depth may support input adequacy checks but cannot become invented reference/alternate pseudo-counts.

PyClone-VI can drop mutations missing across detected samples. OncoSyn therefore validates the selected mutation-by-sample inference matrix and reports exclusions/incompleteness before execution; it cannot silently alter the inference universe.

Documented result fields are `mutation_id`, `sample_id`, `cluster_id`, `cellular_prevalence`, `cellular_prevalence_std`, and `cluster_assignment_prob`. These retain uncertainty and do not imply ancestry.

## Selected provider boundary

```text
normalized molecular evidence
  -> ClonalInferenceProvider.eligibility()
  -> explicit provider selection
     -> PyCloneVIProvider -> pinned process/files -> normalized result
     -> PrecomputedClonalEvidenceProvider -> validate provenance/schema -> normalized result
  -> candidate -> mutation -> uncertain clone distribution
```

Provider input: stable mutation/sample IDs, actual allele counts, allele-specific/normal copy number, explicit tumour content, sample design, settings, source identities, and input-tier metadata. Provider output: normalized mutation/cluster assignments, cellular prevalence and standard error, assignment posterior, exclusions/limitations, provider/version/settings, provenance, and typed status. A supplied compatible reconstruction may separately add lineage edges; inference output alone may not.

`PyCloneVIProvider` owns TSV serialization, process invocation, private temporary artifacts, result parsing, and normalization. `PrecomputedClonalEvidenceProvider` validates externally supplied estimates and provenance; it is never labelled as a PyClone-VI run. The application service selects a provider explicitly. Domain, optimizer, API, and repositories know only the generic contract and never silently fall back.

Use a pinned external executable/environment if PyClone-VI cannot safely coexist with the application runtime. An unavailable/mis-versioned executable returns `clonal_inference_provider_unavailable`. A deterministic fixture provider is test/demo-only and labelled in every result.

## Targeted-panel policy

Tier A does not guarantee adequate clonal inference. Eligibility must assess actual counts, depth/quality, purity, copy number, mutation density/breadth, and sample completeness. If mandatory PyClone-VI fields are absent, reject that provider. If technically runnable but sparse, expose a reduced-confidence/limited-ascertainment status and state that unobserved tumour heterogeneity is not covered. Minimum defensible thresholds are an open benchmark question in [`../research-and-validation.md`](../research-and-validation.md), not an invented constant.

## Uncertainty and optimizer semantics

Retain `cellular_prevalence_std` and `cluster_assignment_prob` through candidate-to-mutation-to-clone mapping. Do not invent posterior mass for unreported alternative clusters. The optimizer consumes declared perturbations/distributions and reports coverage, residual uncovered mass, or modelled escape risk—not clinical escape probability. Cluster coverage is allowed; ancestry views require separately compatible, provenance-bearing reconstruction.

## Failure and security behaviour

Return typed insufficient/limited input, provider unavailable, serialization/execution, malformed/non-finite result, unmatched-row, timeout/cancellation, and precomputed-provenance failures. Errors identify fields/records without echoing molecular values. No partial/failed inference becomes a successful certificate. Temporary artifacts remain private, unlogged, untracked, and subject to retention policy.

## Test strategy

- Deterministic synthetic matrices with explicit counts/CN/purity; no patient data, credentials, or network.
- Eligibility tests for both tiers, each mandatory field, incomplete matrices, VAF-only rejection, and sparse-panel limitation.
- Golden serializer/parser fixtures plus provider contract tests for PyClone-VI, precomputed evidence, and labelled fixture providers.
- Separately marked offline real-provider test in the pinned environment; compare normalized public fields, not internals.
- Prove provider unavailability never causes fallback; prove mappings preserve uncertainty and scenarios consume it.

## Alternatives rejected and deferred work

Rejected: raw VCF/BAM input; purity/CN/count defaults; pseudo-counts from VAF; cluster-as-tree; silent provider replacement; rejecting all precomputed evidence. Deferred: provider benchmarks, original-PyClone compatibility, other inference tools, raw processing, lineage reconstruction, longitudinal evolution, and calibrated panel-adequacy thresholds. Provider/threshold changes require this ADR and fixture revision.
