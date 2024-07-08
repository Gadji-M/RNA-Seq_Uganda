# RNA-Seq_Uganda
RNA-seq analysis pipeline to detect major differentially expressed genes associated to pyrethroid resistance escalation in An. funestus in Uganda


Contact: gadji.mahamat@crid-cam.net / gadji.mahamat@fasciences-uy1.cm

##### Hey there 👋 😁
We got some new stuff to analyse and come out with DE genes associated to pyrethroid resistance escalation in Anopheles funestus in Uganda. We are describing here a simple pipeline we used to analyse our RNA-Seq data.

## Contents

1. #### [Quality Control of RNA-Seq data](#section-4)
2. #### [Alignment and statistics](#section-5)
3. #### [Quantification of gene expression](#section-6)
4. #### [Differential Gene Expression Analysis (DGE)](#section-7)
5. #### [Functional Enrichment Analysis](#section-8)
6. #### [Pathway Analysis](#section-9)
7. #### [Visualisation and Interpretation](#section-10)
8. #### [Validation](#section-11)



Requirements:
- FastQC
- MultiQC
- fastp
- STAR, HISAT2 or TopHat
- FeatureCounts, HTSeq or StringTie
- Samtools 1.13 or latest
- Bedtools v2.30.0
- Picard tool
- Varscan
- freebayes v1.3.6
- awk 5.1.0
- SnpEff 5.1d
- R 4.2.3 or latest
- DEseq2, EdgeR or Limma-voom
- Enrichr, DAVID or g\:Profiler
- Ingenuity Pathway Analysis (IPA), KEGG or Reactome,
- Intergrative Genomic Viewer (IGV)

Before beginning the analysis, please install the above packages and clone this repository in your PC using: git clone https://github.com/Gadji-M/RNA-Seq_Uganda/`

# Quality Control of RNA-Seq data <a name="section-4"></a>

Here, we applied the script `Fastq_Quality_check.sh` previously design in [https://github.com/Gadji-M/PoolSeq_OMIcsTouch/](https://github.com/Gadji-M/PoolSeq_OMIcsTouch/) to quality check our data. Please have a look and follow the instructions to run the command.


## Trimming data using [fastp](https://open.bioqueue.org/home/knowledge/showKnowledge/sig/fastp) based on QC reports
Here, we'll use `fastp` to trim reads from RNA-Seq data: 
`fastp -i Path/to/read1 R1_1.fq.gz -I Path/to/read2 R2_2.fq.gz -o Path/to/output/read1 trimmed_R1_1.fq.gz -O Path/to/output/read2 trimmed_R2_2.fq.gz -a read1_adapter_Seq --adapter_sequence_r2 read2_adapter_seq -l 25 -j Path/to/json {sample}.fastp.json -h Path/to/html/ {sample}.fastp.html -w threads`

Note: Please refer to the [fastp](https://open.bioqueue.org/home/knowledge/showKnowledge/sig/fastp) documentation or manual for more details on the options.
