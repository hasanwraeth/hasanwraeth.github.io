---
title: "From Aligned Reads to Regulatory Programs with ChIP-seq"
date: 2020-06-23
summary: "A practical ChIP-seq workflow using Trim Galore, BWA, samtools, Picard, deepTools, MACS2, DiffBind, and Bioconductor annotation tools."
tags:
  - ChIP-seq
  - Bioinformatics
  - R
  - MACS2
  - DiffBind
authors:
  - me
featured: false
---

ChIP-seq analysis has two distinct jobs: turn sequencing reads into trustworthy genomic intervals, then interpret those intervals in the context of genes, pathways, and regulatory motifs. The accompanying scripts show that progression from FASTQ files to aligned BAMs, normalized BigWig tracks, MACS2 peaks, differential binding, and peak annotation.

This post is based on a `bash.sh` and `ChIP.R`. The code is a compact research workflow and teaching record rather than a drop-in production pipeline. Several paths and sample variables are project-specific, so they must be made explicit before submitting a large job.

## The workflow

```text
FASTQ -> trimming -> BWA -> filtered/sorted BAM -> duplicate handling
      -> BigWig tracks -> MACS2 peaks -> DiffBind -> annotation and enrichment
```

The shell script is configured as an LSF job with 128 GB of memory, 16 CPUs, and a 24-hour wall-time limit. Adjust those settings for the number and size of samples. The `bash.sh` filename is retained from the source script, but its work is ChIP-seq preprocessing and peak calling.

## Inputs and references

```text
project/
├── SraRunTable2.txt          # comma-delimited sample metadata
├── fastq/                    # input FASTQ files
├── GRCh38_Ensembl_106        # BWA index basename
├── bam/                      # alignment and duplicate-marking outputs
├── bw/                       # BigWig signal tracks
├── macs2/                    # MACS2 peak outputs
├── bash.sh
└── ChIP.R
```

The preprocessing job requires `trim_galore`, `bwa`, `samtools`, `picard`, `bamCoverage`, and `macs2`. The BWA index must use the same assembly and compatible chromosome names as the downstream hg38 annotation packages. The BWA command expects paired-end reads because it uses `$SRR1` and `$SRR2`.

The ChIP.R uses DiffBind, GenomicRanges, rtracklayer, ChIPseeker, the hg38 known-gene transcript database, Ensembl release 105, clusterProfiler, org.Hs.eg.db, enrichplot, msigdbr, goseq, rGREAT, the hg38 BSgenome package, and JASPAR2020.

## 1. Trim and align reads

The shell script reads sample identifiers from the first column of `SraRunTable2.txt`:

```bash
GEO=$(tail -n +2 SraRunTable2.txt | cut -d ',' -f 1)
```

The intended trimming and alignment pattern is:

```bash
mkdir -p fastq.trimmed sam
trim_galore -q 15 --fastqc -o fastq.trimmed/ "$SRR1" "$SRR2"
bwa mem -M -t 4 GRCh38_Ensembl_106 \
  "fastq.trimmed/$SRR1" "fastq.trimmed/$SRR2" \
  > "sam/bwa.hs38.$SRR.sam" 2> "bwa.hs38.$SRR.log.txt"
```

`-M` marks shorter split hits as secondary alignments for Picard compatibility. Keep the BWA logs because mapping rate and alignment quality are essential checks before interpreting peaks.

The current `bash.sh` does not define `$SRR`, `$SRR1`, or `$SRR2`, and passes only `$i.fastq` to Trim Galore even though BWA expects two mates. Define those variables from a validated sample sheet before running the job.

## 2. Filter, sort, and deduplicate BAMs

```bash
mkdir -p bam bam/picard
samtools view -bhS -q 30 "sam/bwa.hs38.$SRR.sam" > "bam/bwa.hs38.$SRR.bam"
samtools sort "bam/bwa.hs38.$SRR.bam" -o "bam/bwa.hs38.$SRR.srt.bam"
samtools index "bam/bwa.hs38.$SRR.srt.bam"

picard MarkDuplicates \
  INPUT="bam/bwa.hs38.$SRR.srt.bam" \
  OUTPUT="bam/bwa.hs38.$SRR.srt.markDup.bam" \
  METRICS_FILE="bam/picard/$SRR.metrics.txt" \
  REMOVE_DUPLICATES=true ASSUME_SORTED=true \
  VALIDATION_STRINGENCY=LENIENT
samtools index "bam/bwa.hs38.$SRR.srt.markDup.bam"
```

