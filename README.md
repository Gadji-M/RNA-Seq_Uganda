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

Before beginning the analysis, please install the above packages and clone this repository in your PC using: git clone https://github.com/Gadji-M/RNA-Seq_Uganda/

## Quality Control of RNA-Seq data <a name="section-4"></a>

Here, we applied the script `Fastq_Quality_check.sh` previously design in [https://github.com/Gadji-M/PoolSeq_OMIcsTouch/](https://github.com/Gadji-M/PoolSeq_OMIcsTouch/) to quality check our data. Please have a look and follow the instructions to run the command.


### Trimming data using [fastp](https://open.bioqueue.org/home/knowledge/showKnowledge/sig/fastp) based on QC reports
Here, we'll use `fastp` to trim reads from RNA-Seq data: 
`fastp -i Path/to/read1 R1_1.fq.gz -I Path/to/read2 R2_2.fq.gz -o Path/to/output/read1 trimmed_R1_1.fq.gz -O Path/to/output/read2 trimmed_R2_2.fq.gz -a read1_adapter_Seq --adapter_sequence_r2 read2_adapter_seq -l 25 -j Path/to/json {sample}.fastp.json -h Path/to/html/ {sample}.fastp.html -w threads`

Note: Please refer to the [fastp](https://open.bioqueue.org/home/knowledge/showKnowledge/sig/fastp) documentation or manual for more details on the options.

## Alignment and statistics <a name="section-5"></a>
Alignment is done using various aligners that have been developed by bioinformaticians. These tools include HISAT2, STAR, Kallisto, etc. The choice of the aligner to use for RNA-Seq alignment depends on various factors and the type of output desired. One commonly used aligner is the splice-aware aligner [STAR](https://github.com/alexdobin/STAR), which is quite fast and can output both transcriptome alignment for differential gene expression (DGE) analysis and genome alignment for visualization. In our case, we used STAR for the alignment of the Uganda raw RNA-seq data to the reference genome, along with a reference transcriptome, to output two alignment files: the genome alignment and the transcriptome alignment. Please feel free to go through the manual available [here](https://github.com/alexdobin/STAR/blob/master/doc/STARmanual.pdf).
One key step before running the alignment is to download the reference genome and index it. You can use **curl** or **wget** to download it. In our case, we have two reference genomes on [VectorBase](vectorbase.org/), the [AfunF3](https://vectorbase.org/common/downloads/release-68/AfunestusFUMOZ/fasta/data/VectorBase-68_AfunestusFUMOZ_Genome.fasta) (FUMOZ) from Mozambique and the latest [AfunGA1](https://vectorbase.org/common/downloads/release-68/AfunestusAfunGA1/fasta/data/VectorBase-68_AfunestusAfunGA1_Genome.fasta) from Gabon.  I preferred using the FUMOZ one here because we have been using it for a while, and the pipeline was already set up. It is easy to work with, whereas using the AfunGA1 is not straightforward as it requires troubleshooting and optimization of some errors. However, it is advisable to use AfunGA1 as it is recent and has improved assembly and annotations.

After downloading the reference genome and the annotation file (i.e., the .gtf file) into a directory, use this command to index it:

`GENOMEDIR=/path/to/indexed/genome`.

`readlength -1`: specifies the length of the genomic sequence around the annotated junction to be used in constructing the splice junctions database. Ideally, this length should be equal to the ReadLength-1, where ReadLength is the length of the reads. For instance, for Illumina 2x150b paired-end reads, the ideal value is 150-1=149. In case of reads of varying length, the ideal value is max(ReadLength)-1. In most cases, the default value of 100 will work as well as the ideal value.

`STAR --runThreadN 8 --runMode genomeGenerate --genomeDir $GENOMEDIR --genomeFastaFiles $GENOMEDIR/genome.fasta --sjdbGTFfile annotation.gtf --sjdbOverhang readlength -1`

Because I had five treatments and four biological replicates for each treatment, totaling 20 samples, I preferred to use loops to align them consecutively and save time for other tasks. Below is the command I used for the alignment, which you can edit as needed:

`find Permethrin/ -type f -name "*_1.fq.gz" | while read -r i; do j="${i/_1.fq.gz/_2.fq.gz}" output_prefix="path/to/raw/fastq_directory/$(basename "$i" _1.fq.gz)" STAR --runThreadN 70 --genomeDir path/star/index/ --readFilesIn "$i" "$j" --readFilesCommand zcat --outFileNamePrefix "${output_prefix}." --quantMode TranscriptomeSAM --outSAMattributes NH HI AS NM MD --outSAMtype BAM SortedByCoordinate > "${output_prefix}_alignment.log" 2> "${output_prefix}_alignment.err"; done`          
