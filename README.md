# Comparative Analysis of Variant Calling Pipelines in Breast Cancer

## About This Project

This project compares **6 different NGS pipelines** for detecting somatic mutations in breast cancer. I used the SEQC2 benchmark dataset (HCC1395 cell line) and measured each pipeline's accuracy against a validated ground truth.

**Why?** Different bioinformatics tools can give different results from the same data. Choosing the right pipeline matters for clinical decisions. This project explores how mapper and variant caller choices affect mutation detection.

## Dataset

| Sample | Description | Accession |
|--------|-------------|-----------|
| Tumor | HCC1395 (Triple-negative breast cancer) | SRR7890850 |
| Normal | HCC1395BL (Matched normal B-lymphoblast) | SRR7890851 |

- **Sequencing:** Whole Exome Sequencing (WES), Illumina HiSeq 4000, paired-end 150bp
- **Reference genome:** GRCh38 (hg38)
- **Ground truth:** SEQC2 high-confidence somatic SNVs (39,648 variants)

## Pipelines Tested

| # | Mapper | Variant Caller |
|---|--------|---------------|
| 1 | BWA-MEM2 | Mutect2 (GATK) |
| 2 | BWA-MEM2 | VarScan2 |
| 3 | BWA-MEM2 | SomaticSniper |
| 4 | Bowtie2 | Mutect2 (GATK) |
| 5 | Bowtie2 | VarScan2 |
| 6 | Bowtie2 | SomaticSniper |

## Pipeline Workflow

```
FASTQ (raw reads)
  → fastp (quality trimming + adapter removal)
    → BWA-MEM2 / Bowtie2 (alignment to reference)
      → SAM/BAM (sorted + indexed)
        → MarkDuplicates (GATK)
          → BQSR (Base Quality Score Recalibration)
            → Mutect2 / VarScan2 / SomaticSniper (variant calling)
              → VCF (filtered somatic SNVs)
                → Comparison with ground truth
```

## Evaluation Metrics

- **Precision** — How many of the called variants are real?
- **Recall** — How many of the real variants were found?
- **F1-Score** — Balance between precision and recall
- **Jaccard Index** — Overall overlap with ground truth



## Tools Used

| Tool | Version | Purpose |
|------|---------|---------|
| fastp | 1.1.0 | Read trimming and quality filtering |
| BWA-MEM2 | latest | Read alignment (mapper 1) |
| Bowtie2 | 2.5.4 | Read alignment (mapper 2) |
| GATK | 4.6.2.0 | MarkDuplicates, BQSR, Mutect2 |
| VarScan | 2.4.6 | Somatic variant calling |
| SomaticSniper | latest | Somatic variant calling |
| samtools | 1.23 | BAM processing |
| bcftools | 1.23 | VCF comparison and filtering |
| FastQC | 0.12.1 | Quality reports |
| MultiQC | 1.33 | Aggregated QC reports |

## How to Run

All analysis was done on **Google Colab**. Open the notebooks in order (01 → 04) and follow the instructions inside each one.

**Requirements:**
- Google Colab (Pro recommended for faster runtime)
- Google Drive for storing results
- ~30 GB free disk space on Colab

## Results

### Performance Summary

| Pipeline | Calls | TP | FP | FN | Precision | Recall | F1 |
|----------|------:|---:|---:|---:|----------:|-------:|---:|
| BWA-MEM2 + Mutect2 | 3,094 | 1,132 | 1,962 | 38,428 | 0.3659 | 0.0286 | 0.0531 |
| BWA-MEM2 + VarScan2 | 3,215 | 897 | 2,318 | 38,663 | 0.2790 | 0.0227 | 0.0419 |
| BWA-MEM2 + SomaticSniper | 80,042 | 1,399 | 78,643 | 38,161 | 0.0175 | 0.0354 | 0.0234 |
| Bowtie2 + Mutect2 | 2,708 | 1,075 | 1,633 | 38,485 | 0.3970 | 0.0272 | 0.0509 |
| Bowtie2 + VarScan2 | 2,986 | 886 | 2,100 | 38,674 | 0.2967 | 0.0224 | 0.0416 |
| Bowtie2 + SomaticSniper | 64,301 | 1,350 | 62,951 | 38,210 | 0.0210 | 0.0341 | 0.0260 |

**Best F1-Score:** BWA-MEM2 + Mutect2 (0.0531)

### Key Findings

1. **Mutect2 had the highest precision** across both mappers (~37-40%), meaning it makes fewer false calls compared to VarScan2 and SomaticSniper.

2. **SomaticSniper produced the most calls** (64K-80K) with very low precision (~2%), indicating many false positives. However, it achieved the highest recall (~3.4-3.5%).

3. **BWA-MEM2 slightly outperformed Bowtie2** — BWA-MEM2 + Mutect2 had both higher TP count (1,132 vs 1,075) and better F1-score (0.0531 vs 0.0509).

4. **VarScan2 was in the middle** — moderate precision (~28-30%) but lowest recall (~2.2%).

### Important Note on Low Recall

The recall values are low (~2-3%) because the **ground truth contains 39,648 SNVs from Whole Genome Sequencing (WGS)**, but our data is **Whole Exome Sequencing (WES)** which only covers ~1-2% of the genome (protein-coding regions). Most ground truth variants fall outside exome capture regions, making them impossible to detect with WES data. This is expected and does not indicate poor pipeline performance. The relative ranking between pipelines remains valid for comparison purposes.


## Author

Undergraduate student at Istanbul Technical University (ITU), Department of Molecular Biology and Genetics.
Melisa Agri

## License

This project is for educational purposes.