The MAPQ 30 filter keeps high-confidence alignments. That threshold is an analysis choice and should be reviewed alongside read length, genome repetitiveness, and library quality. Duplicate removal should be checked against Picard metrics rather than treated as an automatic quality fix.

## 3. Create tracks and call peaks

The script creates CPM-normalized BigWig tracks for IGV:

```bash
mkdir -p bw
bamCoverage -b "bam/bwa.hs38.$SRR.srt.markDup.bam" \
  -o "bw/bwa.hs38.$SRR.bw" --normalizeUsing CPM
```

CPM normalization helps visual comparison across libraries. It does not replace DiffBind normalization or account for an input control.

MACS2 is then run on the duplicate-filtered BAM:

```bash
mkdir -p "macs2/$SRR"
macs2 callpeak -t "bam/bwa.hs38.$SRR.srt.markDup.bam" \
  -f BAM -g 2.7e9 -n "bwa.hs38.$SRR" \
  --outdir "macs2/$SRR" -q 0.05 -B \
  --nomodel --extsize 150
```

The source script uses `-g 10e6` for a toy genome. It must be replaced with the effective mappable genome size for the real organism and reference. `--nomodel --extsize 150` manually supplies a fragment extension because the original dataset was too small for MACS2 to infer a model. Validate that choice against fragment-size information and add treatment/control inputs when the design includes them.

## 4. Build a DiffBind sample sheet

`ChIP.R` discovers duplicate-marked BAMs and MACS2 narrowPeak files:

```r
bams <- dir("bam", "markDup.bam$", full.names = TRUE)
peaks <- dir("macs2", "narrowPeak$", full.names = TRUE, recursive = TRUE)
sample_id <- basename(dirname(peaks))
condition <- sub(".rep.", "", sample_id)
replicate <- sub("^.*rep", "", sample_id)

samples <- data.frame(
  SampleID = sample_id, Condition = condition, Replicate = replicate,
  bamReads = bams, Peaks = peaks, Peakcaller = "macs2",
  PeakFormat = "narrowPeak", ScoreCol = 5
)
write.csv(samples, "DiffBind/sample.csv", row.names = FALSE)
chip <- dba(sampleSheet = "DiffBind/sample.csv")
```

This assumes directories such as `K3.rep1`. Verify that every BAM and peak file refers to the same sample; independent directory listings can silently pair the wrong files. Explicit metadata columns are safer for a production analysis.

## 5. Explore and test differential binding

DiffBind produces correlation and PCA plots before modeling:

```r
pdf("DiffBind/DiffBind.sample.correlation.pdf", width = 9, height = 9)
plot(chip)
dev.off()
pdf("DiffBind/DiffBind.PCA.plot.pdf")
dba.plotPCA(chip, DBA_CONDITION, label = DBA_ID)
dev.off()
```

The count, normalization, contrast, and analysis stages are:

```r
chip <- dba.count(chip)
chip <- dba.normalize(chip)
chip <- dba.blacklist(chip, blacklist = FALSE, greylist = FALSE)
chip <- dba.contrast(chip, minMembers = 1, categories = DBA_CONDITION)
chip <- dba.analyze(chip, bBlacklist = FALSE, bGreylist = FALSE)
chip_db <- dba.report(chip, th = 4)
write.csv(chip_db, "DiffBind/DiffBind.results.csv", row.names = FALSE)
```

The source disables blacklist and greylist filtering because its comments refer to a fish dataset with no blacklist. For human hg38 data, use the appropriate documented artifact blacklist. The `th = 4` report setting is a DiffBind parameter, not a universal significance cutoff.

## 6. Annotate peaks and test pathways

The ChIP.R imports a MACS2 narrowPeak file and annotates it against hg38 genes and transcription start sites:

