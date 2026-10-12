# Quantitative Proteomic Profiling of Differential EV Uptake in Osteoblastic Niches

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/)
[![License: Internal/Academic](https://img.shields.io/badge/license-Academic_Research-green.svg)]()

This repository contains the computational pipeline and orthogonal statistical framework for quantifying differential extracellular vesicle (EV) uptake by osteoblastic SAOS-2 cells, comparing non-metastatic (**PC3**) and aggressive metastatic (**PC3M**) prostate carcinoma cargo.

---

## Experimental Framework & Methodology
- **Cargo Tracing:** Heavy metabolic labelling (Arg10 / Lys-D8 SILAC) of donor EV cargo against unlabelled recipient SAOS-2 cellular background.
- **Multiplexed Acquisition:** 18-plex TMTpro workflow measured on Thermo Orbitrap Eclipse using SPS-MS3 (MIPS precursor isolation, strict co-isolation threshold $\le 75\%$).
- **Identification:** Dual-branch Sequest HT searches with Percolator SVM target-decoy scoring ($q \le 0.01$).

---

## Analytical Pipeline
1. **Background Noise Decontamination:** Experimental subtraction relative to recipient control channels (`SAOS2 alone` $N=3$); threshold $\ge 2\times$ baseline.
2. **Spike-in & Loading Calibration:** Ratio-to-spike normalization accounting for NTA particle loading ($159$ complete cases across all $7$ channels).
3. **Sample-Median Centering:** Transformation resolving specific molecular cargo composition from bulk endocytosis volume artifacts.
4. **Univariate Hypothesis Testing:** Two-sided Welch's unequal variances $t$-test with Benjamini–Hochberg FDR control ($q < 0.10, \vert{}\text{Log}_2\text{FC}\vert{} \ge 1.0$), identifying **12 core uptake markers**.
5. **Orthogonal Machine Learning:** Random Forest ensemble of decision stumps (`max_depth=1`, 15,000 trees; LOOCV Accuracy = **85.7%**), revealing an invariant linear separability plateau of **58 proteins** with **ITGB1 (Integrin $\beta 1$)** at rank #1.

---

## Reproducibility & Environment Setup

This project uses [Pixi](https://pixi.sh) for deterministic Conda environment resolution:

```bash
# Recommended (instant setup via Pixi):
pixi run jupyter

# Alternative (derived from pixi):
# conda env create -f environment.yml
```

---

## Repository Contents
- `main_refractored.ipynb`: Fully documented Python pipeline (data cleaning, PCA diagnostics, Welch's test, Random Forest, Clustered Heatmap, Orthogonal Volcano).
- `Quantitative_Proteomic_Profiling_of_EV_Uptake.pdf`: Complete 21-slide deck prepared for laboratory leadership review.

*Note: Raw mass spectrometry output files are excluded from version control in compliance with institutional data governance policies.*

## Citation & License

This project is licensed under the terms of the GNU General Public License v3.0 (GPL-3.0). 
Any reuse, adaptation, or inclusion of this pipeline in academic publications or derived codebases requires explicit attribution to the author.

If you use this work, please cite:
```bibtex
@software{mantsevich2026ev,
  author = {Mantsevich, Vladislav},
  title = {Reproducible Proteomic Profiling Pipeline of Extracellular Vesicle Uptake},
  year = {2026},
  url = {[https://github.com/Dudkanist/ev-uptake-proteomics](https://github.com/Dudkanist/ev-uptake-proteomics)}
}
