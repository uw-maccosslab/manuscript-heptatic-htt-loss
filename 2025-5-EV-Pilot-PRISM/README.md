# Carroll Lab Pilot to Study Huntingtin's mutants

### Received 40 mouse samples from two experiments.  
- There was a knockout (fl/fl; CAG/+) and a knockout control (fl/fl; +/+). There were 10 animals in each group 50/50 male/female split.  
- There was also a knockin of Q111 and a WT control. There were 10 animals in each group 50/50 male/female split.  

### Sample Preparation and Data Collection.
- We used our normal EV Mag-Net sample preparation. 
- Data were acquired using the Orbitrap Astral
- These are the same data as the 2025-5-EV-Pilot/ dataset, but we are reanalyzing with PRISM to get improved normalized protein abundances for downstream analysis.

### Data Analysis
- DIA-NN v2.2.3 using the Carafe UI.
- Imported into Skyline and used Nick's imputation to eliminate peptide missing data.
- Exported the PRISM Report and MetaData-Replicates.csv from the document grid.
- Data were processed using [skyline-prism](https://github.com/maccoss/skyline-prism) for:
    - Library-assisted transition rollup (interference detection)
    - RT-LOWESS peptide normalization
    - Median-polish protein rollup
    - Used the 2025-5-EV-Pilot-PRISM/PRISM-Output/corrected_proteins.parquet as the main input for downstream analysis.
- Configuration: [`prism-config.yaml`](prism-config.yaml)