```r
macs_peaks <- import("SRR1642059_peaks.narrowPeak", format = "narrowPeak")
peak_anno <- annotatePeak(
  macs_peaks, tssRegion = c(-500, 500),
  TxDb = TxDb.Hsapiens.UCSC.hg38.knownGene,
  annoDb = "org.Hs.eg.db"
)
plotAnnoBar(peak_anno)
plotDistToTSS(peak_anno)
upsetplot(peak_anno, vennpie = FALSE)
```

The example defines promoters as 500 bp upstream to 500 bp downstream of a TSS. Annotation depends on the transcript model and chromosome naming style, so convert all inputs consistently and record package versions.

Promoter-associated genes are tested with Gene Ontology and Hallmark gene sets:

```r
peak_anno_gr <- as.GRanges(peak_anno)
promoter_peaks <- peak_anno_gr[peak_anno_gr$annotation == "Promoter", ]
genes_with_peak <- unique(promoter_peaks$geneId)
all_gene_ids <- genes(TxDb.Hsapiens.UCSC.hg38.knownGene)$gene_id

go_result <- enrichGO(gene = genes_with_peak, universe = all_gene_ids,
  OrgDb = org.Hs.eg.db, ont = "BP")
emapplot(pairwise_termsim(go_result), showCategory = 20)

msig_t2g <- msigdbr(species = "Homo sapiens", category = "H")
msig_t2g <- msig_t2g[, c("gs_name", "entrez_gene")]
hallmark <- enricher(gene = genes_with_peak, universe = all_gene_ids,
  TERM2GENE = msig_t2g)
```

The gene universe is a major assumption. All annotated genes are used in the example, but an experiment-specific universe of genes eligible for detection or represented in the tested peak set may be more appropriate. `goseq` is included for length-aware testing, and `rGREAT` submits regions to an external service; do not send private coordinates without approval.

## 7. Profile signal and extract motif sequences

`ChIP.R` uses `rtracklayer` and `EnrichedHeatmap` to read selected BigWig intervals around peak centers rather than loading an entire track. It takes the first 10,000 peaks, extends each center by 2.5 kb, normalizes the signal matrix, and clips the color range at the 99th percentile. This produces a useful signal summary but is not a differential-binding test.

The script also converts MACS2 summits into 100 bp windows, keeps standard hg38 chromosomes, and extracts sequences with `BSgenome.Hsapiens.UCSC.hg38`:

```r
summits <- resize(macs_peaks, 100, fix = "center")
summits <- summits[
  as.character(seqnames(summits)) %in%
    standardChromosomes(BSgenome.Hsapiens.UCSC.hg38)
]
writeXStringSet(getSeq(BSgenome.Hsapiens.UCSC.hg38, summits),
  "SRR1642059.fa")
```

These sequences can be scanned with JASPAR2020 matrices through `TFBSTools`. The assembly, chromosome style, summit coordinate convention, and interval width must agree or motif coordinates can be shifted.

## Checks before running

- The `bash.sh` references `$SRR`, `$SRR1`, `$SRR2`, and `${tag[$i]}` without defining them. Derive them from a validated sample sheet.
- The shell loop passes one FASTQ path to Trim Galore but later aligns two files. Confirm paired-end status and provide both mates consistently.
- Replace MACS2's toy `-g 10e6` with the effective genome size for the actual reference.
- Confirm that BAM and narrowPeak records are paired by sample, not merely discovered in directory order.
- Add and document input/control samples when the experiment contains them.
- Use an appropriate hg38 blacklist; the source script disables blacklist and greylist filtering.
- `ChIP.R` contains hardcoded Windows working directories and example files such as `SRR1642058.bw`; use project-relative or parameterized paths.
- The ChIP.R mixes a fish-data blacklist comment with human hg38 annotation. Verify organism, assembly, and annotation packages before interpretation.
- Interactive calls and live services (`View()`, plotting devices, and GREAT) need replacement for headless or automated runs.

The most useful result is not a single peak count. It is a chain of auditable artifacts: alignment logs, duplicate metrics, indexed BAMs, BigWig tracks, per-sample peak files, DiffBind tables, annotated peaks, enrichment results, and motif sequences. Tying each artifact to an exact reference and parameter set makes the regulatory interpretation reproducible.
