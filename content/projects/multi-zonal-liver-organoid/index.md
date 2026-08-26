---
title: "Multi-Zonal Liver Organoids from Human Pluripotent Stem Cells"
date: 2025-04-16
summary: "Self-assembling zone-specific liver organoids that replicate hepatic portal-central zonal architecture, validated by single-nucleus RNA-seq, ChIP-seq, and transplantation in a rat liver-failure model"
tags:
  - Single-cell Genomics
  - Stem Cell Biology
  - Organoids
  - Epigenomics
  - Bioinformatics
  - Regenerative Medicine
tech_stack:
  - snRNA-seq (10x Chromium)
  - ChIP-seq
  - RNA Velocity
  - 10x Xenium Spatial Transcriptomics
  - UMAP & Clustering
  - Pathway Enrichment
  - Pseudotime Trajectory Analysis
links:
  - type: paper
    url: https://www.nature.com/articles/s41586-025-08850-1
    label: Nature Paper
  - type: dataset
    url: https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE222654
    label: GEO Dataset
featured: true
status: "Published"
role: "First Author — Bioinformatics Lead"
duration: "2019–2025"
team_size: 8
highlights:
  - "First in vitro model to replicate liver zonal architecture (portal-central axis)"
  - "Led all bioinformatics: snRNA-seq, ChIP-seq, RNA velocity, spatial transcriptomics"
  - "Transplantation improved survival in a rat bile duct ligation liver-failure model"
  - "Published in Nature (IF 50.5), April 2025"
---

A self-assembling multi-zonal human liver organoid (mZ-HLO) system that recreates the spatially segregated hepatocyte subpopulations along the liver's portal-central axis — a structural feature never before replicated in vitro. Developed as first author with bioinformatics leadership.

## Overview

The liver's hepatocytes are spatially organized into zones along the portal-central axis, each with distinct metabolic functions (urea cycle, glutathione synthesis, drug metabolism). This zonal architecture is critical to understanding metabolic homeostasis and injury, but no in vitro model had successfully replicated it. We developed a self-assembling organoid system by co-culturing ascorbate-enriched (zone 1 / periportal) and bilirubin-enriched (zone 3 / pericentral) hepatic progenitors derived from human induced pluripotent stem cells (hiPSCs).

## My Role

As first author, I designed and performed the research, led all bioinformatics analyses, and wrote the paper. My computational work spanned:

- **Single-nucleus RNA-seq pipeline**: Processing, clustering, and cell-type annotation of zonally patterned organoids, identifying a hepatoblast differentiation trajectory yielding periportal, interzonal, and pericentral hepatocytes
- **Epigenomic analysis**: ChIP-seq for EP300 binding, revealing that zonal identity is orchestrated by ascorbate-dependent EP300-TET1 binding (zone 1) and bilirubin-dependent EP300-HIF1α binding (zone 3)
- **Trajectory inference**: RNA velocity and pseudotime analysis to reconstruct the hepatoblast-to-zonal-hepatocyte differentiation dynamics
- **Spatial transcriptomics**: Validation of zonal marker expression (TAT, HAMP, CYP3A4) against 10x Xenium healthy human liver data
- **Cross-dataset integration**: Benchmarking organoid zonal profiles against primary human liver snRNA-seq datasets (Andrews et al., Human Cell Atlas)

## Key Results

### Zonal Patterning
Preconditioning hepatic progenitors with ascorbate or bilirubin generated zone-specific organoids (Z1-HLOs and Z3-HLOs) with distinct functional profiles:
- **Zone 1 (periportal)**: Urea cycle genes (CPS1, ASL, OTC), glutamate synthesis (GLS2)
- **Zone 3 (pericentral)**: Glutathione synthesis (GLUL), drug metabolism (CYP3A4, CYP1A2)

### Self-Assembly into Multi-Zonal Organoids
Co-culturing Z1 and Z3 organoids at high density with continued bilirubin treatment drove organoid fusion via NOTCH signaling, producing self-assembled multi-zonal organoids (mZ-HLOs) with spatially segregated zonal populations.

### Epigenetic Mechanism
ChIP-seq and ChIP-reChIP revealed that EP300 acts as a shared co-activator with zone-specific partners:
- **Ascorbate → EP300 + TET1** → zone 1 gene activation
- **Bilirubin → EP300 + HIF1α** → zone 3 gene activation

### Transplantation Rescue
Transplantation of mZ-HLOs into immunodeficient RRG rats subjected to bile duct ligation improved survival by ameliorating hyperammonaemia and hyperbilirubinaemia — demonstrating therapeutic potential for liver failure.

## Computational Methods

| Analysis | Tool / Approach |
|---|---|
| snRNA-seq processing | 10x Chromium, Cell Ranger |
| Clustering & visualization | UMAP, Leiden clustering |
| Cell-type annotation | Marker gene scoring, dot plots, violin plots |
| Trajectory inference | RNA velocity, pseudotime ordering |
| Epigenomic profiling | ChIP-seq peak calling, genome browser visualization |
| ChIP-reChIP | EP300-TF co-occupancy analysis |
| Spatial validation | 10x Xenium, pseudo-spatial profiling |
| Cross-dataset integration | Seurat/Scanpy integration with primary liver atlases |
| Pathway enrichment | Gene Ontology, pathway network analysis |

## Data Availability

All sequencing data deposited at NCBI GEO under accession **GSE222654**. Public datasets integrated from seven independent studies (GSE96981, GSE154883, GSE245379, GSE207889, GSE141183, GSE188541, PRJNA239635) and the Human Cell Atlas liver development collection.

## Publication

**Reza, H.A.**, Santangelo, C., Iwasawa, K., Al Reza, A., Sekiya, S., Glaser, K., Bondoc, A., Merola, J. & Takebe, T. *Multi-zonal liver organoids from human pluripotent stem cells.* **Nature** 641, 1258–1267 (2025).

## Significance

This work establishes the first in vitro model that recapitulates hepatic zonal architecture, providing a platform for studying liver development, zone-specific drug toxicity, and regenerative medicine applications. The organoid system bridges a long-standing gap between 2D hepatocyte cultures and in vivo liver complexity.
