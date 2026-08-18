# ADR 0008: Hybrid neoantigen candidate-provider boundary

## Status

Accepted for MVP planning after a primary-source review on 2026-08-18. No candidate provider, scientific runtime, schema, or fixture is implemented.

## Problem and prior art

OncoSyn needs a broad, provenance-bearing peptide candidate universe without reimplementing mature transcript interpretation or making a competitor's final ranking its domain model. Targeted-panel inputs may contain processed mutant protein context or supplied peptides, while research-rich inputs may contain VEP-annotated variants. Those inputs require different scientifically valid candidate-generation paths.

[pVACtools 7.1.2](https://pypi.org/project/pvactools/) is a mature neoantigen suite. pVACseq consumes a genotype-bearing, VEP-annotated VCF and performs transcript-aware mutant/wild-type sequence construction, peptide enumeration, prediction, filtering, tiering, and optional ML evaluation. Its `all_epitopes.tsv` retains broad peptide/HLA/transcript evidence; its default `filtered.tsv` applies a Top Score filter, and its aggregate report selects a Best Peptide per variant. The reduced outputs are unsuitable as OncoSyn's optimization universe. See the [pVACseq output contract](https://pvactools.readthedocs.io/en/stable/pvacseq/output_files.html) and [filter order](https://pvactools.readthedocs.io/en/latest/pvacseq/filter_commands.html).

Portfolio optimization across tumour heterogeneity is not unoccupied prior art. [NeoAgDT](https://pmc.ncbi.nlm.nih.gov/articles/PMC11076149/) already uses pVACtools-derived evidence, simulated heterogeneous tumour cells, a vaccine budget, and ILP MinSum/MinMax objectives. OncoSyn therefore treats explicit clonal-inference uncertainty, scenario robustness, stability, and auditability as a research hypothesis to benchmark—not established novelty or superiority.

## Selected design

Use an explicit provider boundary:

```text
NeoantigenCandidateProvider
  |-- PVACSeqProvider
  `-- NativeOncoSynProvider
        |-- PeptideEnumerator
        `-- MHCFlurryPredictor
```

- `PVACSeqProvider` is eligible only for a labelled Tier B VEP-annotated-VCF subtype satisfying the pinned pVACseq contract.
- `NativeOncoSynProvider` is eligible only for adequate processed mutant protein context or explicitly supplied peptides. It does not interpret genomic consequences or become a partial VEP replacement.
- Selection is explicit in versioned analysis configuration. Providers never silently fall back, and candidates from different providers are not mixed in one run without a separately designed harmonization protocol.
- Every result and certificate names the candidate provider, version, runtime, settings, reference build/data where applicable, predictors/models, filters, and source-data versions.

## pVACseq adapter contract

The adapter owns eligibility, input validation, version checking, private temporary files, subprocess execution, timeout/cancellation, output parsing, normalization, provenance, and cleanup. Domain code never sees pVACtools TSV columns or paths.

The adapter ingests `all_epitopes.tsv` before Top Score reduction. It may apply only explicitly selected, versioned standalone eligibility filters. It must not use `filtered.tsv`, aggregate Best Peptide, aggregate metrics, or ML evaluation as the candidate universe. Those reduced products may be retained as pVACseq baseline artifacts.

The input VCF is processed variant evidence, not raw sequencing. It requires sample `GT`, compatible VEP annotations and plugins, an explicit reference build/cache version, and normalized variant records. Optional tumour/normal coverage, RNA evidence, and proximal variants remain sourced evidence rather than OncoSyn-generated truth. pVACseq's VAF/purity clonality tiering is not clonal inference and cannot replace `ClonalInferenceProvider`.

## Native adapter contract

The enumerator deterministically emits requested peptide-length windows that contain the declared mutated residue(s), retaining sequence, mutation offset, source mutation, and flanks. Supplied peptides are validated rather than regenerated. MHCflurry produces per-peptide/per-HLA evidence; unsupported alleles and missing model bundles remain typed outcomes.

The Version 1 optimization unit is a unique mutant peptide. HLA-specific predictions are evidence attached to that peptide rather than separate portfolio slots. Version 2 Track A may promote `(peptide, HLA)` to target identity under a separate TCR-T contract.

## Mutation identity and normalized output

Provider-specific identifiers, gene names, protein changes, and peptide sequences are not mutation identity. A genomic mutation retains reference build, normalized contig/position/reference/alternate allele, original source identity, and an OncoSyn `mutation_id`. Multiallelic records are decomposed before identity assignment. Protein-context-only input uses its declared stable source mutation ID and cannot claim genomic equivalence.

Both providers return normalized `Candidate` and `CandidateEvidence` records containing source mutation, peptide, transcript/consequence provenance when present, HLA-specific raw and normalized predictor evidence, wild-type comparison when present, expression/coverage/transcript evidence when sourced, missingness, exclusions, and complete version/settings provenance. Eligibility, candidate quality, clonal evidence, and portfolio objectives remain distinct.

## Runtime and licensing

pVACtools 7.1.2 currently requires Python `>=3.9,<3.12`; MHCflurry 2.2.1 requires Python `>=3.10` and uses PyTorch. Run pVACtools in a pinned external environment and keep MHCflurry behind its adapter; verify versions again when implemented. The pVACtools package is BSD-3-Clause-Clear and MHCflurry is Apache-2.0, but many optional predictors are academic/non-profit only or unclear. MVP predictors require an explicit commercial-use-compatible allow-list and legal review; availability through pVACtools is not licensing approval.

## Failure and security behaviour

Reject incompatible input/provider combinations before execution. Preserve unsupported consequence/allele, malformed provider output, version mismatch, unavailable executable/model, timeout, partial output, unmatched mutation identity, and licensing-disabled predictor as typed failures. Do not echo molecular values, HLA genotypes, private paths, or raw payloads in logs/errors. No provider failure becomes a successful candidate universe.

## Testing and benchmarks

- Deterministic native enumeration and supplied-peptide tests, including mutation containment and deduplication.
- Golden pVACseq `all_epitopes` parser fixtures proving per-HLA/per-predictor evidence and source mutation survive while Top Score is excluded.
- Contract tests shared by both providers; explicit eligibility/no-fallback/no-mixing tests.
- Stable mutation-identity tests for normalized SNVs, indels, multiallelic records, transcripts, samples, and protein-context-only evidence.
- Selection-only benchmarks on the same normalized candidates and `K`; full-pipeline comparisons only where both branches can consume equivalent source evidence.
- pVACseq aggregate ranking and NeoAgDT-style MinSum/MinMax are material baselines where reproducible and legally usable. Negative, equivalent, and non-comparable results remain visible.

## Alternatives rejected and migration

- **Native-only:** rejected for research-rich inputs because rebuilding transcript consequence, frameshift, proximal-variant, and peptide construction is high-risk and not OncoSyn's research contribution.
- **pVACseq-only:** rejected because targeted processed protein/supplied-peptide inputs cannot satisfy its VCF/VEP contract and permanent coupling would weaken independent evolution.
- **Consume pVACseq final ranking:** rejected because Top Score and aggregate reduction discard portfolio diversity before optimization.

The native provider can expand only through separately tested consequence contracts. Additional pVACtools providers such as pVACbind, new predictors, or mixed-provider evidence require an ADR revision, licensing review, harmonization tests, and benchmark revision.
