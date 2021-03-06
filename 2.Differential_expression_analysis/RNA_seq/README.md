## QC and DE for RNA-seq

#### 0. obtain ID for rRNA and mitochondria genes, write out in .bed format.
For preparation for the next step.

RNAseq_0.convert_gtf_to_bed_get_rDNA_mito.Rmd


#### 1. separate reads from rRNA and mitochondria. 
We did this step to (1) count how many reads were rRNA and from mitochondria, to see whether library prep was good quality. (2) remove these reads from downstream analysis, so they don't biase normalization. As germ cells differentiate, their rRNA and mitochondria content change very dramatically.

RNAseq_1.remove_rRNA_mito_reads.Rmd


#### 2. quanlity control
To see how good the replicates are. There are more samples than the paper, they are for another project.

RNAseq_2.all_sample_QC.Rmd


#### 3. Differential expression analysis for different treatment-control pairs. 
This is written so one can specify multiple treatment-control pairs in RNAseq_3.DE_pairwise_master.R, and for each pair, the script will call RNAseq_DE_pairwise.Rmd to do the DE and generate a report. 

RNAseq_3.DE_pairwise_master.R

RNAseq_DE_pairwise.Rmd


#### 4. Normalization across all samples
DE is done in a pair-wise way, here does normalization across all samples. Input information is in: sampleTable_RNA_all.txt

RNAseq_4.normalize_counts_all.Rmd

============== 
Session Info see output .html
