---
title: "Decoding the LAM Niche Microenvironment via Integrative Single-Cell Multiomics and Spatial Transcriptomics"
date: 2026-06-25
summary: "First comprehensive multiomics atlas of the human LAM niche, integrating sc/snRNA-seq, snATAC-seq, and spatial transcriptomics to reveal three LAM cell subtypes and a structured tumor microenvironment ecosystem"
tags:
  - Single-cell Multiomics
  - Spatial Transcriptomics
  - Lung Disease
  - Tumor Microenvironment
  - Bioinformatics
  - Chromatin Accessibility
tech_stack:
  - sc/snRNA-seq (10x Chromium)
  - snATAC-seq
  - Spatial Transcriptomics (10x Visium / Xenium)
  - Multiomic Integration
  - Transcriptional Network Analysis
  - UMAP & Clustering
  - Multimodal Imaging Validation
links:
  - type: paper
    url: https://doi.org/10.1183/13993003.02049-2025
    label: ERJ Paper
featured: true
status: "Published"
role: "Co-Author — Computational Analysis"
duration: "2023–2026"
team_size: 16
highlights:
  - "First multiomics atlas of the human LAM niche (scRNA-seq + snATAC-seq + spatial transcriptomics)"
  - "Identified three novel LAM cell subtypes: LAMCORE1, LAMCORE2, LAMCORE3"
  - "Discovered spatially resolved LAM-associated fibroblast states driving TGF-β signaling and ECM remodeling"
  - "Published in European Respiratory Journal (IF 23.8), June 2026"
---

The first comprehensive multiomics atlas of the human Lymphangioleiomyomatosis (LAM) niche microenvironment, integrating single-cell/nucleus RNA-seq, single-nucleus ATAC-seq, and spatial transcriptomics to deconvolute the complex cellular architecture of this rare, destructive lung disease.

## Overview

Lymphangioleiomyomatosis (LAM) is a rare, progressive lung disease caused by mutations in TSC1 or TSC2, leading to mTORC1 hyperactivation. While the mTOR inhibitor sirolimus — the only FDA-approved drug — stabilizes lung function in most patients, it does not eliminate LAM cells. This gap underscores a critical need to understand the tumor microenvironment and cellular heterogeneity driving disease progression.

This study provides the first integrated multiomics atlas of the human LAM niche, combining three complementary modalities:
- **Single-cell / single-nucleus RNA-seq** — to profile transcriptomic cell states
- **Single-nucleus ATAC-seq** — to map chromatin accessibility and transcriptional drivers
- **Spatial transcriptomics** — to resolve spatial organization of the LAM niche

## Key Results

### Three Distinct LAM Cell Subtypes
Integrative analysis identified three LAM cell subtypes with distinct transcriptional programs:

| Subtype | Identity | Key Features |
|---|---|---|
| **LAMCORE1** | Canonical LAM | Uterine smooth muscle-like, mTORC1-hyperactive |
| **LAMCORE2** | Novel fibroblast-like | Potent extracellular matrix (ECM) remodeling activity |
| **LAMCORE3** | LAMCORE1 substate | Lower transcriptional activity, protein translation enrichment |

### Spatially Resolved Fibroblast States
Two LAM-associated fibroblast (LAF) states were discovered:
- **LAF-seed** — initiates niche formation
- **LAF-niche** — orchestrates TGF-β signaling, ECM deposition/remodeling, and niche expansion

### Structured Niche Ecosystem
Spatial mapping revealed a layered ecosystem:
- **Core**: LAMCORE1 cells enmeshed with lymphatic endothelium
- **Surrounding ring**: LAFs, LAMCORE2 cells, and reprogrammed immune and epithelial cells

This spatial architecture was validated through multimodal imaging technologies.

### Transcriptional Drivers
snATAC-seq revealed that transcriptomic heterogeneity across LAM subtypes is orchestrated by distinct transcriptional drivers and regulatory networks, providing mechanistic insight into how different LAM cell states are maintained.

## Computational Methods

| Analysis | Modality | Approach |
|---|---|---|
| Cell state identification | sc/snRNA-seq | Clustering, marker gene analysis, subtype annotation |
| Chromatin accessibility | snATAC-seq | Peak calling, motif enrichment, transcription factor activity |
| Multiomic integration | RNA + ATAC | Joint embedding, regulatory linkage between peaks and genes |
| Spatial mapping | Spatial transcriptomics | Spot deconvolution, spatial co-localization analysis |
| Transcriptional networks | Multiomic | Regulatory network inference, driver TF identification |
| Niche architecture | Spatial + imaging | Ecosystem reconstruction, spatial neighborhood analysis |
| Validation | Multimodal imaging | Cross-validation of spatial findings with imaging |

## Publication

Chen, K., Zhao, S., Guo, M., **Reza, H.**, Wagner, A., Cakar, A.C., Jiang, C., Zhang, E., Green, J., Martin, E.P., Wikenheiser, G.A., Wikenheiser-Brokamp, K.A., Perl, A.K., Sinner, D., Yu, J.J. & Xu, Y. *Decoding the Lymphangioleiomyomatosis (LAM) Niche Microenvironment via Integrative Analysis of Single Cell Multiomics and Spatial Transcriptomics.* **European Respiratory Journal** (2026). DOI: 10.1183/13993003.02049-2025.

## Significance

This work establishes the first high-resolution blueprint of the LAM niche microenvironment, revealing novel cell states (LAMCORE2, LAF-seed, LAF-niche) and structured crosstalk patterns that identify promising therapeutic targets beyond mTOR inhibition. The integrative multiomics framework — combining transcriptomics, epigenomics, and spatial resolution — demonstrates a computational approach applicable to other rare disease niche microenvironments.
