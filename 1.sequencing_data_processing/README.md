## Here are the original scripts used to process raw sequencing data for the paper that does the following:

### **Raw sequencing data .fastq -> .bw for genomic browser viewing** 

### **Raw sequencing data .fastq -> read counts for downstream analysis**

#### A few notes:
- Put all .fastq files and the scripts into the same folder unless otherwise noted.
- Make sure to change the related parameters before running, especially number of jobs for array jobs. One can also supply these parameters while submitting the job, but having them within the scripts keeps a record.

    SGE version (starts from 1)
    `#$ -t 1-9`

    SLURM version (starts from 0)
    `#SBATCH --array=0-8`

- PCR duplicates are removed for ATAC-seq and ChIP-seq, if both ends of the read pair map to the exact same genomic positions (I highly recommend paired-end sequencing for them). PCR duplicates are not removed for RNAseq because read pairs mapped to the same genomic location could still flank different splicing variants. **In general it is better to use as much input material, and as few PCR cycles as possible to increase library complexity.**

- CAGE protocol involved no PCR amplification step. 

- All multimapping reads were removed for quantification or viewing in genomic browser. I also tried a version (for all experiment) to assign each multimapping reads to 1 randomly selected location and it didn't seem to affect the genes we are interested in. 
