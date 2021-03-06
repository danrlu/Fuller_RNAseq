## Differential expression (DE) analysis for RNA-seq, ATAC-seq, ChIP-seq and CAGE

Here on is after obtaining raw read counts for genes/genomic regions. 

========================
### Quanlity control (QC)

The things to look for include: is rRNA depletion efficient? are replicates good? is there batch effect if all experiment were done in batches?

========================
### To see changes of the same gene/genomic region across different conditions

The input files for all DE anlaysis is **read counts for genomic regions** for each sample. 

#### 1. To obtain the read count files, one first need **genomic regions (aka features) to count the reads** in. 
  - For RNA-seq, the regions are usualy genes, defined in .gtf and .gff3 files.
  - For ATAC-seq/ChIP-seq, the regions are usually open chromatin/ChIP peaks called by MACS2. **One set of common regions** needs to be used for all samples, so peaks called from different samples needs to be merged first. Here they are combined and overlapping peaks merged into 1. 

    `cat *.narrowPeak | sort -k1,1 -k2,2n | bedtools merge -i - > all.narrowPeak`
 
    - Do NOT use peaks called from 1 condition. This is equivalent of doing DE with a selected set of genes that will strongly bias the DE results.
    - The more samples there are, the wider the merged peaks tend to be. 
    - Alternatively for ATAC-seq, a region of defined width can be used, such as +/- X bp from the gene TSS or +/- X bp from the peak summits. I usually try different ways and see whether the conclusions stay the same.
    
- For CAGE, the regions are TSS clusters built and then combined by CAGEr to become consensus clusters. 
  
#### 2. To count reads in each region 
typical option is HTSeq. It was really slow in my hands (in 2016), so instead:
  - For RNA-seq, STAR does counting on the fly while mapping with `--quantMode` which is equivalent of the default setting of HTSeq.
  - For ATAC-seq/ChIP-seq/CAGE:
  `bedtools coverage -counts -a $region_file -b $mapped_reads_file | awk -v OFS='\t' '{print $4, $5}' > counts.txt`
    - \$mapped_reads_file for ATAC-seq/CAGE is the adjusted.bed that contains the 5' end of reads, for ChIP-seq is the .bam file generated by BWA
    - the last step `awk -v OFS='\t' '{print $4, $5}'` is to format the output file to meet downstream input requirement that would vary from pipeline to pipeline.

#### 3. DE analysis. 
  - Once having the read count table for different samples, DE for RNA-seq/ATAC-seq/ChIP-seq/CAGE can all be done with DESeq2 or EdgeR. They are the foundation of DE analysis. They use slightly different underlying statistical models but are both widely used and accepted. For many packages that are more specialized such as DiffBind or Sleuth, DESeq2 and EdgeR are called within these packages to do the DE analysis. 
  
  - The first step of DE is between sample normalization: https://haroldpimentel.wordpress.com/2014/12/08/in-rna-seq-2-2-between-sample-normalization/
  
  - After normalization, the DE package try to figure out whether the difference in gene expression (or chromatin openness etc) across different conditions are more significant than random fluctuations in gene expression and/or measurement. **Biological replicates are a must**, they are where random fluctuations in gene expression and/or measurement is estimated based on. 
  
  - **DE is deep water. Interpret results with caution**. For example, DESeq2 does between sample normalization assuming majority of the genes do not change between samples and therefore the (roughly) median gene stays the same. So there will be bias if vast majority of genes do change. I highly recommend reading and trying to understand the assumptions behind the statistical models. 
  
  - Our CAGE got strange DE results, so I'm not sure whether DESeq2 can be used for CAGE. Comparing to RNA-seq, there are very few genes that are significantly downregulated. It's possible the statistics for DESeq2 is more optimized for RNA-seq. 
  
  
========================  
### To compare different genes/genomic regions in the same sample

  - RNA-seq: when comparing expression level of different genes in the same sample, one needs to consider the length of gene/transcript (longer genes will have more reads than shorter ones expressed at the same level). The complication is that one gene may have several isoforms, and therefore the 'length of gene' is not straightforward to calculate. I tend to use transcript-level analysis if comparing levels of different genes with Kallisto which returns TPM and makes life a lot easier. More on within sample normalization: https://haroldpimentel.wordpress.com/2014/05/08/what-the-fpkm-a-review-rna-seq-expression-units/

  - CAGE intrinsically measures the number of transcript generated at each promoter region, independent of gene length. If simply comparing expression level of promoters, no within sample normalization is needed. 

  - ATAC-seq signal is really __strongly biased by the sequence preference of the transposase__. So to compare accessibility of different genomic region is much less meaningful (almost meaningless) than comparing the accessibility of the same region across different conditions.
  
  - ChIP-seq: when calling peaks, MACS2 gives a score of the peak which takes into account of the local background (aka how strong the peak is comparing to the surrounding area), which is a good estimate when comparing different genomic locations.
  

