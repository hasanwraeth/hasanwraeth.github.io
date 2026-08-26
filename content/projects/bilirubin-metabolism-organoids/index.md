---
title: "Synthetic Augmentation of Bilirubin Metabolism in Human Pluripotent Stem Cell-Derived Liver Organoids"
date: 2023-10-12
summary: "Engineered human liver organoids with synthetic GULO gene to boost intracellular ascorbate and NRF2-driven bilirubin conjugation, validated in a Crigler-Najjar syndrome rat transplant model"
tags:
  - Synthetic Biology
  - Stem Cell Biology
  - Organoids
  - RNA-seq
  - Epigenomics
  - Bioinformatics
  - Regenerative Medicine
tech_stack:
  - RNA-seq & GSEA
  - ChIP-seq / ChIP-reChIP
  - Synthetic Gene Knock-in (Tet-ON)
  - iPSC-Derived Organoids
  - Flow Cytometry
  - R (Statistical Analysis)
  - Bilirubin Conjugation Assays
links:
  - type: paper
    url: https://doi.org/10.1016/j.stemcr.2023.09.006
    label: Stem Cell Reports Paper
  - type: github
    url: https://github.com/hasanwraeth/RNAseq
    label: Bioinformatics Code
  - type: dataset
    url: https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE222362
    label: GEO Dataset
featured: true
status: "Published"
role: "First Author — Experimental & Computational Lead"
duration: "2020–2023"
team_size: 9
highlights:
  - "First author paper in Stem Cell Reports (Cell Press), recognized as Cincinnati Children's Top Scientific Achievement"
  - "Engineered organoids with synthetic GULO gene to boost bilirubin conjugation via NRF2 reactivation"
  - "Modeled Crigler-Najjar syndrome using patient-derived iPSCs with UGT1A1 mutation"
  - "Transplantation reduced bilirubin by ~30% in Gunn rats, persisting to day 60"
  - "Bioinformatics analysis code published on GitHub"
---

Engineered human liver organoids (eHLOs) with a synthetic GULO gene to augment intracellular ascorbate synthesis, reactivating NRF2-driven bilirubin conjugation — a synthetic biology approach to treating Crigler-Najjar syndrome, a rare genetic liver disorder.

## Overview

UGT1A1 is the primary enzyme required for bilirubin conjugation, and its deficiency causes Crigler-Najjar syndrome (CNS), a devastating condition requiring lifelong phototherapy or liver transplantation. Animal models lack key human transporter proteins and have different epigenetic regulation of UGT1A1, necessitating a human-specific model.

We developed human liver organoids (HLOs) from iPSCs that emulate bilirubin conjugation, modeled CNS using patient-derived iPSCs, and engineered a synthetic biology solution to boost conjugation capacity.

## My Role

As first author, I designed and performed all experiments and computational analyses, and wrote the paper. This work was part of my PhD dissertation at the University of Cincinnati / Cincinnati Children's Hospital Medical Center.

Key contributions:
- **Patient-derived iPSC modeling**: Generated stable iPSC lines from a Crigler-Najjar syndrome patient (UGT1A1 c.858C>A, p.Cys280X), differentiated to HLOs, and validated the conjugation failure phenotype
- **Epigenetic mechanism discovery**: ChIP and ChIP-reChIP experiments revealing that glucocorticoid receptor recruits MECP2 to repress UGT1A1, while glucocorticoid antagonists (mifepristone) prevent this recruitment
- **Transcriptomic analysis**: RNA-seq with GSEA identifying NRF2 as the key downstream target — activated by glucocorticoid antagonism but with off-target inflammatory effects
- **Synthetic biology engineering**: Designed and implemented mGULO (murine L-gulonolactone oxidase) knock-in with P2A-mCherry under a Tet-ON system to augment intracellular ascorbate for NRF2 reactivation
- **In vivo validation**: Orthotopic transplantation of eHLOs into Gunn rats (CNS model) via portal vein injection

## Key Results

