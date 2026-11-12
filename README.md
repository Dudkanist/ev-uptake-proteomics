# Quantitative Proteomic Profiling of Differential EV Cargo Uptake in Osteoblastic Niches

[![Pixi Badge](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/prefix-dev/pixi/main/assets/badge/v0.json&style=flat-square)](https://pixi.sh)
[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg?style=flat-square)](LICENSE)
[![Python 3.14+](https://img.shields.io/badge/python-3.14+-blue.svg?style=flat-square)](https://www.python.org/downloads/)

This repository contains the end-to-end computational workflow, statistical framework, and technical reports for quantifying differential extracellular vesicle (EV) cargo internalization by recipient osteoblast-like **SAOS-2** cells. The experimental model compares the uptake of vesicles derived from parental non-metastatic (**PC3**) versus bone-metastatic (**PC3M-luc-C6**) prostate cancer lines to distinguish active, selective cargo delivery from passive bulk endocytosis.

---

## Experimental Design & MS Acquisition

- **Metabolic Pulse-Chase Tracing:** Donor prostate cancer cells were metabolically labeled with Heavy isotopes (Arg10 / Lys-D8 SILAC), allowing unambiguous separation of internalized donor EV cargo (2.5% of identified PSMs, $n = 2,209$) from the unlabeled Light recipient SAOS-2 cellular background (97.5% of PSMs, $n = 85,929$).
- **Multiplexed Isobaric Quantification:** 18-plex TMTpro SPS-MS3 acquisition on a Thermo Orbitrap Eclipse across natural uptake co-cultures, unincubated cell-lysate + EV spike-in standards, negative biological controls (`SAOS-2 alone`), empty channels, and concentrated EV boost channels.
- **Targeted Real-Time Search (RTS):** RTS duty-cycle filtering selectively triggered SPS-MS3 scans for Heavy precursors, mitigating reporter ion ratio compression from the co-isolated Light cellular background ($q \le 0.01$ at PSM and protein levels via Sequest HT / Percolator).

---

## Analytical Pipeline & Key Findings

The computational workflow in [`main.ipynb`](main.ipynb) progresses through eight stages documented across the project reports:

1. **Upstream Spectral QC & Carrier Artifact Depletion:** Verification of tryptic digestion parity across Light and Heavy pools, RTS duty-cycle diagnostics (`Quan Info`), and exclusion of 7 carrier-restricted peptides detected exclusively in boost/spike-in channels ($1,018 \to 1,011$ quantifiable Heavy peptides).
2. **Sequence-Level Shared Peptide Curation (*Progress Report 1*):** Re-mapping of 73 ambiguous shared peptides against the reviewed human Swiss-Prot reference proteome (`UP000005640`). Using an Occam's razor parsimony rule anchored to 372 uniquely confirmed genes, 45 records (44 unique peptides after merging modification variants of `SEVDMLK` in *ANXA2*) were rescued across 20 confirmed gene loci, while 28 unresolved paralogous peptides were grouped into 19 standardized multigene families (`Family_*`), yielding a consolidated matrix of **393 protein groups**.
3. **Sample-Level QC & Empirical Background Gating (*Methodological Rationale*):** Exclusion of technical preparation outlier `PC3_EV_R1` (>3.5-fold intensity collapse, 34.1% dropout rate), establishing a validated 6-channel biological uptake cohort ($n = 2$ PC3, $n = 4$ PC3M). Non-parametric dual-threshold gating against `SAOS-2 alone` negative controls ($\alpha = 6.6$ baseline median noise floor; $\beta = 2.0$ signal-to-background fold-change cutoff derived from a 23.9% control CV) retained **226 high-confidence internalized EV cargo proteins**.
4. **Condition-Matched Spike-In Normalization & Scale Decoupling:** Boundary-level missing values (3 isolated zero measurements; 99.78% completeness) were imputed at $\text{LOD}/2 = 0.85$. Uptake intensities were normalized against condition-matched unincubated spike-in references. Unsupervised PCA on uncentered uptake profiles revealed a dominant global scale effect along PC1 (84.8% variance; 200/226 proteins projecting positively), reflecting higher overall bulk endocytosis of metastatic PC3M vesicles. Applying **sample-median centering** decoupled bulk uptake volume (and any uniform NTA particle dosing offset) from relative cargo stoichiometry, rebalancing PC1 to 50.6% explained variance.
5. **Empirical Bayes Differential Uptake Modeling (`limma-trend`):** While unmoderated Welch's $t$-testing ($q < 0.10, \vert{}\log_2\text{FC}\vert{} \ge 1.0$) identified 24 candidates due to degree-of-freedom collapse ($n_1 - 1 = 1$), borrowing variance information across the proteome via Empirical Bayes shrinkage (`limma-trend`) expanded discovery to **42 statistically significant cargo drivers** (23 enriched in metastatic PC3M uptake, 19 enriched in parental PC3 uptake).
6. **Unsupervised Phenotypic Clustering:** Two-way hierarchical clustering of the 42 significant drivers cleanly bifurcated parental PC3 and metastatic PC3M replicates without supervision, resolving a **metastatic ECM remodeling & integrin adhesion program** (`COL4A2`, `TGM2`, `FN1`, `TNC`, `CCN1`, `ITGA1`, `ITGA3`, `ITGB1`, `ANPEP`, `CTNND1`) versus a **parental ER proteostasis & vesicular trafficking module** (`CD63`, `MAP3K12`, `HM13`, `LMAN2`, `GDF15`, `HSP90B1`, `HSPA5`).
7. **Non-Parametric Rank-Shift Benchmarking (*Progress Report 2*):** Orthogonal validation using intra-channel ordinal abundance ranks ($\Delta\text{Rank} = \text{Rank}_{\text{Uptake}} - \text{Rank}_{\text{Spike-in}}$ and cross-phenotype $\Delta\Delta\text{Rank} = \Delta\text{Rank}_{\text{PC3M}} - \Delta\text{Rank}_{\text{PC3}}$) confirmed strong monotonic concordance with parametric `limma-trend` effect sizes (Spearman $r_s = 0.913$), with 27 core drivers passing both parametric thresholds and $\vert{}\Delta\Delta\text{Rank}\vert{} \ge 50$.
8. **Topological Stratification & Orthogonal Literature Validation (*Progress Report 3*):** Deterministic gating against the *In Silico Human Surfaceome* (`SURFY`) and UniProt subcellular ontologies stratified the 42 drivers into `Membrane` ($n = 11$), `Extracellular` ($n = 7$), `Organellar` ($n = 10$), and `Intracellular` ($n = 14$) compartments, revealing a 6-fold enrichment of extracellular matrix/corona proteins in metastatic PC3M vesicles (6 vs. 1 in PC3). Cross-referencing against human EV proteomics records in **Vesiclepedia (v5.1)** (accounting for post-2018 HGNC nomenclature updates) confirmed **100% (42/42)** of significant candidates as established human EV constituents.

---

## Documentation & Project Reports

To provide full transparency into both the mathematical derivations and iterative pipeline refinements, the repository includes the primary methodological report alongside three focused follow-up reports:

| Document | Scope & Key Contents |
| :--- | :--- |
| **[`Methodological Rationale.pdf`](Methodological%20Rationale.pdf)** | **Primary Technical Report:** Experimental setup, spectral QC, $\alpha/\beta$ noise gating, spike-in normalization, mathematical proof of dosing invariance, PCA scale-effect removal, and `limma-trend` modeling (initially evaluated on the conservative unique-peptide baseline cohort of $N = 372 \to 212$ proteins). |
| **[`Progress Report 1.pdf`](Progress%20Report%201.pdf)** | **Ambiguous Peptide & Protein Group Resolution:** Reference re-mapping against Swiss-Prot (`UP000005640`), Occam's razor parsimony rescue of 20 gene loci, and creation of 19 multigene families (`Family_*`), expanding the quantified cohort from $372 \to 393$ total proteins ($212 \to 226$ post-filtering). |
| **[`Progress Report 2.pdf`](Progress%20Report%202.pdf)** | **Non-Parametric Rank-Shift Selectivity:** Formulation of $\Delta\text{Rank}$ and $\Delta\Delta\text{Rank}$ metrics and macro-concordance benchmarking against `limma-trend` ($r_s = 0.913$). |
| **[`Progress Report 3.pdf`](Progress%20Report%203.pdf)** | **Topological Stratification & Orthogonal Validation:** Structural classification of the 42 differential drivers via `SURFY` and 100% empirical validation across published human EV studies in *Vesiclepedia v5.1*. |

---

## Quick Start & Environment Setup

### Option A: Using Pixi (Recommended)

This project uses [Pixi](https://pixi.sh) for reproducible dependency resolution across Python and R environments (`r-base`, `bioconductor-limma`, `rpy2`).

```bash
# 1. Clone the repository
git clone https://github.com/Dudkanist/ev-uptake-proteomics.git
cd ev-uptake-proteomics

# 2. Launch JupyterLab in the managed environment
pixi run jupyter

```

### Option B: Using Conda / Mamba

A cross-platform Conda specification is provided via `environment.yml`:

```bash
# 1. Create and activate environment
conda env create -f environment.yml
conda activate EV

# 2. Launch JupyterLab
jupyter lab

```

---

## Repository Structure

```text
├── main.ipynb                                         # Complete reproducible analysis notebook
├── Methodological Rationale.pdf                       # Primary technical report and mathematical rationale
├── Progress Report 1.pdf                              # Follow-up #1: Shared peptide parsimony & multigene families
├── Progress Report 2.pdf                              # Follow-up #2: Non-parametric rank-shift selectivity (ΔRank / ΔΔRank)
├── Progress Report 3.pdf                              # Follow-up #3: SURFY topology & Vesiclepedia v5.1 validation
├── Quantitative Proteomic Profiling of EV Uptake.pdf  # Project presentation deck
├── data/                                              # Input directory (excluded from VCS; see Data Availability)
│   ├── MEK_25_02_label_included.txt                   # Proteome Discoverer peptide-level export
│   ├── UP000005640_9606.fasta.gz                      # UniProt reviewed human reference proteome (auto-downloaded)
│   ├── table_S3_surfaceome.xlsx                       # SURFY in silico human surfaceome master table
│   └── VESICLEPEDIA_PROTEIN_MRNA_DETAILS_5.1.txt      # Vesiclepedia v5.1 compendium flat file
├── pixi.toml / pixi.lock                              # Declarative dependency locks (Python 3.14 + R limma)
├── environment.yml                                    # Conda-compatible environment export
├── LICENSE                                            # GNU General Public License v3.0
└── README.md

```

### Data Availability

Raw mass spectrometry output files (`MEK_25_02_label_included.txt`) are excluded from version control in compliance with institutional data governance policies and are available upon reasonable request. The canonical human proteome (`UP000005640_9606.fasta.gz`) is automatically fetched from UniProt by `main.ipynb` if not present locally, while public reference tables (`SURFY` Table S3 and `Vesiclepedia` v5.1) can be obtained from their respective publications.

---

## Citation & License

This project is distributed under the [GNU General Public License v3.0 (GPL-3.0)](https://www.google.com/search?q=LICENSE).

```bibtex
@software{mantsevich2026ev,
  author    = {Mantsevich, Vladislav},
  title     = {Quantitative Proteomic Profiling of Differential Extracellular Vesicle Uptake in Osteoblastic Niches},
  year      = {2026},
  publisher = {GitHub},
  url       = {https://github.com/Dudkanist/ev-uptake-proteomics}
}

```
