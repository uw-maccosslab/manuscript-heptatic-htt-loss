# EV Pilot PRISM-Normalized Protein Analysis

## Overview

This document describes the analysis strategy for the Carroll Lab EV Pilot experiment (May 2025) using PRISM-corrected protein-level output. It covers the data processing decisions, statistical framework, and toolkit usage for the PRISM analysis arm of this experiment.

The companion raw-data analysis is documented in the `2025-5-EV-Pilot/` directory and [notebook entry](ev_pilot_raw_analysis.md).

---

## Experimental Design

**Objective:** Characterize EV (extracellular vesicle) proteome changes in two mouse models relevant to Huntington's disease — a knock-in (KI) model expressing the Q111 allele and a conditional knockout (KO) model.

**48 total samples** run in 4 batches (A–D, 12 samples/batch):

| Group | Genotype | Age (weeks) | n | Description |
|-------|----------|-------------|---|-------------|
| KI | Q111 | 12 | 10 | Huntington's KI — disease model |
| KI Control | WT | 12 | 10 | Wild-type age/sex-matched controls |
| KO | fl/fl; CAG/+ | 7 | 10 | Conditional KO — loss-of-function model |
| KO Control | fl/fl; +/+ | 7 | 10 | Cre-negative controls |
| Pool | Carolyn pooled reference | — | 4 | Inter-batch reference (PRISM uses as `reference_pattern`) |
| Carl | C57BL/6 reference | — | 4 | QC reference strain (PRISM uses as `qc_pattern`) |

**Statistical contrasts:**
1. KI vs KI Control — Huntington's KI effect at 12 weeks
2. KO vs KO Control — conditional KO effect at 7 weeks
3. KO Control vs KI Control — baseline age/strain difference between controls
4. Male vs Female — sex effect across all experimental samples

---

## PRISM Processing

