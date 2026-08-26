---
title: "From Spatial Counts to Cellular Niches"
date: 2025-06-13
summary: "A practical spatial transcriptomics workflow for Visium, Visium HD, Slide-seq, and Xenium data using Seurat, RCTD, CellChat, NicheNet, and image-based segmentation."
tags:
  - Spatial Transcriptomics
  - Visium
  - Xenium
  - Seurat
  - CellChat
  - NicheNet
authors:
  - me
featured: false
---

Spatial transcriptomics connects gene expression to coordinates. That changes the analysis question from “which cells express this gene?” to “where is the expression, which cell types occupy that region, and what neighboring populations might be communicating?”

This workflow combines spot-based, bead-based, and imaging-based assays. It covers loading spatial data, quality control, clustering, image segmentation, cell-type deconvolution, spatially variable genes, niche construction, ligand-receptor inference, and ligand prioritization. Every coding block uses generic variables and paths so it can be adapted to another tissue or platform.

## The analysis map

```text
FASTQ + tissue image
  -> Space Ranger count -> spatial counts + image
  -> coordinate-aware object and QC
  -> normalization, dimension reduction, clustering
  -> segmentation or cell-type deconvolution
  -> spatially variable genes and niches
  -> spatial communication and ligand activity
```

## 1. Generate spatial counts with Space Ranger

For a 10x Visium or Visium HD experiment, Space Ranger converts tissue-associated FASTQ reads and the registered tissue image into a spatial feature-barcode matrix, coordinate files, image assets, and quality-control reports:

```bash
sample_id="sample_a"
fastq_dir="path/to/fastq"
reference_path="path/to/spaceranger_reference"
tissue_image="path/to/tissue_image.tif"
slide_id="slide_id"
capture_area="area_id"
cores=16
memory_gb=64

spaceranger count --id="${sample_id}" --transcriptome="${reference_path}" --fastqs="${fastq_dir}" --sample="${sample_id}" --image="${tissue_image}" --slide="${slide_id}" --area="${capture_area}" --localcores="${cores}" --localmem="${memory_gb}"
```

Use the image, slide identifier, and capture area appropriate for the library and instrument. For Visium HD, follow the installed Space Ranger version's requirements for the image and capture-area arguments. The downstream loading step should point to the generated `outs/` directory, especially the filtered feature-barcode matrix and spatial coordinate assets. Preserve the Space Ranger version, reference package, chemistry, FASTQ manifest, image registration inputs, and command line.

If the FASTQs are still in BCL format, demultiplex them before running `count`. Do not reuse a reference built for a different genome assembly or gene annotation without documenting the change.

## 2. Load the spatial assay

For a 10x spot-based assay, load the output directory at the desired bin size. The bin size is a biological and computational choice: smaller bins provide finer spatial resolution but usually contain fewer transcripts.

```r
library(Seurat)

data_dir <- "path/to/spatial_output"
bin_size <- 8

spatial_obj <- Load10X_Spatial(
  data.dir = data_dir,
  slice = "sample_slice",
  bin.size = bin_size
)

Assays(spatial_obj)
DefaultAssay(spatial_obj) <- "Spatial"
SpatialDimPlot(spatial_obj)
```

For Slide-seq or another coordinate-based assay, construct the object with the count matrix and coordinate table while preserving the same barcode order. For Xenium, use the platform loader and retain the field of view (FOV) and molecule-level information:

```r
image_dir <- "path/to/imaging_output"
spatial_obj <- LoadXenium(image_dir, fov = "fov")
ImageDimPlot(spatial_obj, fov = "fov")
```

Confirm that the genome annotation, coordinate units, image orientation, and assay names are correct before joining metadata.

## 3. Inspect spatial quality

Empty bins, low transcript counts, and low feature counts can dominate a high-resolution dataset. Start with distributions and spatial plots:

```r
spatial_obj <- subset(
  spatial_obj,
  subset = nCount_Spatial > min_counts & nFeature_Spatial > min_features
)

min_counts <- 100
min_features <- 20
VlnPlot(spatial_obj, features = c("nCount_Spatial", "nFeature_Spatial"))
SpatialFeaturePlot(spatial_obj, features = "nCount_Spatial")
```

