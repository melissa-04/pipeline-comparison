# Comparing Variant Calling Pipelines in Breast Cancer

## What is this project?

I compared 6 different bioinformatics pipelines to find somatic mutations in breast cancer data. The goal was to see how the choice of tools (mapper + variant caller) affects the results. I used the SEQC2 benchmark dataset which has a validated list of real mutations, so I could measure how well each pipeline performs.


## Dataset

I used the SEQC2 HCC1395 dataset. HCC1395 is a triple-negative breast cancer cell line. There are two samples:

- **Tumor (SRR7890850):** DNA from the cancer cells. Contains both normal and cancer-specific mutations mixed together.
- **Normal (SRR7890851):** DNA from healthy cells of the same patient. Only has inherited (germline) mutations.

By comparing tumor vs normal, we can find mutations that exist only in the tumor. These are called somatic mutations.

The sequencing type is Whole Exome Sequencing (WES), which means only the protein-coding regions (~1-2% of the genome) were sequenced. The reference genome is GRCh38 (hg38).

## Pipelines I tested

I used 2 mappers and 3 variant callers, making 6 combinations:

**Mappers (alignment tools):**
- BWA-MEM2 — fast, uses Burrows-Wheeler Transform
- Bowtie2 — uses FM-index, different algorithm

**Variant Callers (mutation detection tools):**
- Mutect2 (GATK) — Bayesian model, considered the gold standard for somatic calling
- VarScan2 — uses Fisher's exact test on pileup data
- SomaticSniper — calculates genotype likelihoods

So the 6 pipelines are: BWA-MEM2+Mutect2, BWA-MEM2+VarScan2, BWA-MEM2+SomaticSniper, Bowtie2+Mutect2, Bowtie2+VarScan2, Bowtie2+SomaticSniper.

## How the pipeline works

Each pipeline follows the same steps:

1. **Quality control:** fastp removes low-quality reads and adapter sequences
2. **Mapping:** BWA-MEM2 or Bowtie2 aligns reads to the reference genome, producing BAM files
3. **Post-processing:** MarkDuplicates removes PCR duplicates, BQSR corrects base quality scores
4. **Variant calling:** Mutect2, VarScan2, or SomaticSniper detects somatic mutations
5. **Filtering:** Only high-confidence SNVs (single nucleotide variants) are kept
6. **Comparison:** Each pipeline's results are compared against the SEQC2 ground truth
7. 
## Results

The ground truth from SEQC2 contains 39,648 validated somatic SNVs.

Here is what each pipeline found:

- **BWA-MEM2 + Mutect2:** 3,094 calls, 1,132 true positives, precision 0.37, recall 0.03, F1 0.053
- **BWA-MEM2 + VarScan2:** 3,215 calls, 897 true positives, precision 0.28, recall 0.02, F1 0.042
- **BWA-MEM2 + SomaticSniper:** 80,042 calls, 1,399 true positives, precision 0.02, recall 0.04, F1 0.023
- **Bowtie2 + Mutect2:** 2,708 calls, 1,075 true positives, precision 0.40, recall 0.03, F1 0.051
- **Bowtie2 + VarScan2:** 2,986 calls, 886 true positives, precision 0.30, recall 0.02, F1 0.042
- **Bowtie2 + SomaticSniper:** 64,301 calls, 1,350 true positives, precision 0.02, recall 0.03, F1 0.026

Best F1-score: BWA-MEM2 + Mutect2 (0.053)

## What I observed

**Mutect2 has the best precision.** It makes fewer false calls (~37-40% of its calls are real). This makes sense because Mutect2 uses a sophisticated Bayesian model to distinguish real mutations from sequencing errors.

**SomaticSniper produces way too many calls.** It found 64,000-80,000 variants while the others found around 3,000. Most of these are false positives (precision only ~2%). However, it catches slightly more real mutations than the others.

**The mapper choice does not make a big difference.** BWA-MEM2 is slightly better than Bowtie2, but the difference is small. This is consistent with the findings in Ertürk et al. (2025), where they also found that the mapper has minimal impact compared to the variant caller.

**VarScan2 is in the middle.** Moderate precision, lowest recall.

## Why is the recall so low?

This is important to explain. My recall values are around 2-3%, which looks very low. But there is a reason for this. The ground truth contains 39,648 SNVs from **Whole Genome Sequencing (WGS)**. This means it includes mutations from everywhere in the genome. But my data is **Whole Exome Sequencing (WES)**, which only covers about 1-2% of the genome (the protein-coding exons). So most of the 39,648 ground truth mutations are in regions that my WES data simply cannot see. These all count as false negatives, which makes recall very low. In the Baysan et al. (2025) paper, they applied exome filtering to the ground truth, reducing it from 39,648 to 1,161 SNPs (only the ones inside exome regions). With this filtering, their recall values were around 50-70%. I did not apply this exome filtering step, which is why my recall numbers are not directly comparable to the paper. The pipeline ranking and relative comparison between tools is still valid, but the absolute recall and F1 values would be much higher with proper exome filtering. This is something I plan to fix.
## Tools and versions

- fastp 1.1.0 (read trimming)
- BWA-MEM2 (read alignment)
- Bowtie2 2.5.4 (read alignment)
- GATK 4.6.2.0 (MarkDuplicates, BQSR, Mutect2)
- VarScan 2.4.6 (variant calling)
- SomaticSniper (variant calling)
- samtools 1.23 (BAM processing)
- bcftools 1.23 (VCF filtering and comparison)
- FastQC 0.12.1 and MultiQC 1.33 (quality reports)

Everything was installed via Conda (Miniforge) on Google Colab.



## Reference

https://doi.org/10.1371/journal.pcbi.1013552

## About me

I am an undergraduate student at Istanbul Technical University, Department of Molecular Biology and Genetics (3rd year). This project was done independently to learn NGS analysis and to understand how different pipeline choices affect somatic mutation detection. Melisa Agri
