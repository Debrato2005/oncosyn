# ADR 0006: PyClone-VI clonal-inference provider

## Status

Accepted for MVP planning. No runtime, PyClone-VI environment, provider, or test fixture exists yet.

## Problem

OncoSyn's portfolio objective needs clone/subclone assignments, cellular prevalence (CCF), and uncertainty. Those values are not inferred by OncoSyn's novel algorithm. The MVP adopts PyClone-VI as an upstream biological-inference component and must neither invent missing variant/copy-number/purity inputs nor couple the domain or optimizer to PyClone-VI's TSV, CLI, or HDF5 files.

## Verified upstream contract

PyClone-VI documents a tab-delimited mutation-by-sample input with these required columns in its [official README](https://github.com/Roth-Lab/pyclone-vi):

| Column | Required meaning |
| --- | --- |
| `mutation_id` | Stable mutation identifier shared across samples. |
| `sample_id` | Tumour sample identifier. |
| `ref_counts` | Reads supporting the reference allele. |
| `alt_counts` | Reads supporting the alternate allele. |
| `major_cn` | Major allele copy number for the segment overlapping the mutation. |
| `minor_cn` | Minor allele copy number for that segment. |
| `normal_cn` | Total copy number in healthy tissue at that segment. |

`tumour_content` and `error_rate` are optional upstream columns; PyClone-VI defaults them to `1.0` and `0.001`. OncoSyn will be stricter: `tumour_content` is required and must be explicitly supplied for every selected tumour sample. OncoSyn will not manufacture purity, copy-number values, or read counts. Variant allele frequency is derived from the supplied counts when needed for display or validation; it is not an independent input and must not be fabricated.

PyClone-VI removes mutations lacking entries for all detected samples. The OncoSyn input adapter therefore requires a complete mutation-by-selected-sample count/copy-number matrix for the inference set, or rejects the inference request with a typed insufficient-input error. It never silently lets upstream dropping change the clone universe.

The official result file contains `mutation_id`, `sample_id`, `cluster_id`, `cellular_prevalence`, `cellular_prevalence_std`, and `cluster_assignment_prob`. `cluster_id` is the most probable assignment; cellular prevalence is the CCF estimate. These are uncertainty-bearing estimates, not definitive lineage or biological facts.

## Selected integration boundary

```text
normalized variant/copy-number/purity records
  -> ClonalInferenceProvider interface
  -> PyCloneVIProvider serializes a private TSV
  -> `pyclone-vi fit` produces a private HDF5 result
  -> `pyclone-vi write-results-file` produces a TSV
  -> provider parses and validates normalized inference result
  -> mutation -> inferred-clone assignment with uncertainty
  -> candidate -> source mutation -> inferred clone assignment
```

The domain receives only normalized records:

- **input:** stable mutation and sample IDs; reference/alternate read counts; allele-specific major/minor copy number; normal copy number; explicit tumour content; declared PyClone-VI settings and provider version;
- **output:** mutation/sample most-probable cluster assignment, cellular prevalence and standard error, posterior for that assignment, provider version/settings, source-record identities, and a typed status;
- **not output:** a definitive parent-child clone tree or any clinical conclusion.

`PyCloneVIProvider` owns TSV serialization, process invocation, private temporary files, HDF5/result parsing, and translation to the normalized result. The application service owns provider selection and run status. The domain, candidate pipeline, optimizer, repositories, and API must not import PyClone-VI or read its file formats.

Use a version-pinned dedicated PyClone-VI environment or executable, not an unverified in-process dependency of the future application runtime. The official installation path is a Conda/Bioconda environment. Before enabling the real provider, verify the executable and version in the target runtime. If that check fails, return `clonal_inference_provider_unavailable`; do not silently substitute another inference algorithm. A deterministic fixture provider exists only for tests and offline demonstrations and is labelled as such in its output.

## Uncertainty and mapping semantics

The normalized result retains both `cellular_prevalence_std` and `cluster_assignment_prob`. Candidate-to-clone mapping must retain the source mutation's most-probable `cluster_id` and its assignment posterior; it must not invent probabilities for unreported alternative clusters. The portfolio service receives clone prevalence/CCF uncertainty and candidate-to-clone assignment uncertainty as scenario inputs. It may optimize against declared baseline and adverse scenarios but must report modelled residual uncovered clone mass or modelled escape risk, never a clinical escape probability.

PyClone-VI clusters mutation prevalence patterns. It does not establish a definitive evolutionary ancestry. Clone coverage is always allowed; a phylogenetic/ancestry visualization is allowed only if an explicit compatible reconstruction supplies lineage edges and provenance.

## Validation and failure behaviour

Reject before invoking the provider when any selected mutation/sample record is missing an identifier, read counts, allele-specific copy-number values, normal copy number, or explicit tumour content; when counts are unusable; or when the selected inference matrix is incomplete. The error identifies missing fields and record identities without echoing sensitive molecular values.

Return typed failures for:

- unavailable/mis-versioned PyClone-VI executable or environment;
- serialization or private-workspace failure;
- non-zero `fit` or `write-results-file` execution;
- malformed, incomplete, or non-finite result values;
- result rows that cannot be matched back to the immutable normalized input; and
- cancellation or timeout once execution limits are introduced.

No failed or partial inference result becomes a successful escape certificate. Temporary upstream artifacts are private, excluded from logs and version control, and deleted according to the run's approved retention policy once such storage exists.

## Test strategy before implementation

- Use deterministic synthetic normalized variant matrices containing explicit counts, copy numbers, and tumour content. No real patient data, external credentials, or network access is permitted.
- Test input validation for every mandatory field, incomplete mutation/sample matrices, and rejected absent purity/copy-number information.
- Unit-test the serializer against a checked-in synthetic TSV expectation and the parser against checked-in synthetic PyClone-VI result TSV fixtures.
- Contract-test the provider interface with a deterministic fixture provider that returns known cluster distributions, CCF standard errors, and assignment probabilities.
- Add a separately marked, offline integration test that runs the real version-pinned PyClone-VI executable against the same synthetic fixture only after the environment is provisioned. It must compare normalized fields, not rely on an undocumented internal file format.
- Test that unavailable PyClone-VI produces the typed unavailable result and never falls back to a different algorithm.
- Test that candidate-to-mutation-to-clone mapping preserves the documented assignment uncertainty and that the optimizer consumes it through declared scenarios.

## Alternatives rejected

- **Continue requiring supplied clone assignments:** rejected because the MVP now includes clonal inference.
- **Pass raw VCF/BAM files directly to PyClone-VI:** rejected because the upstream contract needs extracted read counts, allele-specific copy number, and tumour content; raw ingestion and variant calling remain out of scope.
- **Fill missing purity/copy-number values with defaults:** rejected because it creates untraceable biological assumptions. PyClone-VI's upstream defaults do not authorize OncoSyn to invent them.
- **Treat `cluster_id` as a phylogenetic tree:** rejected because clustering is not definitive ancestry reconstruction.
- **Replace a missing PyClone-VI runtime with another tool:** rejected because it changes biological inference without an explicit decision and validation.

## Deferred work

Raw-data processing, variant calling, purity/copy-number estimation, lineage reconstruction, longitudinal clonal evolution, and probability calibration are separate work. Any change to PyClone-VI version, input-completeness policy, or provider replacement requires a revision to this record and the test fixtures.