For an imaging assay, use the assay-specific names and remove cells with zero counts:

```r
spatial_obj <- subset(spatial_obj, subset = nCount_Xenium > 0)
VlnPlot(spatial_obj, features = c("nFeature_Xenium", "nCount_Xenium"))
ImageFeaturePlot(spatial_obj, features = "marker_gene")
```

Thresholds must be chosen from the assay and tissue. A low-density tissue region can legitimately have fewer transcripts than a dense region. Keep a record of removed barcodes and inspect the filter on the tissue image.

## 4. Normalize and cluster

SCTransform is used in the workflows for spot, bead, and imaging assays, followed by PCA, a neighbor graph, UMAP, and clustering:

```r
assay_name <- DefaultAssay(spatial_obj)
spatial_obj <- SCTransform(spatial_obj, assay = assay_name, verbose = FALSE)
spatial_obj <- RunPCA(spatial_obj, npcs = 30, verbose = FALSE)
dimensions <- 1:30
spatial_obj <- FindNeighbors(spatial_obj, reduction = "pca", dims = dimensions)
spatial_obj <- FindClusters(spatial_obj, resolution = 0.3)
spatial_obj <- RunUMAP(spatial_obj, reduction = "pca", dims = dimensions)

DimPlot(spatial_obj, reduction = "umap", label = TRUE)
SpatialDimPlot(spatial_obj, label = TRUE, repel = TRUE)
```

For a very large Visium HD section, sketch-based analysis reduces the cost of model fitting. A subset is clustered first, then the learned reductions and labels are projected back to all bins:

```r
spatial_obj <- SketchData(
  object = spatial_obj,
  ncells = sketch_cells,
  method = "LeverageScore",
  sketched.assay = "sketch"
)
DefaultAssay(spatial_obj) <- "sketch"
spatial_obj <- FindVariableFeatures(spatial_obj)
spatial_obj <- ScaleData(spatial_obj)
spatial_obj <- RunPCA(spatial_obj, assay = "sketch", reduction.name = "pca.sketch")
spatial_obj <- FindNeighbors(spatial_obj, reduction = "pca.sketch", dims = 1:25)
spatial_obj <- FindClusters(spatial_obj, cluster.name = "sketch_clusters")
spatial_obj <- RunUMAP(spatial_obj, reduction = "pca.sketch",
  reduction.name = "umap.sketch", return.model = TRUE, dims = 1:25)
```

Projection is an approximation. Compare projected labels with the original coordinates and inspect whether rare spatial populations are represented in the sketch.

## 5. Identify spatially variable genes

Spatially variable genes should be tested with a method that uses coordinates rather than only expression variance:

```r
spatial_obj <- FindSpatiallyVariableFeatures(
  spatial_obj,
  assay = "SCT",
  selection.method = "moransi",
  features = VariableFeatures(spatial_obj)[seq_len(1000)],
  x.cuts = 100,
  y.cuts = 100
)

spatial_genes <- SpatiallyVariableFeatures(
  spatial_obj, selection.method = "moransi"
)
SpatialFeaturePlot(spatial_obj, features = head(spatial_genes, 6))
```

Spatial autocorrelation can reflect tissue structure, cell density, segmentation artifacts, or technical gradients. Compare spatially variable genes with QC metrics and image features before assigning biological meaning.

## 6. Segment nuclei and aggregate expression

For high-resolution imaging data, a nuclei mask can be created from a stained image with a pretrained segmentation model. The generic Python pattern is:

```python
from csbdeep.utils import normalize
from stardist.models import StarDist2D
from tifffile import imread

image = imread("path/to/stained_image.tif")
image = normalize(image, 5, 95)
model = StarDist2D.from_pretrained("2D_versatile_he")
labels, polygons = model.predict_instances_big(
    image,
    axes="YXC",
    block_size=4096,
    prob_thresh=0.01,
    nms_thresh=0.001,
    min_overlap=128,
    context=128
)
```

Convert predicted polygons into a GeoDataFrame, assign stable IDs, and spatially join expression barcodes to polygons:

