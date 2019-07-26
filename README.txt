avana_public_19Q1

This Achilles dataset contains the results of genome-scale CRISPR knockout screens for 17,634 genes in 558 cell lines. It was processed using the following steps:

- Sum raw readcounts by replicate and guide
- Remove the list of guides with suspected off-target activity
- Remove guides with pDNA counts less than one millionth of the pDNA pool
- Remove replicates that fail fingerprinting match to parent or derivative lines
- Remove replicates with total reads less than 15 million
- Remove replicates that do not have a Pearson coefficient > .7 with at least one other replicate for the line
- Calculate log2-fold-change from pDNA counts for each replicate
 - Calculate the SSMD for each cell line using guides targeting the Hart reference essentials and non-essentials, and remove those with values more positive than -0.5. See Hart et al., Mol. Syst. Biol, 2014.
- Calculate the SSMD for each cell line after averaging remaining replicates, and remove those more positive than -0.5
- Run CERES to generate gene-level scores.
- Identify pan-dependent genes as those for whom 90% of cell lines rank the gene above a given dependency cutoff. The cutoff is determined from the central minimum in a histogram of gene ranks in their 90th percentile least dependent line.
- For each CERES gene score, infer the probability that the score represents a true dependency or not. This is done using an EM step until convergence independently in each cell line. The dependent distribution is determined empirically from the scores of the pan-dependent genes. The null distribution is determined from unexpressed gene scores in those cell lines that have expression data available, and from the Hart non-essential gene list in the remainder.

The source for copy number data varies by cell line. Copy number data  indicated as "Sanger WES" are based on the Sanger Institute whole exome sequencing data (COSMIC: http://cancer.sanger.ac.uk/cell_lines, EGA accession number: EGAD00001001039) reprocessed using CCLE pipelines. Copy number source was chosen according to the following logic:
- Broad WES for lines where available
- Broad SNP when Broad WES is not available and Sanger WES not available, or Sanger WES copy number has less correlation with logfold change than Broad SNP
- Sanger WES in all other cases

This dataset also included batch-corrected versions of gene_effect, gene_dependency, and pan_dependent_genes. This batch correction consists of identifying low-dimensional principle components of the data that are strongly related to experimental confounders such as screen quality, pDNA batch, and Cas9 activity and removing them. We find the first six principle components consistently appear the most related to these undesirable confounders and remove them. Specifically:
- Each column in gene_effect is mean-centered and scaled so it had standard deviation 1
- The first six principle components are identified and removed
- Gene means and standard deviations are restored
- gene_effect is renormalized per cell line so positive controls are centered at -1
- gene_dependency and pan_dependent_genes are calculated as above using this corrected gene_effect matrix

More details about this correction and a comparison to other common data correction methods will be published on bioarxiv in winter 19Q1.

*****************
Dataset contents:
*****************

## Post-CERES files:

README - Raw

gene_effect - NumericalMatrix
CERES data normalized to positive controls. 
Columns: genes in the format  “HUGO (Entrez)”
Rows: cell lines (Broad IDs)

gene_effect_corrected - NumericalMatrix
CERES data with principle components strongly related to known batch effects removed. 
Columns: genes in the format  “HUGO (Entrez)”
Rows: cell lines (Broad IDs)

gene_dependency - NumericalMatrix
Probability that knocking out the gene has a real depletion effect.
Columns: genes in the format  “HUGO (Entrez)”
Rows: cell lines (Broad IDs)

gene_dependency_corrected - NumericalMatrix
Probability that knocking out the gene has a real depletion effect using gene_effect_corrected.
Columns: genes in the format  “HUGO (Entrez)”
Rows: cell lines (Broad IDs)

guide_efficacy - Table
Columns:
sgrna (nucleotides)
efficacy - CERES inferred efficacy for the guide

pan_dependent_genes - Table
List of genes identified as dependencies in all lines, one per line. The scores of these genes are used as the dependent distribution for inferring dependency probability.

pan_dependent_genes_corrected - Table
List of genes identified as dependencies in all lines, one per line, using gene_effect_corrected. The scores of these genes are used as the dependent distribution for inferring dependency probability.


## Pre-CERES files
essential_genes - Table
List of genes used as positive controls, currently the 217 Hart panessentials in the format “HUGO (Entrez)”. Each entry is separated by a newline.

nonessential_genes - Table
List of genes used as negative controls (Hart nonessentials) in the format “HUGO (Entrez)”. Each entry is separated by a newline.

raw_readcounts - NumericalMatrix
Summed counts for each replicate/PDNA
Columns: replicate/pDNA IDs 
Rows: Guides (nucleotides)

logfold_change - NumericalMatrix
Post-QC log2-fold change (not ZMADed)
Columns: replicate IDs
Rows: Guides (nucleotides)

guide_gene_map - Table
Columns:
sgrna (nucleotides) - appears more than once
genome_alignment
gene (“HUGO (Entrez)”)
n_alignments (integer number of perfect matches for that guide)

copy_number - Table
Segmented copy number data for included lines
Columns:
Broad_ID
Chromosome (integer, X, Y)
Start (bp)
End (bp)
Num_Probes
Segment_Mean (logfold change from average)

replicate_map - Table
Columns:
replicate_ID (str)
Broad_ID
pDNA_batch (int): indicates which processing batch the replicate belongs to and therefore which pDNA reference it should be compared with.

dropped_guides - Raw
Guides dropped for suspected off-target activity, separated by newlines.

## Annotations

sample_info - Table
Columns:
Broad_ID
CCLE_name (“ID_PRIMARYSITE”)
n_replicates (int): number of replicates surviving QC
primary_tissue (str): primary tissue of origin
secondary_tissue (str): secondary tissue of origin
tertiary_tissue (str): tertiary tissue of origin
cas9_activity (float): percentage score, lower is better
culture_medium (str): annotation of basic culture
culture_type (str): “adherent” or none
cell_line_SSMD (float): Difference between positive and negative controls
aliases (string): alternate names for the cell line
