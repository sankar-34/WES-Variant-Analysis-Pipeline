#  WES Variant Analysis Pipeline 
WES variant analysis pipeline (QC, trimming, alignment, variant calling &amp; annotation)

--
## Synopsis
Executed a structured whole-exome sequencing (WES) analysis workflow using real human NGS data. Covered core steps from raw data preprocessing (FastQC, Fastp) to read alignment (BWA), BAM processing (samtools), and variant interpretation (VCF/annotation).

Tools and platforms: Ubuntu (WSL), Galaxy, BWA, samtools, bcftools, FastQC, Fastp

Focus: Command-line operations, bioinformatics file formats, automation-ready workflow structure. Suitable for low-resource systems

##  Objectives
- Gain hands-on experience with NGS data preprocessing and quality control  
- Understand sequence alignment and reference genome indexing  
- Learn file format conversions and data handling in Linux environment  
- Explore variant calling concepts and the purpose of annotation  
- Develop ability to document and execute bioinformatics workflows clearly 

---

## Workflow in WSL/Ubuntu-

| Step                   | Tool / Source           | Sample Command                                                                 | Input → Output Format                         | Comment                                                                                                                                                                                                                                                                                     |
|------------------------|--------------------------|----------------------------------------------------------------------------------|------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **1. Data Upload**     | ENA / NCBI / Manual      | `wget ftp://.../sequence_R1.fq.gz`<br> or `fasterq-dump SRRXXXXXXX`<br> or (manual upload) | `.fq.gz` or `.sra` → `.fastq`                | FASTQ format stores raw read sequences along with base quality scores.<br> Each record has 4 lines: `@Header`, sequence line (A/T/G/C/N), a `+`, and the quality line.<br> The quality string uses ASCII characters to encode Phred scores. <br><br>For Phred+33 encoding:<br> `!` = Q0 (worst),<br> `5` = Q20 (acceptable), <br>`:` = Q25 (good), <br>`?` = Q30 (high), <br> `I` = Q40 (excellent).<br> These scores indicate the probability that a base call is incorrect. |
| **2. QC (raw)**        | `FastQC`                | `fastqc sequence_R1.fq sequence_R2.fq`                                          | `.fastq` → `.html`, `.zip`                   | Raw QC helps detect base quality issues, adapter contamination, or GC bias.<br><br> Skipping QC may lead to alignment errors, false variants, or poor downstream performance.                                                                                                                        |
| **3. Trimming**        | `Fastp`                 | `fastp -i sequence_R1.fq -I sequence_R2.fq -o trimmed_R1.fq -O trimmed_R2.fq`   | `.fastq` → trimmed `.fastq`                  | Trimming removes adapters and low-quality bases.<br><br> Fastp is faster than Trimmomatic, auto-detects adapters, supports quality filtering, and outputs both HTML and JSON reports.<br> <br>It’s efficient for short-read NGS data and works across platforms.                                             |
| **4. QC (post-trim)**  | `FastQC`                | `fastqc trimmed_R1.fq trimmed_R2.fq`                                            | trimmed `.fastq` → `.html`, `.zip`           | Post-trim QC checks whether trimming actually improved the quality.<br><br> Look for improved per-base quality, reduced adapter content, and more uniform GC distribution.<br><br> Skipping this may allow poor-quality reads to persist undetected.                                                         |
| **5. Index Reference** | `bwa`                   | `bwa index hg38.fa`                                                             | `.fa` → indexed files (`.amb`, `.ann`, etc.) | Indexing creates a searchable binary version of the genome so that alignment tools can rapidly map short reads.<br><br> Without indexing, alignment won't work.<br> <br>Reference genomes (e.g., hg38) provide a standard to compare sequences and identify mutations.                                       |
| **6. Align Reads**     | `bwa mem`               | `bwa mem <reference.fa> <R1.fq> <R2.fq> > <output.sam>`                     | trimmed `.fastq` + indexed `.fa` → `.sam`    | Aligns reads to the reference genome to find their most likely origin and detect mismatches (potential variants).<br><br> Alignment helps determine where each read maps in the genome, critical for downstream variant calling.                                                                    |
| **7. Alignment Conversion & QC** | `samtools` | `samtools view -b sample_aligned.sam -o sample_unsorted.bam`<br><br>`samtools sort sample_unsorted.bam -o sample_sorted.bam`<br><br>`samtools index sample_sorted.bam`<br><br>`samtools flagstat sample_sorted.bam` | **Input:** `.sam` → `.bam`<br>**Output:** `.bam`, `.bai`, summary stats | Converts text SAM to BAM, sorts it by coordinates, indexes it for access, and outputs key alignment stats (total, mapped, paired reads etc.) |                                    |
| **8. Index BAM**       | `samtools`              | `samtools index aligned.bam`                                                    | `.bam` → `.bam.bai`                          | BAM indexing allows fast access to specific genome regions.<br><br> It enables tools to extract only the data needed without scanning the full file, making analyses faster and more memory-efficient.                                                                                               |
| **9. Variant Calling** | `bcftools` or `GATK`    | `bcftools call -mv -Ov -o variants.vcf` *(run externally in this case)*         | `.bam` + `.fa` → `.vcf`                      | Variant calling identifies SNPs/INDELs (differences from the reference). <br><br>Tools like GATK (preferred for precision) or bcftools (lightweight) compare aligned reads to reference positions and output variants in VCF format. This is the core goal of the pipeline.                          |
| **10. Annotation**     | `ANNOVAR`, `Ensembl VEP`, `SnpEff` | Depends on tool (CLI or Web)                                           | `.vcf` → annotated `.vcf`, `.tsv`, etc.      | Annotation adds biological interpretation to variants: <br><br>gene name, effect (e.g., missense, silent), population frequency, disease link, etc.<br> <br>ANNOVAR and VEP are widely used.<br><br> Annotation helps identify clinically relevant mutations.                      |


                    

---

 Project Structure


## Skills and Project Highlights-
- Implemented core steps of a WES (Whole Exome Sequencing) variant analysis pipeline  
  — including quality control, trimming, alignment, and data inspection
- Gained hands-on experience with widely-used tools:  
  `FastQC`, `Fastp`, `BWA MEM`, `SAMtools`
- Worked in a Linux (WSL) environment, improving command-line skills and file management  
- Strengthened understanding of `.fastq`, `.bam`, `.vcf` formats and their use in genomics workflows
- Laid groundwork for future automation and scalable variant analysis projects