```python
import geopandas as gpd
from shapely.geometry import Point, Polygon

polygon_geometries = [
    Polygon([(y, x) for x, y in zip(coords[0], coords[1])])
    for coords in polygons["coord"]
]
polygon_gdf = gpd.GeoDataFrame({
    "cell_id": [f"cell_{i}" for i in range(len(polygon_geometries))]
}, geometry=polygon_geometries)

coordinate_gdf = gpd.GeoDataFrame(
    coordinate_table,
    geometry=[Point(x, y) for x, y in zip(
        coordinate_table["x"], coordinate_table["y"])]
)
joined = gpd.sjoin(coordinate_gdf, polygon_gdf,
                    how="left", predicate="within")
```

Remove barcodes assigned to overlapping polygons and aggregate counts by `cell_id`:

```python
valid = joined.dropna(subset=["cell_id"])
valid = valid[~valid["cell_id"].duplicated(keep=False)]
grouped = expression_adata[valid.index].to_df().groupby(
    valid["cell_id"]
).sum()
```

Segmentation errors propagate into every downstream analysis. Inspect masks overlaid on the image, quantify polygon area, and retain the mapping from original barcode to segmented object.

## 7. Assign cell types with deconvolution

Spot-based assays contain mixtures of cells. RCTD can use a single-cell reference to estimate cell-type composition:

```r
library(spacexr)

reference_counts <- GetAssayData(reference_obj, assay = "RNA", slot = "counts")
reference_groups <- factor(reference_obj$cell_type)
reference_umi <- reference_obj$nCount_RNA
reference <- Reference(reference_counts, reference_groups, reference_umi)

query_counts <- GetAssayData(spatial_obj, assay = "Spatial", slot = "counts")
coordinates <- GetTissueCoordinates(spatial_obj)
coordinates <- coordinates[, c("x", "y")]
query <- SpatialRNA(coordinates, query_counts, colSums(query_counts))

rctd_obj <- create.RCTD(query, reference, max_cores = 8)
rctd_obj <- run.RCTD(rctd_obj, doublet_mode = "doublet")
spatial_obj <- AddMetaData(spatial_obj, rctd_obj@results$results_df)
SpatialDimPlot(spatial_obj, group.by = "first_type")
```

Deconvolution labels are estimates whose accuracy depends on reference quality, gene compatibility, count depth, and cell-type similarity. Inspect confidence and doublet results, and do not treat a mixture estimate as a directly observed single cell.

## 8. Define spatial niches

Once spots or cells have labels, a niche assay can summarize local neighborhoods:

```r
spatial_obj <- BuildNicheAssay(
  object = spatial_obj,
  fov = "fov",
  group.by = "cell_type",
  assay = "niche",
  cluster.name = "niche_cluster",
  neighbors.k = 30,
  niches.k = 5
)

ImageDimPlot(spatial_obj, group.by = "niche_cluster")
table(spatial_obj$cell_type, spatial_obj$niche_cluster)
```

For coordinate tables, niches can also be defined by distance to a focal cell type or gene-expressing region:

```r
nearby_barcodes <- Get_Periphery(
  barcodes = barcode_table,
  celltype = "cell_type_a",
  distance = neighborhood_distance,
  data.dir = data_dir
)
```

Choose neighborhood radius and neighbor count in physical units when possible. A niche is a computational definition of local context, not a fixed anatomical entity.

## 9. Analyze spatial communication with CellChat

CellChat can incorporate expression, cell labels, and spatial distances. For 10x spatial data, obtain the original full-resolution coordinates and the image scale factor rather than assuming spot coordinates are already in micrometers:

