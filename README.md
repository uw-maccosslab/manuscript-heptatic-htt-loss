# Hepatic huntingtin loss drives an acute phase response and liver injury in multiple mouse models

---

## Project: 2025-5 EV Pilot

Proteomics characterization of extracellular vesicle (EV) proteomes from two mouse models of Huntington's disease (HD):

- **KI model** — Q111 knock-in allele (12 weeks)
- **KO model** — Conditional knockout via CAG-Cre (7 weeks)

48 samples within 1 batch. Primary comparisons: KI vs KI Control, KO vs KO Control.

### Analysis Arms

| Directory | Description |
|-----------|-------------|
| [`2025-5-EV-Pilot/`](2025-5-EV-Pilot/) | Raw Skyline export analysis — initial exploration with VSN normalization |
| [`2025-5-EV-Pilot-PRISM/`](2025-5-EV-Pilot-PRISM/) | **Primary analysis** — PRISM-normalized protein data with `proteomics_toolkit` |

### Key Files

| File | Description |
|------|-------------|
| `2025-5-EV-Pilot/2025-5-EV-Pilot_Analysis.ipynb` | Raw data exploratory notebook |
| `2025-5-EV-Pilot-PRISM/2025-5-EV-Pilot-PRISM_Analysis.ipynb` | PRISM analysis notebook (primary) |
| `2025-5-EV-Pilot-PRISM/PRISM-Output/corrected_proteins.parquet` | PRISM-normalized protein abundances |
| `2025-5-EV-Pilot-PRISM/PRISM-Output/qc_report.html` | PRISM QC diagnostic report |
| `2025-5-EV-Pilot-PRISM/prism-config.yaml` | PRISM run configuration |

---

## Repository Structure

```
collab-uw-carroll/
├── 2025-5-EV-Pilot/                 # Raw Skyline data + initial analysis
│   ├── 2025-5-EV-Pilot_Analysis.ipynb
│   ├── dataset-protein-totalareas.csv
│   └── MetaData-Replicates.csv
├── 2025-5-EV-Pilot-PRISM/           # PRISM-processed data + primary analysis
│   ├── 2025-5-EV-Pilot-PRISM_Analysis.ipynb
│   ├── prism-config.yaml
│   ├── report.stats.tsv
│   └── PRISM-Output/
│       ├── corrected_proteins.parquet
│       ├── sample_metadata.csv
│       ├── qc_report.html
│       └── qc_plots/
├── proteomics_toolkit/              # Shared analysis library
│   ├── data_import.py               # Includes load_prism_data()
│   ├── preprocessing.py
│   ├── normalization.py
│   ├── statistical_analysis.py
│   ├── visualization.py
│   ├── enrichment.py
│   ├── temporal_clustering.py
│   ├── validation.py
│   └── export.py
├── notebook/                        # Technical design documents
│   ├── agents.md                    # Notebook formatting guide
│   ├── ev_pilot_raw_analysis.md     # Raw analysis documentation
│   └── ev_pilot_prism_analysis.md   # PRISM analysis documentation
├── .venv/                           # Python virtual environment
└── CLAUDE.md                        # Claude Code instructions
```

---

## Setup

### Python Environment

```bash
# Create and activate virtual environment
python3 -m venv .venv
source .venv/bin/activate       # macOS/Linux
# .venv\Scripts\activate        # Windows

# Install dependencies
pip install pandas numpy scipy statsmodels scikit-learn \
            matplotlib seaborn pyarrow jupyter ipykernel

# Register kernel for Jupyter
python -m ipykernel install --user --name collab-uw-carroll \
       --display-name "collab-uw-carroll"
```

### Running Notebooks

```bash
cd 2025-5-EV-Pilot-PRISM
jupyter notebook 2025-5-EV-Pilot-PRISM_Analysis.ipynb
```

---

## proteomics_toolkit

A shared Python library for proteomics data analysis. Key modules:

| Module | Purpose |
|--------|---------|
| `data_import` | Load Skyline CSV or PRISM parquet data (`load_skyline_data`, `load_prism_data`) |
| `preprocessing` | Parse protein identifiers, classify samples |
| `normalization` | Median, VSN, quantile, LOESS normalization |
| `statistical_analysis` | t-test, mixed effects models, BH correction |
| `visualization` | Volcano plots, PCA, box plots, correlation heatmaps |
| `enrichment` | Enrichr-based gene set enrichment |
| `export` | Export results and configurations |

```python
import proteomics_toolkit as ptk

# Load PRISM data
protein_data, metadata, sample_cols = ptk.load_prism_data(
    'PRISM-Output/corrected_proteins.parquet',
    'PRISM-Output/sample_metadata.csv'
)
```

See [`proteomics_toolkit/README.md`](proteomics_toolkit/README.md) and [`PROTEOMICS_TOOLKIT_GUIDE.md`](PROTEOMICS_TOOLKIT_GUIDE.md) for full documentation.

---

## PRISM

Data were processed using [skyline-prism](https://github.com/maccoss/skyline-prism) for:
- Library-assisted transition rollup (interference detection)
- RT-LOWESS peptide normalization
- Median-polish protein rollup
- Reference-sample-based correction

Configuration: [`2025-5-EV-Pilot-PRISM/prism-config.yaml`](2025-5-EV-Pilot-PRISM/prism-config.yaml)

---

## Contact

- **MacCoss Lab:** [maccosslab.org](https://maccosslab.org) — University of Washington, Department of Genome Sciences
- **Carroll Lab:** University of Washington — Huntington's disease research


