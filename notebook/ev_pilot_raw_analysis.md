# EV Pilot Raw Data Analysis

## Overview

This document describes the exploratory analysis of the Carroll Lab EV Pilot experiment (May 2025) using raw Skyline protein export data. This was the initial analysis conducted before PRISM normalization and batch correction were applied.

The PRISM-normalized analysis is documented in [ev_pilot_prism_analysis.md](ev_pilot_prism_analysis.md).

---

## Experimental Design

Same as the PRISM analysis — see [ev_pilot_prism_analysis.md](ev_pilot_prism_analysis.md#experimental-design) for the full table. In brief:
- 48 samples across 4 batches (A–D)
- 4 groups: KI, KI Control, KO, KO Control
- 2 control/reference types: Pool (inter-batch reference), Carl (C57BL/6 QC)

---

## Analysis Notebook

**Location:** `2025-5-EV-Pilot/2025-5-EV-Pilot_Analysis.ipynb`

### Input Files

| File | Description |
|------|-------------|
| `dataset-protein-totalareas.csv` | Skyline protein total area export (~9,528 proteins × 51 columns) |
| `MetaData-Replicates.csv` | Sample annotations (Replicate, Group, Genotype, Age, Sex) |
| `report.stats.tsv` | DIA-NN/Skyline MS acquisition QC metrics |

### Workflow Summary

```
1. Load dataset-protein-totalareas.csv + MetaData-Replicates.csv
2. Explore raw distributions (box plots by group)
3. Evaluate 5 normalization approaches:
   ├── Unnormalized          — raw Skyline protein total areas
   ├── Median centering      — additive offset to overall median
   ├── Median scaling        — multiplicative scaling to median-of-medians
   ├── VSN (selected)        — arcsinh + quantile-based scaling (variance-stabilizing)
   └── Median + MAD          — (x − median) / MAD standardization
4. Correlation analysis under each normalization (Pool + Carl samples)
5. PCA comparison — 2×2 grid: Original, VSN, Median Scaled, Median+MAD
6. Statistical analysis on VSN data:
   ├── KO vs KO Control
   ├── KI vs KI Control
   ├── KO Control vs KI Control (with enhanced labeling)
   └── Male vs Female
7. Statistical analysis on Median Scaled data (KO, KI contrasts only)
8. MS acquisition QC metrics (report.stats.tsv)
```

---

## Normalization Decision

After evaluating five approaches, **VSN (arcsinh transformation)** was selected as the primary normalization for this raw analysis:

```
scaling_factor = 1 / np.quantile(data, 0.5)
vsn_data = np.arcsinh(data * scaling_factor)
```

**Rationale:**
- Median centering performed poorly (correlation between controls was poor)
- Median scaling improved distributions but did not stabilize variance
- VSN stabilized variance across the intensity range and improved control correlations
- Median+MAD was considered but VSN was preferred for its established use in proteomics

**Comparison approach:** Correlation matrices (Pearson) for Pool and Carl reference samples were computed under each normalization to assess technical reproducibility. PCA plots were generated in a 2×2 grid comparing Original, VSN, Median Scaled, and Median+MAD to visualize sample clustering behavior.

> **Note:** In the PRISM analysis, RT-LOWESS normalization (which additionally corrects for RT-dependent ion suppression effects) was used, resulting in better overall data quality. The PRISM analysis is the primary analysis for downstream biological interpretation.

---

## Key Findings (Raw Analysis)

- ~9,528 proteins detected before filtering
- Pool and Carl control samples showed good correlation after VSN normalization
- 4 statistical contrasts performed on VSN-normalized data:
  - KO vs KO Control, KI vs KI Control, KO Control vs KI Control, Male vs Female
- KO/KI contrasts also repeated with Median Scaled normalization for comparison
- Volcano plots produced for each contrast (|log₂FC| ≥ 0.5 threshold shown)
- No gene set enrichment analysis in this notebook (see PRISM analysis for enrichment)

### Statistical Methods
- **Test:** Independent samples t-test (two-tailed)
- **Multiple testing correction:** Benjamini-Hochberg FDR
- **Significance threshold (implicit):** adj. p < 0.05
- **Fold-change threshold (shown in plots):** |log₂FC| ≥ 0.5

---

## Relationship to PRISM Analysis

| Aspect | Raw Analysis | PRISM Analysis |
|--------|-------------|----------------|
| Normalization | VSN (ad hoc, 5 methods compared) | RT-LOWESS + median (PRISM) |
| Tool | Pandas/scipy in notebook | `proteomics_toolkit` + PRISM |
| Contrasts | 4 (KI, KO, Ctrl, Sex) | 4 (same) |
| Enrichment | No | Yes (Enrichr) |
| Purpose | Initial exploration, normalization evaluation | Primary analysis |
| Recommended for publication | No | Yes |

---

## Open Questions and Discussion Points

### 1. Normalization Comparison
- Direct comparison between VSN-normalized and PRISM-normalized results could reveal which proteins are sensitive to the choice of normalization method.

### 2. Reproducibility
- Differences between the raw and PRISM analyses should be documented; large differences warrant investigation.

---

## References

1. **Huber W, et al.** (2002). Variance stabilization applied to microarray data calibration. *Bioinformatics* 18(Suppl 1):S96–S104.
   - *VSN normalization approach used in raw analysis.*

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 0.1 | 2026-03-08 | Initial documentation of existing raw analysis notebook |
| 0.2 | 2026-03-09 | Updated workflow to reflect all 4 contrasts, 5 normalization methods, correlation/PCA comparison details, statistical methods |
