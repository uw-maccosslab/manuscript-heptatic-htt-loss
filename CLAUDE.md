# CLAUDE.md — collab-uw-carroll

Instructions for Claude Code when working in this repository.

---

## Project Overview

MacCoss Lab × Carroll Lab collaboration on Huntington's disease EV proteomics (mouse models). The primary analysis pipeline is:

1. **DIA-MS acquisition** → raw `.raw` files
2. **DIA-NN / Skyline** → protein/peptide quantitation reports
3. **PRISM** ([skyline-prism](https://github.com/maccoss/skyline-prism)) → normalized, batch-corrected protein parquet
4. **Jupyter notebooks** → statistical analysis and visualization using `proteomics_toolkit`

---

## Repository Structure

```
collab-uw-carroll/
├── 2025-5-EV-Pilot/                 # Raw Skyline analysis (exploratory)
├── 2025-5-EV-Pilot-PRISM/           # PRISM-normalized analysis (primary)
│   ├── 2025-5-EV-Pilot-PRISM_Analysis.ipynb
│   ├── PRISM-Output/
│   │   ├── corrected_proteins.parquet   ← main input for analysis
│   │   └── sample_metadata.csv
│   └── prism-config.yaml
├── proteomics_toolkit/              # Shared analysis library (edit as needed)
├── notebook/                        # Technical design documents (Markdown)
│   └── agents.md                   ← formatting guide for notebook entries
├── .venv/                          # Python virtual environment
└── CLAUDE.md                       # This file
```

---

## Python Environment

- **Virtual environment:** `.venv/` (Python 3.12)
- **Activate:** `source .venv/bin/activate`
- **Key packages:** pandas, numpy, scipy, statsmodels, scikit-learn, matplotlib, seaborn, pyarrow, jupyter, ipykernel
- **Install:** `.venv/bin/pip install <package>`

Always use `.venv/bin/python` or `.venv/bin/pip` for this project.

---

## proteomics_toolkit

This is the **standard analysis framework** for all data in this repo. It lives at `proteomics_toolkit/`. When performing analyses:

- **Always use `proteomics_toolkit` functions where they exist.** Do not reinvent normalization, visualization, or statistical testing from scratch.
- If needed functionality is missing, **extend the toolkit** (add a new function to the appropriate module) rather than writing one-off code in a notebook.
- Export new functions from `proteomics_toolkit/__init__.py` so they are available as `ptk.<function>`.

### Key functions for PRISM data

```python
import proteomics_toolkit as ptk

# Load PRISM corrected parquet + metadata
protein_data, metadata, sample_cols = ptk.load_prism_data(
    'PRISM-Output/corrected_proteins.parquet',
    'PRISM-Output/sample_metadata.csv'
)

# Strip batch suffix (__@__) for display
col_map = ptk.strip_batch_suffix(sample_cols)

# QC
ptk.plot_box_plot(data, sample_cols, meta_dict, ...)
ptk.plot_pca(data, sample_cols, meta_dict, ...)

# Stats — see "Statistical Analysis Data Prep" section below for full details
config = ptk.StatisticalConfig()
config.group_column = 'Group'
config.group_labels = ['KI Control', 'KI']  # [reference, study]
config.statistical_test_method = 'welch_t'
# ... set other config attributes

# IMPORTANT: pass protein_annotations so results include accession, gene, description
results = ptk.run_comprehensive_statistical_analysis(data, sample_meta_dict, config, protein_annotations=annot_df)

# Visualization
ptk.plot_volcano(results, ...)
```

---

## Notebook Conventions

### Jupyter Notebooks

- Each experiment directory (e.g., `2025-5-EV-Pilot-PRISM/`) gets its own analysis notebook.
- Notebook naming: `<project-dir-name>_Analysis.ipynb`
- Figures go to `figures/`, results CSVs to `results/` (created with `os.makedirs(..., exist_ok=True)`)
- Use the `.venv` kernel (`collab-uw-carroll`) for all notebooks.

### Notebook Entries (Design Documents)

Technical design documents live in `notebook/`. See `notebook/agents.md` for the full formatting spec. Key rules:
- Required sections: Title, Overview, Core Content, Open Questions, References, Version History
- File naming: `<topic>_<subtopic>.md` in snake_case
- Use tables, section separators (`---`), and the formatting conventions in `agents.md`
- One entry per major analysis arm or experimental question

---

## PRISM Data Format

PRISM outputs protein data with:
- **Annotation columns:** `protein_group`, `leading_protein`, `leading_name`, `leading_uniprot_id`, `leading_gene_name`, `leading_description`, `n_peptides`, `n_unique_peptides`, `low_confidence`
- **Sample columns:** `<ReplicateID>__@__<batch_name>` (e.g., `A1-DDY6-001__@__2025-5-Carroll-Mouse-EV-PRISM`)
- Use `ptk.strip_batch_suffix()` to map full column names → short replicate IDs for metadata joining
- Filter `low_confidence == True` rows before analysis

---

## Statistical Analysis Defaults

- **Test:** Two-sample t-test (Welch's)
- **Multiple testing:** Benjamini-Hochberg FDR
- **Significance:** adj. p < 0.05, |log₂FC| ≥ 1.0
- **Enrichment:** Enrichr API via `ptk.run_differential_enrichment()`

### Statistical Analysis Data Prep

`run_comprehensive_statistical_analysis()` expects a specific data structure:

1. **Data input:** First 5 columns must be `['Protein', 'Description', 'Protein Gene', 'UniProt_Accession', 'UniProt_Entry_Name']`, followed by sample columns.
2. **DataFrame index:** Set the index to the `Protein` column (accession number) so results contain meaningful protein identifiers — **not** integer row indices.
3. **Metadata:** Pass as a `dict` mapping sample column names → metadata dicts: `{col_name: {'Group': ..., 'Replicate': ...}}`.
4. **`StatisticalConfig`**: Zero-argument constructor. Set attributes individually (e.g., `config.group_column = 'Group'`).
5. **`protein_annotations`:** **Always pass** a DataFrame with `Protein`, `Description`, `Protein Gene`, `UniProt_Accession`, `UniProt_Entry_Name` columns. Without this, results will **lack** gene names, descriptions, and accessions — making them unreadable and breaking downstream enrichment analysis.

```python
# Correct data prep pattern for PRISM data:
annot = protein_data_filtered[['leading_protein', 'leading_description', 'leading_gene_name',
                                'leading_uniprot_id', 'leading_name']].copy()
annot.columns = ['Protein', 'Description', 'Protein Gene', 'UniProt_Accession', 'UniProt_Entry_Name']
data = pd.concat([annot.reset_index(drop=True), protein_data_filtered[sample_cols].reset_index(drop=True)], axis=1)
data.index = data['Protein']  # accession as index

results = ptk.run_comprehensive_statistical_analysis(data, meta_dict, config, protein_annotations=annot)
```

**Output columns include:** `Protein`, `logFC`, `P.Value`, `adj.P.Val`, `AveExpr`, `t`, `Protein Gene`, `Description`, `UniProt_Accession`, `Gene`

---

## Git / Workflow

- This repo was moved from another project context — the main branch is `main`.
- Commit frequently with descriptive messages.
- Do not commit large data files (parquet, CSV) unless they are essential and small.
  - The `.gitignore` should cover most data files.
- Do not commit `.venv/` — it is project-local.

---

## Common Tasks

### Add a new analysis to an existing experiment

1. Add a new section to the relevant `*_Analysis.ipynb` notebook.
2. If a new toolkit function is needed, add it to the appropriate `proteomics_toolkit/*.py` module and export from `__init__.py`.
3. Create or update the corresponding notebook entry in `notebook/`.

### Start a new experiment

1. Create `<experiment-dir>/` with the Skyline/PRISM outputs.
2. If using PRISM, add `<experiment-dir>/<experiment-dir>_Analysis.ipynb` modeled on `2025-5-EV-Pilot-PRISM/2025-5-EV-Pilot-PRISM_Analysis.ipynb`.
3. Add notebook entries in `notebook/` following `agents.md`.
4. Update `README.md`.