```r
expression_matrix <- GetAssayData(spatial_obj, assay = "SCT", slot = "data")
metadata <- data.frame(
  labels = spatial_obj$cell_type,
  sample = spatial_obj$sample_id,
  row.names = colnames(spatial_obj)
)
spatial_locations <- GetTissueCoordinates(spatial_obj)
spatial_locations <- spatial_locations[, c("x", "y")]

spatial_factors <- list(
  ratio = microns_per_pixel,
  tol = distance_tolerance
)

cellchat_obj <- createCellChat(
  object = expression_matrix,
  meta = metadata,
  group.by = "labels",
  datatype = "spatial",
  coordinates = spatial_locations,
  spatial.factors = spatial_factors
)
cellchat_obj@DB <- subsetDB(CellChatDB.human,
  search = "Secreted Signaling")
cellchat_obj <- subsetData(cellchat_obj)
cellchat_obj <- identifyOverExpressedGenes(cellchat_obj)
cellchat_obj <- identifyOverExpressedInteractions(cellchat_obj)
cellchat_obj <- computeCommunProb(cellchat_obj, type = "truncatedMean")
cellchat_obj <- filterCommunication(cellchat_obj, min.cells = 10)
cellchat_obj <- computeCommunProbPathway(cellchat_obj)
cellchat_obj <- aggregateNet(cellchat_obj)
```

Inspect communication in both network and tissue space:

```r
interaction_table <- subsetCommunication(cellchat_obj)
netVisual_circle(cellchat_obj@net$count)
netVisual_circle(cellchat_obj@net$weight)
netVisual_spatial(cellchat_obj, signaling = "SIGNALING_PATHWAY")
cellchat_obj <- netAnalysis_computeCentrality(cellchat_obj, slot.name = "netP")
```

For multiple conditions, create and analyze separate CellChat objects before comparing them. Predicted interactions are hypotheses based on expression and a curated database, not proof of physical contact or signaling activity.

## 10. Prioritize upstream ligands with NicheNet

NicheNet ranks ligands by how well their predicted target programs explain a receiver-cell gene set. The inputs are a receiver population, sender populations, a receiver gene set, a background gene set, and compatible ligand-receptor and ligand-target networks:

```r
library(nichenetr)

receiver_type <- "receiver_type"
sender_types <- c("sender_type_a", "sender_type_b")
receiver_genes <- get_expressed_genes(receiver_type, spatial_obj, pct = 0.01)
sender_genes <- unique(unlist(lapply(
  sender_types, get_expressed_genes, seurat_obj = spatial_obj, pct = 0.01
)))

expressed_receptors <- intersect(unique(lr_network$to), receiver_genes)
potential_ligands <- lr_network |>
  dplyr::filter(to %in% expressed_receptors) |>
  dplyr::pull(from) |>
  unique()
potential_ligands <- intersect(potential_ligands, sender_genes)

receiver_markers <- FindMarkers(
  spatial_obj, ident.1 = receiver_type, min.pct = 0.01
)
genes_of_interest <- rownames(receiver_markers)[
  receiver_markers$p_val_adj <= 0.05 &
    abs(receiver_markers$avg_log2FC) >= 0.25
]
genes_of_interest <- intersect(genes_of_interest,
  rownames(ligand_target_matrix))
background_genes <- intersect(receiver_genes,
  rownames(ligand_target_matrix))

ligand_activity <- predict_ligand_activities(
  geneset = genes_of_interest,
  background_expressed_genes = background_genes,
  ligand_target_matrix = ligand_target_matrix,
  potential_ligands = potential_ligands
)
```

Ranked ligands should be checked against spatial sender expression, receptor localization, CellChat interactions, and the biological condition. A high ligand-activity score means that the model's target program is concordant with the receiver gene set; it does not establish causal signaling.

## Reproducibility and failure checks

- Keep image files, coordinate tables, count matrices, segmentation masks, and derived objects in separate output directories.
- Record platform, bin size, image scale factors, coordinate units, genome annotation, reference cell types, and software versions.
- Verify coordinate orientation and units after every conversion between image, Seurat, GeoPandas, and spatial-analysis objects.
- Inspect low-count bins, empty polygons, overlapping segments, and cells assigned to multiple spatial objects.
- Validate deconvolution with reference markers and confidence fields; do not interpret unknown or doublet labels as definitive cell types.
- Test niche results across reasonable neighbor counts and physical radii.
- Analyze conditions separately for communication comparisons and preserve sample identity for replicate-aware interpretation.
- Treat CellChat and NicheNet outputs as model-based hypotheses requiring orthogonal validation.
- Avoid sending private coordinates to external services without approval.

Spatial analysis is strongest when expression, geometry, and image evidence agree. Preserve those three layers together so a reported niche or interaction can be traced back to the cells, coordinates, and measurements that produced it.
