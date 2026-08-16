# ADR 0002: Escape-aware portfolio result as an auditable certificate

## Status

Accepted as the intended MVP design. No optimizer or API has been implemented yet.

## Problem

Independent candidate ranking can choose redundant targets while leaving a tumour subclone uncovered. A portfolio optimizer alone is insufficient if it hides clonal uncertainty, HLA restriction, biological assumptions, source evidence, or the reason a candidate was excluded.

## Constraints

- The MVP is limited to neoantigen-vaccine portfolio exploration.
- Inputs include uncertainty-bearing PyClone-VI mutation-cluster assignment and cellular-prevalence estimates derived from validated read-count, copy-number, and tumour-content evidence; optional expression evidence may supplement them.
- A clone tree cannot be inferred from clustering alone and must not be invented for visualization.
- Computational outputs cannot establish clinical efficacy.
- The result must remain reproducible when prediction tools, inputs, or scenario assumptions change.

## Selected design

Represent the outcome as an **escape certificate**, not a bare top-K list. A certificate contains:

- the immutable normalized input and schema version;
- candidate evidence and provenance, including peptide-enumerator, predictor/source versions;
- PyClone-VI provider version/settings, validated molecular-input identity, and inferred clone-assignment/prevalence uncertainty;
- selected and excluded candidates with their marginal portfolio contribution;
- clone-level expected and worst-case coverage/escape summaries;
- declared escape scenarios and their assumptions, including HLA-specific or global presentation failures when supported by the input model;
- optimizer version, settings, deterministic tie-breaking, solve status, and infeasibility explanation;
- uncertainty that could change the selected portfolio; and
- ranked next evidence actions, explicitly labeled public external evidence or patient-specific verification.

The optimizer receives normalized candidate and clone-evidence records; it does not call prediction tools or databases directly. The delivery layer renders the certificate but cannot alter its contents.

## Alternatives rejected

- **Top-K individual scores:** rejected because it does not encode collective clone coverage or explicit worst-case escape behavior.
- **An opaque composite score:** rejected because it makes trade-offs and failure modes unauditable.
- **A forced portfolio when constraints cannot be met:** rejected because it disguises uncovered clones. Return an infeasible or explicitly partial result instead.
- **Always render a phylogeny:** rejected because parent-child lineage requires compatible reconstruction evidence beyond a clone cluster label.

## Intended data flow

```text
candidate evidence and inferred clone-distribution records
  -> validate scenario configuration and K constraint
  -> optimize expected and robust clone coverage
  -> calculate selected/excluded contributions and uncertainty sensitivity
  -> assemble immutable escape certificate
  -> present coverage map or evidence-backed clone tree
```

## Error and security behavior

- Invalid candidate, inferred-clone, HLA, or scenario records fail validation with a typed, safe error.
- Insufficient PyClone-VI input or unavailable inference provider produces a typed failure and no successful certificate.
- If the solver cannot meet hard constraints, return a structured infeasible result with unmet constraints; never silently drop clones.
- If optional evidence is unavailable, retain an explicit missing/uncertain status and continue only when the configured scenario permits it.
- Do not expose solver internals, stack traces, or sensitive input values through an external interface.

## Testing strategy

- Unit tests for overlap, K limits, clone coverage, deterministic ties, PyClone-VI-derived assignment/prevalence uncertainty, HLA-specific scenario effects, missing evidence, and infeasible constraints.
- Property or scenario tests proving that adding a redundant candidate cannot be represented as new clone coverage.
- End-to-end synthetic cases comparing the robust portfolio to naive top-K ranking, including transparent certificate provenance.
- Contract tests ensuring a displayed tree is absent unless lineage edges are supplied by the input contract.

## Deferred work

Calibration of scenario probabilities with clinical data, longitudinal tumour evolution, drug-target mode, raw-data integration, and clinical outcome claims are outside the MVP.
