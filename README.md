# Quantitative Proteomic Profiling of Differential EV Uptake in Osteoblastic Niches

[![Pixi Badge](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/prefix-dev/pixi/main/assets/badge/v0.json&style=flat-square)](https://pixi.sh)
[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg?style=flat-square)](https://www.gnu.org/licenses/gpl-3.0)
[![Python 3.14+](https://img.shields.io/badge/python-3.14+-blue.svg?style=flat-square)](https://www.python.org/downloads/)

This repository contains the end-to-end computational pipeline and statistical framework for quantifying differential extracellular vesicle (EV) cargo uptake by recipient osteoblastic SAOS-2 cells. The biological model compares internalization of parental (**PC3**) versus metastatic (**PC3M-luc-C6**) prostate carcinoma EV cargo.
---

## Experimental Design & MS Acquisition
- **Metabolic Pulse-Chase Tracing:** Heavy metabolic labelling (Arg10 / Lys-D8 SILAC) of donor EV cargo against an unlabelled recipient SAOS-2 cellular background (accounting for ~97.5% of PSMs).
- **Multiplexed Acquisition:** 18-plex TMTpro SPS-MS3 workflow acquired on a Thermo Orbitrap Eclipse with synchronous precursor selection (MIPS precursor isolation, co-isolation threshold < 75% to substantially mitigate ratio compression).
- **Identification & Confidence:** Real-Time Search (RTS) triggering, Sequest HT search engine, and Percolator target-decoy scoring ($q \le 0.01$ at PSM and protein levels).
---

## Analytical Pipeline

1. **Pre-analytical Sample QC & Outlier Exclusion:** Evaluation of channel-wide reporter yield identified severe labeling suppression in replicate `PC3_EV_R1` (>3.5-fold drop in total intensity, 33.1% dropout rate). It was excluded to prevent systemic dilution bias, yielding an authenticated 6-channel framework ($n=2$ for PC3, $n=4$ for PC3M).
2. **Background Noise Gating:** Empirical dual-threshold filtering against non-specific recipient background (`SAOS2 alone`, $N=3$) and empty channels. Retained $N=212$ high-confidence EV cargo proteins satisfying $\max(\text{Median}) > \alpha$ ($\alpha = 6.6$) and signal-to-background ratio $\ge \beta \times \text{Median}_{\text{ctrl}}$ ($\beta = 2.0$).
3. **Internal Spike-in Normalization & Loading Invariance:** Co-culture reporter signals were normalized against matched cell-lysate + EV reference spike standards (mean of technical replicates). Downstream sample-median centering algebraically cancels uniform additive dosing offsets (including NTA particle adjustments), ensuring that downstream metrics reflect intrinsic stoichiometric selectivity rather than particle count discrepancies.
4. **Resolution of Global Scale Effect (PC1 Bias):** Unsupervised PCA on uncentered uptake data revealed that PC1 accounted for 87.4% of total variance with 99.5% unidirectional positive loadings. Sample-median centering successfully decoupled bulk endocytic volume kinetics from relative cargo composition.
5. **Statistical Hypothesis Testing:**
   - **Baseline Screening (Welch's $t$-test):** Unmoderated unequal-variance testing ($q < 0.10, \vert{}\log_2\text{FC}\vert{} \ge 1.0$) identified **22 differential candidates**.
   - **Empirical Bayes Moderation (`limma-trend`):** Variance shrinkage borrowing dispersion information across the proteomic ensemble ($d_g = 4 \to d_g + d_0 \approx 7.1$) mitigated small-sample instability, expanding discovery to **37 statistically significant drivers** (21 PC3M-enriched, 16 PC3-enriched).
6. **Unsupervised Phenotype Clustering:** Hierarchical clustering on significant hits yielded clean blind bifurcation of biological replicates and partitioned the cargo into an **ECM remodeling/adhesion program** (COL4A2, TGM2, FN1, TNC, ITGA1/A3/B1) and an **ER proteostasis/endosomal module** (CD63, MAP3K12, HM13, GDF15, HSP90B1).

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

A cross-platform conda specification is provided via `environment.yml`:

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
├── main.ipynb                                     # Complete reproducible analysis notebook
├── TMT_pool.xlsx                                  # Experimental design and channel annotation metadata
├── Methodological Rationale.pdf                   # Detailed technical report and mathematical rationale
├── Quantitative Proteomic Profiling of EV Uptake.pdf  # Project presentation deck
├── pixi.toml / pixi.lock                          # Declarative dependency locks (Python 3.14 + R limma)
├── environment.yml                                # Conda-compatible environment export
├── LICENSE                                        # GNU General Public License v3.0
└── README.md

```

Note: Raw mass spectrometry output files (`MEK_*.txt`) are excluded from version control in compliance with institutional data governance policies. Data matrices are available upon reasonable request.

---

## Citation & License

This project is open-source under the [GNU General Public License v3.0 (GPL-3.0)](https://www.google.com/search?q=LICENSE).

```bibtex
@software{mantsevich2026ev,
  author    = {Mantsevich, Vladislav},
  title     = {Quantitative Proteomic Profiling of Differential Extracellular Vesicle Uptake in Osteoblastic Niches},
  year      = {2026},
  publisher = {GitHub},
  url       = {https://github.com/Dudkanist/ev-uptake-proteomics}
}
```
