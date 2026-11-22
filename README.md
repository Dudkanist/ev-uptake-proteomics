# Quantitative Proteomic Profiling of Differential EV Cargo Uptake in Osteoblastic Niches

[![Pixi Badge](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/prefix-dev/pixi/main/assets/badge/v0.json&style=flat-square)](https://pixi.sh)
[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg?style=flat-square)](LICENSE)
[![Python 3.14+](https://img.shields.io/badge/python-3.14+-blue.svg?style=flat-square)](https://www.python.org/downloads/)

This repository contains the end-to-end computational workflow, statistical framework, presentation slides, and technical reports for quantifying differential extracellular vesicle (EV) cargo internalization by recipient osteoblast-like **SAOS-2** cells. The experimental model compares the uptake of vesicles derived from parental non-metastatic (**PC3**) versus bone-metastatic (**PC3M-luc-C6**) prostate cancer lines to distinguish selective cargo enrichment from passive bulk endocytosis.

---

## Experimental Design & MS Acquisition

- **Metabolic Pulse-Chase Tracing:** Donor prostate cancer cells were metabolically labeled with Heavy isotopes (Arg10 / Lys-D8 SILAC), separating internalized donor EV cargo (2.5% of identified PSMs, $n = 2,209$) from the unlabeled Light recipient SAOS-2 cellular background (97.5% of PSMs, $n = 85,929$).
- **Multiplexed Isobaric Quantification:** 18-plex TMTpro SPS-MS3 acquisition on a Thermo Orbitrap Eclipse across natural uptake co-cultures, unincubated cell-lysate + EV spike-in standards, negative biological controls (`SAOS-2 alone`), empty channels, concentrated EV boost channels, and a technical repeat (`135N` of `PC3M_EV_R1`).
- **Targeted Real-Time Search (RTS):** RTS duty-cycle filtering selectively triggered SPS-MS3 scans for Heavy precursors, mitigating reporter ion ratio compression from the co-isolated Light cellular background ($q \le 0.01$ at PSM and protein levels via Sequest HT / Percolator).

---

## Analytical Pipeline & Key Findings

The computational workflow in [`main.ipynb`](main.ipynb) progresses through nine stages documented across the project reports:

1. **Upstream Spectral QC & Carrier Artifact Depletion:** Verification of tryptic digestion parity across Light and Heavy pools, RTS duty-cycle diagnostics (`Quan Info`), and exclusion of 7 carrier-restricted peptides detected exclusively in boost/spike-in channels ($1,018 \to 1,011$ quantifiable Heavy peptides).
2. **Sequence-Level Shared Peptide Curation (*Progress Report 1*):** Re-mapping of 73 ambiguous shared peptides against the reviewed human Swiss-Prot reference proteome (`UP000005640`). Using an Occam's razor parsimony rule anchored to 372 uniquely confirmed genes, 45 records (44 unique peptides after merging modification variants of `SEVDMLK` in *ANXA2*) were rescued across 20 confirmed gene loci, while 28 unresolved paralogous peptides were grouped into 19 standardized multigene families (`Family_*`), yielding a consolidated matrix of **393 protein groups**.
3. **Sample-Level QC, Technical Replicate Removal & Background Gating (*Methodological Rationale* & *Progress Report 4*):** Exclusion of failed preparation replicate `PC3_EV_R1` (`130N`) and verification/exclusion of technical repeat `135N` (duplicate of `PC3M_EV_R1` / `131C`; Pearson $r = 0.9867$, mean $\log_2$ difference $= -0.009$), establishing a clean 5-channel biological uptake cohort (**$n = 2$ PC3, $n = 3$ PC3M**). Non-parametric dual-threshold gating against `SAOS-2 alone` negative controls ($\alpha = 6.6$ median noise floor; $\beta = 2.0$ fold-change cutoff derived from a 23.9% control CV) retained **238 high-confidence internalized EV cargo proteins**.
4. **Condition-Matched Spike-In Normalization & Scale Decoupling:** Isolated zero values (3 measurements across `CCN2`, `RPL8`, and `PALS2`; 99.7% completeness) were imputed at $\text{LOD}/2 = 0.8$. Uptake intensities were normalized against condition-matched unincubated spike-in references. Uncentered PCA revealed a global scale effect along PC1 (67.5% variance; 213/238 positive loadings); applying **sample-median centering** decoupled bulk uptake volume (and any uniform particle dosing offset) from relative cargo stoichiometry (PC1: 56.5% variance, cleanly separating PC3 and PC3M).
5. **Empirical Bayes Differential Uptake Modeling (`limma-trend`):** While unmoderated Welch's $t$-testing ($q < 0.10, \vert{}\log_2\text{FC}\vert{} \ge 1.0$) identified only 8 candidates due to low degrees of freedom ($n_1 = 2$ vs. $n_2 = 3$), Empirical Bayes variance shrinkage (`limma-trend`) identified **51 statistically significant differential uptake targets** (**29** enriched in metastatic PC3M uptake, **22** enriched in parental PC3 uptake), retaining 40/42 (95.2%) targets from the pre-correction run with zero sign flips.
6. **Unsupervised Phenotypic Clustering:** Two-way hierarchical clustering of the 51 significant targets cleanly bifurcated parental PC3 and metastatic PC3M replicates without supervision into two reciprocal uptake modules.
7. **Non-Parametric Rank-Shift Benchmarking (*Progress Report 2*):** Orthogonal validation using intra-channel ordinal abundance ranks (ΔRank = Rank<sub>Uptake</sub> − Rank<sub>Spike-in</sub> and cross-phenotype ΔΔRank = ΔRank<sub>PC3M</sub> − ΔRank<sub>PC3</sub>) confirmed strong monotonic concordance with `limma-trend` effect sizes (Spearman *r*<sub>s</sub> = 0.916), with a symmetric Venn overlap of **37 dual-validated targets** (14 `limma`-only, 14 rank-only at |ΔΔRank| ≥ 50).
8. **Topological Stratification & Orthogonal Literature Validation (*Progress Report 3* & *4*):** Deterministic gating against the *In Silico Human Surfaceome* (`SURFY`) and UniProtKB ontologies (with targeted curation of 5 legacy-symbol/unannotated hits) stratified the 51 targets into `Extracellular` ($n = 8$; **7 PC3M vs. 1 PC3**), `Membrane` ($n = 11$; **5 vs. 6**), `Organellar` ($n = 14$; **9 vs. 5**), and `Intracellular` ($n = 18$; **8 vs. 10**). Cross-referencing against human EV proteomics records in **Vesiclepedia (v5.1)** confirmed **100% (51/51)** of significant targets as established human EV constituents (7 resolved via legacy HGNC symbols).
9. **Conserved High-Uptake Candidates Across Conditions (*Progress Report 4*):** Examining the upper-right quadrant ($n = 76$ proteins gaining rank in both PC3 and PC3M) and ranking the top 15 by mean rank gain revealed a clear topological split: two canonical membrane integrins (`ITGA1`, `ITGA3`) show a sharp relative share jump specifically in PC3M, whereas the remaining organellar- and intracellular-annotated conserved gainers lean toward PC3.

---

## Documentation & Project Reports

| Document | Scope & Key Contents |
| :--- | :--- |
| **[`Methodological Rationale.pdf`](Methodological%20Rationale.pdf)** | **Primary Technical Report:** Experimental setup, spectral QC, $\alpha/\beta$ noise gating, spike-in normalization, mathematical proof of dosing invariance, PCA scale-effect removal, and `limma-trend` modeling (initial unique-peptide baseline). |
| **[`Progress Report 1.pdf`](Progress%20Report%201.pdf)** | **Ambiguous Peptide & Protein Group Resolution:** Reference re-mapping against Swiss-Prot (`UP000005640`), Occam's razor parsimony rescue of 20 gene loci, and creation of 19 multigene families (`Family_*`), expanding the matrix to 393 protein groups. |
| **[`Progress Report 2.pdf`](Progress%20Report%202.pdf)** | **Non-Parametric Rank-Shift Selectivity:** Formulation of $\Delta\text{Rank}$ and $\Delta\Delta\text{Rank}$ metrics and macro-concordance benchmarking against `limma-trend`. |
| **[`Progress Report 3.pdf`](Progress%20Report%203.pdf)** | **Topological Stratification & Orthogonal Validation:** Structural classification via `SURFY` + UniProtKB and 100% empirical validation across human EV studies in *Vesiclepedia v5.1*. |
| **[`Progress Report 4.pdf`](Progress%20Report%204.pdf)** | **Technical Replicate Removal (`135N`) & Pipeline Re-Evaluation:** Verification and exclusion of technical duplicate `135N`, full re-run on clean $2$ vs. $3$ biological replicates (51 targets, 7 vs. 1 extracellular, 9 vs. 5 organellar, $r_s = 0.916$), and conserved high-uptake candidate profiling. |
| **[`Quantitative Proteomic Profiling of EV Uptake.pdf`](Quantitative%20Proteomic%20Profiling%20of%20EV%20Uptake.pdf)** | **Presentation Deck:** Summary slides, speaker notes, and supplemental tables/figures reflecting the final 51-target cohort. |

---

## Quick Start & Environment Setup

This project uses [Pixi](https://pixi.sh) for reproducible dependency resolution across Python and R environments (`r-base`, `bioconductor-limma`, `rpy2`).

```bash
# 1. Clone the repository
git clone https://github.com/Dudkanist/ev-uptake-proteomics.git
cd ev-uptake-proteomics

# 2. Launch JupyterLab in the managed environment
pixi run jupyter

```

---

## Repository Structure

```text
├── main.ipynb                                         # Complete reproducible analysis notebook
├── figures/                                           # Exported figures (fig_001.png – fig_021.png)
├── Methodological Rationale.pdf                       # Primary technical report and mathematical rationale
├── Progress Report 1.pdf                              # Follow-up #1: Shared peptide parsimony & multigene families
├── Progress Report 2.pdf                              # Follow-up #2: Non-parametric rank-shift selectivity (ΔRank / ΔΔRank)
├── Progress Report 3.pdf                              # Follow-up #3: SURFY topology & Vesiclepedia v5.1 validation
├── Progress Report 4.pdf                              # Follow-up #4: Technical replicate (135N) removal & final 51-hit re-run
├── Quantitative Proteomic Profiling of EV Uptake.pdf  # Final project presentation deck
├── data/                                              # Input directory (excluded from VCS; see Data Availability)
│   ├── MEK_25_02_label_included.txt                   # Proteome Discoverer peptide-level export
│   ├── UP000005640_9606.fasta.gz                      # UniProt reviewed human reference proteome (auto-downloaded)
│   ├── table_S3_surfaceome.xlsx                       # SURFY in silico human surfaceome master table
│   └── VESICLEPEDIA_PROTEIN_MRNA_DETAILS_5.1.txt      # Vesiclepedia v5.1 compendium flat file
├── pixi.toml / pixi.lock                              # Declarative dependency locks (Python 3.14 + R limma)
├── environment.yml                                    # Conda-compatible environment export
├── CITATION.cff                                       # Machine-readable citation metadata
├── LICENSE                                            # GNU General Public License v3.0
└── README.md

```

### Data Availability

Raw mass spectrometry output files (`MEK_25_02_label_included.txt`) are excluded from version control in compliance with institutional data governance policies and are available upon reasonable request. The canonical human proteome (`UP000005640_9606.fasta.gz`) is automatically fetched from UniProt by `main.ipynb` if not present locally, while public reference tables (`SURFY` Table S3 and `Vesiclepedia` v5.1) can be obtained from their respective publications.

---

## Citation & License

This project is distributed under the [GNU General Public License v3.0 (GPL-3.0)](LICENSE).

```bibtex
@software{mantsevich2026ev,
  author    = {Mantsevich, Vladislav},
  title     = {Quantitative Proteomic Profiling of Differential Extracellular Vesicle Uptake in Osteoblastic Niches},
  year      = {2026},
  publisher = {GitHub},
  url       = {https://github.com/Dudkanist/ev-uptake-proteomics}
}

```