Data were processed using [skyline-prism](https://github.com/maccoss/skyline-prism). Key configuration decisions:

### Transition → Peptide Rollup
- Method: `library_assist` with spectral library `2025-5-Carroll-Mouse-EV-redo.blib`
- `median_polish` fitting, outlier threshold = 1.0 (removes transitions >2× predicted)
- Minimum 3 transitions required

### Global (Peptide-Level) Normalization
- Method: `rt_lowess` — corrects RT-dependent systematic effects (ion suppression, gradient drift) plus overall loading differences
- Parameters: frac=0.3, n_grid_points=100

### Batch Correction
- Disabled (`batch_correction.enabled: false`) — single batch detected at peptide level

### Protein Rollup
- Method: `median_polish` — robust to outlier peptides (interference, PTMs)
- Minimum 2 peptides per protein

### Protein Normalization
- Method: `median` — final protein-level median centering

### Reference Samples
- **Pool** (`-CarrollPool-`): Used to learn technical variation for PRISM correction
- **Carl** (`-CarlC57`): Used as intra-experiment QC validation

### Output
- `corrected_proteins.parquet` — normalized, batch-corrected protein abundances
- 9,147 proteins (before low-confidence filtering) × 48 experimental samples

---

## Analysis Notebook

**Location:** `2025-5-EV-Pilot-PRISM/2025-5-EV-Pilot-PRISM_Analysis.ipynb`

### Workflow Summary

```
1. Load corrected_proteins.parquet  →  ptk.load_prism_data()
2. Filter low-confidence proteins   →  protein_data['low_confidence'] flag
3. Build sample_metadata dict       →  strip_batch_suffix() + metadata join
4. QC plots                         →  ptk.plot_box_plot(), plot_pca(), plot_control_correlation_analysis()
5. Statistical analysis             →  ptk.run_comprehensive_statistical_analysis()
   ├── KI vs KI Control (Welch t-test, BH correction)
   ├── KO vs KO Control (Welch t-test, BH correction)
   ├── KO Control vs KI Control (Welch t-test, BH correction)
   └── Male vs Female (Welch t-test, BH correction)
6. Volcano plots                    →  ptk.plot_volcano() for all 4 contrasts
7. Significant protein summary      →  Top proteins by FDR and raw p-value
8. Gene set enrichment              →  ptk.run_differential_enrichment()
   ├── KI vs KI Control (Up/Down)
   ├── KO vs KO Control (Up/Down)
   ├── KO Control vs KI Control (Up/Down)
   └── Male vs Female (Up/Down)
9. MS acquisition QC metric review  →  report.stats.tsv
10. Export results                  →  results/*.csv for all 4 contrasts
```

### Key File Paths

| File | Description |
|------|-------------|
| `PRISM-Output/corrected_proteins.parquet` | PRISM-normalized protein abundances (main input) |
| `PRISM-Output/sample_metadata.csv` | Sample annotations (Group, Genotype, Age, Sex) |
| `PRISM-Output/qc_report.html` | PRISM QC report with normalization diagnostics |
| `prism-config.yaml` | PRISM run configuration |
| `report.stats.tsv` | DIA-NN/Skyline MS acquisition QC metrics |
| `results/KI_vs_KIControl_stats.csv` | Differential analysis — KI vs KI Control |
| `results/KO_vs_KOControl_stats.csv` | Differential analysis — KO vs KO Control |
| `results/KOControl_vs_KIControl_stats.csv` | Differential analysis — KO Control vs KI Control |
| `results/Male_vs_Female_stats.csv` | Differential analysis — Male vs Female |

### Output Figures

| Figure | Description |
|--------|-------------|
| `figures/01_protein_boxplot.png` | Protein intensity distributions per sample |
| `figures/02_pca.png` | PCA of PRISM-normalized protein abundances |
| `figures/03_control_correlation.png` | Pool + Carl control sample correlation heatmap |
| `figures/05_volcano_KI_vs_KIControl.png` | Volcano plot — KI contrast |
| `figures/06_volcano_KO_vs_KOControl.png` | Volcano plot — KO contrast |
| `figures/05b_volcano_KOControl_vs_KIControl.png` | Volcano plot — control comparison |
| `figures/05c_volcano_Male_vs_Female.png` | Volcano plot — sex effect |
| `figures/07_enrichment_KI_*.png` | Enrichment bar plots — KI up/down |
| `figures/08_enrichment_KO_*.png` | Enrichment bar plots — KO up/down |
| `figures/08b_enrichment_Ctrl_*.png` | Enrichment bar plots — control comparison up/down |
| `figures/08c_enrichment_Sex_*.png` | Enrichment bar plots — sex effect up/down |
| `figures/09_qc_metrics.png` | MS acquisition QC metrics by group |

---

## proteomics_toolkit Usage

This analysis makes use of `proteomics_toolkit` throughout. Key functions:

| Function | Purpose |
|----------|---------|
| `ptk.load_prism_data()` | Load PRISM parquet + metadata; handles `__@__` batch suffix |
| `ptk.strip_batch_suffix()` | Strip PRISM batch suffix for sample name matching |
| `ptk.plot_box_plot()` | QC — protein intensity distributions |
| `ptk.plot_pca()` | QC — sample clustering |
| `ptk.plot_control_correlation_analysis()` | QC — Pool and Carl sample reproducibility |
| `ptk.StatisticalConfig()` | Configure differential analysis parameters |
| `ptk.run_comprehensive_statistical_analysis()` | Welch t-test + BH multiple testing correction |
| `ptk.display_analysis_summary()` | Print significant protein counts per contrast |
| `ptk.plot_volcano()` | Visualize differential results |
| `ptk.EnrichmentConfig()` | Configure enrichment analysis parameters |
| `ptk.run_differential_enrichment()` | Enrichr-based gene set enrichment (Up/Down split) |
| `ptk.plot_enrichment_barplot()` | Visualize enrichment results |

### Enrichment Configuration

- **Gene set libraries:** GO_Biological_Process_2023, GO_Molecular_Function_2023, KEGG_2021_Human
- **P-value cutoff:** 0.05
- **Top N terms displayed:** 15 per direction per contrast
- **Analysis pattern:** Each contrast is split into Up-regulated and Down-regulated protein lists, each submitted separately to Enrichr

### Toolkit Improvement: `load_prism_data()`

The PRISM parquet format uses a different column convention than Skyline CSV exports. A new `load_prism_data()` function was added to `proteomics_toolkit/data_import.py` to:
1. Read the `corrected_proteins.parquet` file
2. Auto-detect PRISM annotation columns vs sample columns (float64 columns)
3. Detect and report the `__@__` batch suffix
4. Load and normalize the `sample_metadata.csv` column names for toolkit compatibility (`sample` → `Replicate`, `sample_type` → `Sample Type`)

---

## Statistical Approach

- **Test:** Two-sample t-test (Welch's) between study vs control groups
- **Multiple testing correction:** Benjamini-Hochberg FDR
- **Significance threshold:** adj. p < 0.05, |log₂FC| ≥ 1.0
- **Input data:** PRISM-corrected protein abundances (already normalized; log₂-scale from median-polish rollup)
- **Normalization in toolkit:** `normalization_method='prism'` (signals data is already log-scaled)
- **Contrasts:**
  1. KI vs KI Control — `group_labels = ['KI Control', 'KI']` → logFC = KI − KI Control
  2. KO vs KO Control — `group_labels = ['KO Control', 'KO']` → logFC = KO − KO Control
  3. KO Control vs KI Control — `group_labels = ['KI Control', 'KO Control']` → logFC = KO Control − KI Control
  4. Male vs Female — `group_column = 'Sex'`, `group_labels = ['F', 'M']` → logFC = M − F

---

## Comparison to Raw Analysis

| Aspect | Raw Analysis (2025-5-EV-Pilot) | PRISM Analysis (2025-5-EV-Pilot-PRISM) |
|--------|-------------------------------|----------------------------------------|
| Input | `dataset-protein-totalareas.csv` (Skyline export) | `corrected_proteins.parquet` |
| Normalization | Manual VSN (arcsinh + quantile scaling) in notebook | RT-LOWESS + median (PRISM) |
| Protein rollup | Pre-computed by Skyline | Median-polish in PRISM |
| Batch correction | Not applied | PRISM reference-based correction |
| Analysis framework | Ad hoc pandas/scipy | `proteomics_toolkit` |

---

## Open Questions and Discussion Points

### 1. KI vs KI Control — Biological Interpretation
- Q111 Huntington's KI model: which EV proteins are altered at 12 weeks?
- Are there known Huntington's disease biomarkers in the EV proteome?
- How do the enriched pathways (GO/KEGG) relate to known HD biology?

### 2. KO vs KO Control — Biological Interpretation
- Conditional KO with CAG-Cre driver: specificity of knockout should be confirmed by PRISM results (reduced target protein)
- Are the proteome changes consistent across sexes?

### 3. KO Control vs KI Control — Baseline Differences
- This contrast was added to assess whether the two control groups are comparable despite differences in age (7 vs 12 weeks) and strain background
- Significant differences here may confound interpretation of the primary KI/KO contrasts

### 4. Male vs Female — Sex Effects (Implemented)
- Sex-specific differential analysis was performed across all experimental samples
- Results exported to `results/Male_vs_Female_stats.csv`
- Enrichment analysis completed for sex-differential proteins

### 5. PRISM Reference Sample Performance
- Check PRISM QC report (`PRISM-Output/qc_report.html`) for Pool sample CVs and correction quality
- Carl QC samples provide independent validation of technical reproducibility

---

## References

1. **MacCoss et al.** Skyline-PRISM. https://github.com/maccoss/skyline-prism
   - *PRISM normalization and batch correction tool used for this analysis.*

2. **Ritchie ME, et al.** (2015). limma powers differential expression analyses for RNA-sequencing and microarray studies. *Nucleic Acids Research* 43(7):e47. https://doi.org/10.1093/nar/gkv007
   - *Median-polish protein rollup concept.*

3. **Benjamini Y, Hochberg Y.** (1995). Controlling the false discovery rate: a practical and powerful approach to multiple testing. *Journal of the Royal Statistical Society Series B* 57(1):289–300.
   - *Multiple testing correction method used.*

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 0.1 | 2026-03-08 | Initial notebook and analysis setup |
| 0.2 | 2026-03-09 | Updated to reflect 4 contrasts (added KO Control vs KI Control, Male vs Female), enrichment details, output figures catalog, enrichment config |
