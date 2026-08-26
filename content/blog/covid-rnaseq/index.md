---
title: "From SRA Reads to Shared COVID-19 Response Programs in R"
date: 2019-11-19
summary: "A practical RNA-seq workflow using SRA Toolkit, Salmon, tximport, DESeq2, and pathway enrichment to compare SARS-CoV-2 responses in NHBE and A549 cells."
tags:
  - R
  - RNA-seq
  - Bioinformatics
  - DESeq2
  - Salmon
authors:
  - me
featured: false
---

RNA-seq analysis is less about one magic command than about preserving the connection between raw reads, sample metadata, statistical contrasts, and biological interpretation. This post walks through the workflow in the accompanying scripts:

- [get_sra_robustv2.sh](get_sra_robustv2.sh) downloads SRA runs and converts them to compressed FASTQ files.
- [run_salmon.sh](run_salmon.sh) quantifies the reads against a Salmon transcriptome index.
- [covid_rnaseq_analysis.R](covid_rnaseq_analysis.R) imports transcript estimates and analyzes the NHBE and A549 samples.
- [covid_a549_analysis.R](covid_a549_analysis.R) performs a focused infected-versus-mock analysis for A549 cells.
- [covid_comparison_analysis.R](covid_comparison_analysis.R) compares differential-expression results across the two cell models.

The code is a compact analysis notebook rather than a production pipeline. It is useful for learning the mechanics of bulk RNA-seq analysis, but the input metadata, reference versions, and statistical design should be reviewed before using the workflow for a publication.

## The analysis question

The experiment contains mock and infected samples from two cell models: normal human bronchial epithelial cells (NHBE) and A549 cells. The workflow asks three related questions:

1. Which genes change after infection in NHBE cells?
2. Which genes change after infection in A549 cells?
3. Which genes and fold-change patterns are shared between the models?

The scripts use three biological replicates per condition. The analysis is split by cell model before DESeq2 dispersion estimation, so the primary comparisons are mock versus infected within each model rather than a single model containing an interaction term.

## Inputs and layout

The scripts expect a working directory similar to this:

```text
project/
├── SraRunTable.txt       # SRA run metadata used by the shell scripts
├── SraRunTable2.txt      # metadata selected by the R scripts
├── geo_acc.txt           # GEO accessions, one per line
├── gene_map.csv          # transcript-to-Ensembl-gene mapping
├── get_sra_robustv2.sh
├── run_salmon.sh
└── *.R
```

`gene_map.csv` must contain transcript and gene identifiers in the order expected by `tximport`. The R code reads it without a header and names the columns `esntid` and `ensgid`. The first metadata column must be `Sample Name`; those values are also used as the directories containing each sample's `quant.sf` file.

Before running the analysis, check that the metadata order matches the assumptions in the R code: the first six samples are NHBE and the next six are A549, with three mock samples followed by three infected samples in each group. In a real project, this ordering should be replaced with an explicit sample-to-condition join.

## 1. Download and convert the reads

The SRA script extracts run accessions from the first column of `SraRunTable.txt`, skips entries with an existing compressed FASTQ file, and converts each remaining run:

```bash
tail -n +2 SraRunTable.txt | cut -d ',' -f 1

prefetch SRR_ACCESSION
fastq-dump --gzip --defline-qual '+' SRR_ACCESSION/SRR_ACCESSION.SRA
```

Run it from the directory containing the metadata:

```bash
bash get_sra_robustv2.sh
```

This requires the SRA Toolkit and access to the appropriate download cache. The script uses `fastq-dump`, which is convenient for a small teaching dataset but is slower and less configurable than `fasterq-dump`. Confirm the generated filenames and whether the source data are single-end or paired-end before quantification.

## 2. Quantify transcripts with Salmon

After the FASTQ files are available, `run_salmon.sh` groups runs by the accessions in `geo_acc.txt`, builds a whitespace-separated list of FASTQ files, and runs Salmon in single-end mode:

```bash
salmon quant \
  -i gencode_v35_index \
  -l A \
  -r SAMPLE.fastq.gz \
  -o GEO_ACCESSION
```

The `-l A` option lets Salmon infer the library type. The `-r` option is specifically for single-end reads. For paired-end data, the command must use `-1` and `-2` instead, and the input-building logic must preserve read pairs. The index must be built from the same reference release used to create `gene_map.csv`; here the script names that index `gencode_v35_index`.

```bash
bash run_salmon.sh
```

Each output directory should contain a `quant.sf` file. These files, rather than raw count matrices, are the direct inputs to the R analysis.

## 3. Import estimates and normalize in DESeq2

The R scripts construct a named vector mapping each sample to its Salmon result:

```r
loc <- paste0(pull(sample_table, `Sample Name`), "/quant.sf")
names(loc) <- pull(sample_table, `Sample Name`)

gene_map <- read_csv("gene_map.csv", col_names = c("esntid", "ensgid"))

count_data <- tximport(
  files = loc,
  type = "salmon",
  tx2gene = gene_map,
  ignoreTxVersion = TRUE
)
```

`tximport` summarizes transcript-level abundance into gene-level estimates while retaining the information needed by downstream count-based modeling. DESeq2 then estimates library size factors, gene-wise dispersions, and Wald-test statistics. The scripts expose those stages individually for NHBE and also show the equivalent shortcut:

```r
dds_nhbe <- estimateSizeFactors(dds_nhbe)
dds_nhbe <- estimateDispersions(dds_nhbe)
dds_nhbe <- nbinomWaldTest(dds_nhbe)

# Equivalent combined call for a standard DESeq2 workflow:
# dds_nhbe <- DESeq(dds_nhbe)
```