### Disease Modeling
- Patient-derived CNS HLOs lacked UGT1A1 expression and failed to conjugate bilirubin (0 mU/mg vs. 4.75 mU/mg in healthy HLOs, 7 mU/mg in PHHs)
- UGT1A1 mRNA transfection rescued conjugation (3.75 mg/L), confirming the organoid faithfully models CNS pathology

### Glucocorticoid Regulation of UGT1A1
- **Agonists** (hydrocortisone, dexamethasone) **reduced** bilirubin conjugation (2.03–2.28 mg/L vs. 2.92 mg/L control)
- **Antagonists** (ketoconazole, mifepristone) **enhanced** conjugation (4.66–4.95 mg/L vs. 3.19 mg/L control)
- ChIP-reChIP revealed glucocorticoid receptor recruits MECP2 (transcriptional repressor) to UGT1A1 promoter; antagonists block this recruitment

### RNA-seq & Off-Target Effects
GSEA of mifepristone-treated HLOs revealed:
- **Upregulated**: UGT1A1, BLVRA, SOD1 (xenobiotic metabolism)
- **Downregulated**: OATP2, ABCC2, ABCC3 (transporter proteins)
- **Induced**: IL-6, IL-8, IL-1β, VIM (inflammatory/fibrotic genes)
- **Key insight**: NRF2 — master regulator of detoxification — was NOT upregulated, explaining the off-target effects

### Synthetic Biology Solution
- Introduced murine GULO into iPSCs (Tet-ON inducible, P2A-mCherry reporter)
- Dox-induced eHLOs synthesized intracellular vitamin C, activating NRF2 (4.7-fold upregulation)
- eHLOs showed enhanced bilirubin conjugation (>50% increase) and viability (>75%)
- Reduced inflammation (IL-6 downregulated 0.5-fold) compared to glucocorticoid antagonism

### Transplantation Rescue
- eHLOs transplanted into Gunn rats via portal vein (5 × 10^5 cells)
- Human albumin peaked at 527 ng/mL by day 30, persisted to day 60
- Serum bilirubin dropped ~30%, peaking at day 40
- Therapeutic benefit declined after day 60 (xenorejection + bile autotoxicity)

## Computational Methods

| Analysis | Approach |
|---|---|
| RNA-seq | Differential expression, mifepristone vs. control |
| GSEA | Gene Set Enrichment Analysis for pathway discovery |
| Cross-pathway comparison | Xenobiotic vs. ROS metabolism gene set intersection |
| ChIP-reChIP analysis | NR3C1-MECP2 co-occupancy at UGT1A1 promoter |
| Statistical analysis | R v4.2.0 (ANOVA, Tukey's, Kruskal-Wallis, Dunn-Holland-Wolfe, Brunner-Munzel) |
| Power analysis | G*Power (alpha=0.05, power=0.8) |

## Data & Code Availability

- **RNA-seq data**: NCBI GEO accession **GSE222362**
- **Bioinformatics code**: Published on GitHub at [github.com/hasanwraeth/RNAseq](https://github.com/hasanwraeth/RNAseq)

## Publication

**Reza, H.A.**, Farooqui, Z., Al Reza, A., Conroy, C., Iwasawa, K., Ogura, Y., Okita, K., Osafune, K. & Takebe, T. *Synthetic augmentation of bilirubin metabolism in human pluripotent stem cell-derived liver organoids.* **Stem Cell Reports** 18(11), 2071–2083 (2023). DOI: 10.1016/j.stemcr.2023.09.006.

## Recognition

Recognized as a **Top Scientific Achievement** by Cincinnati Children's Hospital Medical Center (2023). Featured in [Cincinnati Children's Science Blog](https://scienceblog.cincinnatichildrens.org/engineered-organoids-might-serve-as-bridge-to-liver-transplantation-in-severe-jaundice/).

## Significance

This work demonstrates a synthetic biology approach to enhancing metabolic function in stem cell-derived organoids — using genetic engineering to overcome a human-specific gene loss (GULO) that limits ascorbate-dependent detoxification. The engineered organoids served as a bridge therapy in a preclinical model of Crigler-Najjar syndrome, potentially delaying the need for liver transplantation. The approach is generalizable: synthetic metabolic augmentation could enhance other organoid systems where human-specific metabolic pathways are incomplete.
