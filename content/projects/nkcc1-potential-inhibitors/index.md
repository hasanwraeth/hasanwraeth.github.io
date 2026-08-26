---
title: "In Silico Identification of Potential NKCC1 Inhibitors with Higher Potency Than Bumetanide for Neurological Disorders"
date: 2021-10-28
summary: "Virtual screening of 1,930 brain-penetrant compounds against NKCC1 ion co-transporter, combining molecular docking, ADMET profiling, and 100 ns molecular dynamics simulations to identify four drug candidates superior to bumetanide"
tags:
  - Computational Drug Discovery
  - Molecular Docking
  - Molecular Dynamics
  - ADMET
  - Neuropharmacology
  - Structural Proteomics
  - Ion Transport
tech_stack:
  - PyRx (Molecular Docking)
  - AutoDock Vina
  - SwissADME
  - admetSAR 2.0
  - ProTox-II
  - MDAnalysis
  - SWISS-MODEL
  - RCSB PDB
  - PubChem Database
links:
  - type: paper
    url: https://doi.org/10.1016/j.imu.2021.100777
    label: Informatics in Medicine Unlocked Paper
  - type: source
    url: https://www.sciencedirect.com/science/article/pii/S2352914821002495
    label: ScienceDirect Article Page
featured: false
status: "Published"
role: "Co-Author — Computational Analysis"
duration: "2020–2021"
team_size: 11
highlights:
  - "Screened 1,930 brain-penetrant compounds against NKCC1 via molecular docking (binding affinities −9.3 to −9.0 kcal/mol)"
  - "Identified 4 top-ranked inhibitor candidates with predicted potency exceeding bumetanide"
  - "ADMET profiling confirmed all 4 candidates as safe (no AMES toxicity, no carcinogenicity)"
  - "100 ns molecular dynamics simulations validated stable binding via RMSD, RMSF, Rg, SASA, and H-bond analysis"
  - "Published in Informatics in Medicine Unlocked (Elsevier), 17 citations"
---

A computational drug discovery study combining virtual screening, molecular docking, ADMET profiling, and molecular dynamics simulations to identify brain-penetrant NKCC1 inhibitors with improved potency and safety over bumetanide for treating neurological disorders.

## Overview

Na⁺-K⁺-2Cl⁻ cotransporter 1 (NKCC1) regulates intracellular ionic homeostasis and cell volume in the brain. Its pathological activation is implicated in ischemic stroke, traumatic brain injury, epilepsy, neonatal seizures, and autism — making NKCC1 a compelling drug target. Bumetanide, a loop diuretic, is the primary NKCC1 inhibitor used in preclinical and clinical studies, but its poor blood-brain barrier penetration and potent diuretic side effects limit therapeutic utility. This study employed an in silico pipeline to screen 1,930 brain-penetrant compounds and identify candidates with superior binding affinity, favorable pharmacokinetics, and stable target engagement compared to bumetanide.

## Key Results

### Virtual Screening and Docking
- **1,930 compounds** pre-filtered for brain-penetrating capacity were docked against NKCC1 using PyRx (AutoDock Vina)
- Binding affinities of the top candidates ranged from **−9.3 to −9.0 kcal/mol**, exceeding bumetanide's docking score
- **4 top-ranked compounds** selected for further validation (PubChem CIDs: 71753382, 5740383, 71692222, 3442850)

### ADMET Profiling
- All 4 candidates passed toxicological filtering with **no AMES toxicity** and **no carcinogenicity**
- Pharmacokinetic properties evaluated using SwissADME and admetSAR 2.0
- Toxicity prediction confirmed via ProTox-II web server

### Molecular Dynamics Validation
- **100 ns MD simulations** performed for each of the 4 candidates plus bumetanide control
- Stability confirmed across five trajectory metrics:

| Metric | Assessment |
|---|---|
| RMSD | Root-mean-square deviation — confirmed structural stability of ligand-protein complexes |
| RMSF | Root-mean-square fluctuation — verified minimal residue-level flexibility at binding site |
| Rg | Radius of gyration — confirmed compactness of protein-ligand complexes |
| SASA | Solvent-accessible surface area — validated burial of binding interface |
| H-bond | Hydrogen bond analysis — confirmed persistent key interactions throughout simulation |

## Computational Methods

| Analysis | Tool / Approach |
|---|---|
| Compound library | PubChem (1,930 brain-penetrant compounds) |
| Protein structure | RCSB PDB / SWISS-MODEL homology modeling |
| Molecular docking | PyRx (AutoDock Vina engine) |
| Pharmacokinetics | SwissADME (drug-likeness, bioavailability) |
| ADMET prediction | admetSAR 2.0 (absorption, distribution, metabolism, excretion, toxicity) |
| Toxicity prediction | ProTox-II (AMES, carcinogenicity, organ toxicity) |
| Molecular dynamics | 100 ns all-atom MD simulations per compound |
| Trajectory analysis | MDAnalysis (RMSD, RMSF, Rg, SASA, H-bond) |
| Control compound | Bumetanide (reference NKCC1 inhibitor) |

## Publication

Roy, A.S., Sawrav, M.S.S., Hossain, M.S., Johura, F.T., Ahmed, S.F., Hami, I., Islam, M.K., **Reza, H.A.**, Bhuiyan, M.I.H., Bahadur, N.M. & Rahaman, M.M. *In silico identification of potential inhibitors with higher potency than bumetanide targeting NKCC1: An important ion co-transporter to treat neurological disorders.* **Informatics in Medicine Unlocked** 27, 100777 (2021). DOI: 10.1016/j.imu.2021.100777.

## Significance

This study demonstrates a complete structure-based drug discovery pipeline — from large-scale virtual screening through ADMET filtering to MD-based binding validation — applied to a clinically relevant neurological target. The multi-stage computational workflow (docking → ADMET → MD simulation) is directly transferable to industry drug discovery settings, where in silico pre-filtering of compound libraries reduces experimental screening costs and accelerates lead identification. The NKCC1 target itself remains actively pursued in neuroscience drug development, with subsequent cryo-EM structures of human NKCC1 bound to bumetanide and furosemide published in Nature (2019) further validating the structural approach used here.