Variance-stabilizing transformation is used for exploratory plots, not for the differential-expression test itself:

```r
vst_nhbe <- varianceStabilizingTransformation(dds_nhbe)
plotPCA(vst_nhbe, intgroup = "conditions") + theme_bw()
```

PCA, hierarchical clustering, and k-means clustering provide checks for sample structure and possible outliers. A strong separation by cell line is expected when all samples are plotted together, which is why the scripts subsequently analyze NHBE and A549 separately.

## 4. Define the infected-versus-mock contrast

The comparison is explicit in the A549 script:

```r
st_a549$conditions <- factor(
  rep(c("mock", "infected"), each = 3),
  levels = c("mock", "infected")
)

results_a549 <- results(
  dds_a549,
  contrast = c("conditions", "infected", "mock")
)
```

The working significance filter is adjusted *p*-value below 0.05 and absolute log2 fold change above 1:

```r
significant <- results_df[
  !is.na(results_df$padj) &
    results_df$padj < 0.05 &
    abs(results_df$log2FoldChange) > 1,
]
```

The scripts visualize the result with an MA plot, a volcano-style scatter plot, and a heatmap of variance-stabilized expression. The heatmap scales genes by row, which emphasizes relative patterns across samples rather than absolute expression magnitude.

## 5. Annotate genes and test pathways

DESeq2 returns Ensembl gene identifiers. `biomaRt` is used to add chromosome coordinates, strand, biotype, descriptions, and external gene names:

```r
ensembl <- useEnsembl(biomart = "ensembl")
ensembl <- useDataset("hsapiens_gene_ensembl", mart = ensembl)

annotation <- getBM(
  attributes = c(
    "ensembl_gene_id", "chromosome_name", "start_position",
    "end_position", "strand", "gene_biotype", "description",
    "external_gene_name"
  ),
  filters = "ensembl_gene_id",
  values = significant$ensgene,
  mart = ensembl
)
```

The annotated Ensembl identifiers are converted to Entrez identifiers for `clusterProfiler`. Gene Ontology biological-process enrichment uses the tested annotated genes as the universe, which is preferable to treating every human gene as eligible:

```r
ego <- enrichGO(
  gene = selected_entrez,
  OrgDb = org.Hs.eg.db,
  ont = "BP",
  universe = tested_entrez,
  readable = TRUE
)

barplot(ego, showCategory = 20)
dotplot(ego, showCategory = 20)
```

The same workflow can be extended to KEGG, but database identifiers and access requirements should be checked for the installed `clusterProfiler` version.

## 6. Compare the cell models

The comparison script sources both analyses and intersects the annotated gene sets:

```r
common_ensgene <- intersect(
  annotated_df$ensgene,
  a549_anno_df$ensgene
)

common_fc <- left_join(
  nhbe_common,
  a549_common,
  by = "ensgene"
)

cor.test(
  common_fc$log2FoldChange.x,
  common_fc$log2FoldChange.y,
  method = "spearman"
)
```

For genes passing the significance and fold-change thresholds in both analyses, the script calculates the shared set, model-specific sets, a Venn diagram, and a fold-change heatmap. The scatter plot compares effect direction and magnitude: points near the red identity line have similar log2 fold changes in the two models, while points on opposite sides of zero suggest a model-specific response.

Spearman correlation describes rank agreement among the intersected genes; it does not establish that the two experiments have identical effects. A formal cross-model interaction test would require a combined design with a `cell_model:infection` interaction and appropriately modeled batch and covariate structure.

## Packages and external tools

The workflow uses:

- SRA Toolkit: `prefetch` and `fastq-dump`
- Salmon
- R packages: `tidyverse`, `tximport`, `DESeq2`, `biomaRt`, `pheatmap`, `plotly`, `RColorBrewer`, `clusterProfiler`, `org.Hs.eg.db`, and `VennDiagram`

Install Bioconductor packages through BiocManager and record the versions used for a reproducible analysis. Network access is required for SRA downloads and the live Ensembl queries in `biomaRt`; cache the annotation or use a versioned local GTF when reproducibility and long-term reruns matter.

## Checks before running

The original scripts are intentionally concise and have a few assumptions worth making explicit:

- `get_sra_robustv2.sh` checks for output in the current directory, so verify where `fastq-dump` writes files before rerunning a partially completed download.
- `run_salmon.sh` assumes single-end reads and uses simple comma-delimited text parsing. Metadata containing quoted commas needs a proper CSV parser.
- The R scripts assume a fixed sample order and exactly three replicates per condition. Validate this from metadata rather than relying on row position.
- In `covid_a549_analysis.R`, the final KEGG call refers to `ent_gene`; the A549 object created earlier is `ent_gene_a549`. Rename that argument before running the final KEGG enrichment.
- In `covid_comparison_analysis.R`, the fold-change heatmap is built from `common_fc`, so it contains genes shared by the two annotated result tables even though the filter uses the union of the significant sets. Use an explicit join on the intended gene universe if a union heatmap is required.
- Replace interactive calls such as `View()`, `plot.new()`, and `ggplotly()` with saved outputs when running on a headless server.

The central lesson is that each stage should leave an inspectable artifact: downloaded reads, Salmon `quant.sf` files, a documented sample table, DESeq2 result tables, annotations, and enrichment results. Keeping those boundaries clear makes it much easier to diagnose whether a surprising biological result comes from the reads, the quantification, the design matrix, or the interpretation step.
