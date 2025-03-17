# Trimming FASTQ Files

This folder explains how to trim your FASTQ files based on:
1. **Quality Control Results** (from FastQC or another QC tool).
2. **Known Adapter Sequences** listed in `adapters.fa`.

By trimming reads, you remove low-quality regions and adapter contamination, improving the accuracy of downstream analyses such as alignment or assembly.

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Step 1: Review FASTQC Results](#step-1-review-fastqc-results)
4. [Step 2: Choose Your Trimming Tool](#step-2-choose-your-trimming-tool)
5. [Step 3: Trimming Single-End Reads](#step-3-trimming-single-end-reads)
6. [Step 4: Trimming Paired-End Reads](#step-4-trimming-paired-end-reads)
7. [Troubleshooting](#troubleshooting)
8. [Further Reading](#further-reading)

---

## Overview

After running FastQC on your raw sequencing data, you may find:
- Low-quality bases at the start or end of reads.
- Overrepresented adapter sequences.
- Other artifacts affecting data quality.

Trimming helps remove these unwanted bases, ensuring your final reads are high-quality and free from adapter contamination. This process is **generalized** and applies to any type of adapter or any quality threshold you decide on, based on your FastQC report.

---

## Prerequisites

1. **FASTQC or Another QC Tool**  
   Already used to assess read quality. You should have a report indicating:
   - Where quality scores drop.
   - Whether adapter contamination is present.
   - Read length distribution.

2. **Trimming Software**  
   Commonly used tools include:
   - [Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic)
   - [Cutadapt](https://cutadapt.readthedocs.io/en/stable/)
   - Others (e.g., BBDuk, FASTP) depending on your preference.

3. **Adapter File (`adapters.fa`)**  
   A FASTA file listing the adapter sequences to be removed. This file may be named differently depending on your library prep kit (e.g., `NexteraPE-PE.fa`, `TruSeq3.fa`), but we’ll refer to it generally as `adapters.fa`.

---

## Step 1: Review FASTQC Results

Before trimming, open your FASTQC HTML report(s) and check for:
1. **Per Base Sequence Quality**: Identify where (if at all) the quality drops off.
2. **Adapter Content**: If FastQC flags adapter contamination, confirm which adapters are overrepresented.
3. **Overrepresented Sequences**: Check if certain sequences (often adapters) appear frequently.

Decide on trimming parameters such as:
- **Minimum Quality Threshold** (e.g., Q20, Q30).
- **Minimum Read Length** (to keep or discard short reads).
- **Adapter Sequences** to remove.

---

## Step 2: Choose Your Trimming Tool

You can use **Trimmomatic**, **Cutadapt**, or another trimming program. The syntax varies slightly, but each has similar capabilities:
- Removing adapters using a FASTA file.
- Trimming low-quality bases at either end of reads.
- Dropping reads below a certain length.

Below are example commands for **Trimmomatic**—the concepts apply similarly to other tools.

---
Below is a detailed explanation of three tools/parameters often used alongside or after FastQC analysis:

- **SLIDINGWINDOW** (a Trimmomatic parameter)  
- **FLASH** (for merging paired-end reads)  
- **fastuniq** (for removing duplicate reads)

We’ll also cover how to choose parameters and what to look for in the FastQC reports to guide your trimming and merging decisions.

---

## 1. SLIDINGWINDOW in Trimmomatic

### What Is It?
`SLIDINGWINDOW` is a parameter in [Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic) that scans your reads in a sliding window of fixed size, checking the average quality within that window. If the average quality falls below a specified threshold, the read is truncated at the beginning of that window.

### Typical Syntax
```
SLIDINGWINDOW:<windowSize>:<requiredQuality>
```
- **windowSize**: How many bases to include in each sliding window.
- **requiredQuality**: The average quality threshold below which trimming will occur.

A common example is `SLIDINGWINDOW:4:20`:
- Scans in windows of 4 bases.
- If the average Phred score < 20 in any window, Trimmomatic trims the read at that position (and discards everything after).

### How to Choose the Parameters
1. **Check “Per Base Sequence Quality” in FastQC**  
   - Look at how quickly (and where) quality drops. If most of the read is high quality but the last 10-20 bases plummet, a smaller window (e.g., 4 or 5) with a moderate threshold (Q20–Q30) can remove those poor bases without over-trimming.
2. **Balance Read Length vs. Quality**  
   - If you set the quality threshold too high (e.g., 30), you might lose more reads/bases. If you set it too low, you might keep poor-quality regions.
3. **Experiment and Re-check**  
   - After choosing a window size and threshold, run Trimmomatic, then run FastQC again on the trimmed data to see if quality has improved without losing too much data.

---

## 2. FLASH (Fast Length Adjustment of SHort reads)

### What Is It?
[FLASH](https://ccb.jhu.edu/software/FLASH/) is a tool designed to merge overlapping paired-end reads. If your insert size is smaller than the combined length of Read 1 (R1) and Read 2 (R2), the forward and reverse reads will overlap. FLASH stitches them into a single read, which can be helpful for:

- Amplicon sequencing (e.g., 16S rRNA studies).
- Short-insert libraries where overlap is common.

### Typical Syntax
```
flash R1.fastq.gz R2.fastq.gz -o merged -M 250
```
- **-o merged**: Output prefix (e.g., `merged.extendedFrags.fastq`).
- **-M 250**: Maximum overlap (in bases). Adjust based on your expected fragment lengths.

Other parameters include:
- **-m** (minimum overlap),
- **-x** (allowable mismatch ratio in the overlap),
- etc.

### How to Choose Parameters
1. **Estimated Insert Size**  
   - If you know your library has an average insert of ~200 bp, and each read is 150 bp, it’s likely you’ll get a substantial overlap (100 bp or more).
   - Set `-M` (max overlap) just above your expected overlap (e.g., 250 bases).
2. **Check “Sequence Length Distribution” in FastQC**  
   - If your library has a very wide distribution or is significantly longer than your reads, overlaps might be small or nonexistent.
   - If you see many shorter fragments, you can expect more overlap and a higher merge rate.
3. **Mismatch Tolerance**  
   - The `-x` parameter controls how tolerant FLASH is of mismatches in the overlapping region. If your quality is good, you can be stricter (lower mismatch ratio). If quality is lower, allow a slightly higher mismatch ratio.

---

## 3. fastuniq

### What Is It?
[fastuniq](https://sourceforge.net/projects/fastuniq/) is used to **remove duplicate reads** in paired-end data. It identifies reads that appear more than once (exact duplicates) and removes them, reducing potential PCR or optical duplicates.

### Why Use It?
- **High Duplication**: If you see in FastQC’s “Sequence Duplication Levels” module that a large proportion of reads are duplicates, you may consider removing them.  
- **Lower Bias**: Duplicate removal can prevent artificial inflation of coverage in downstream analyses (like variant calling).

### Typical Syntax
1. Create an input list for fastuniq (e.g., `fastuniq_input.txt`) containing the path to your R1 and R2 FASTQ files:
   ```
   R1.fastq.gz
   R2.fastq.gz
   ```
2. Run fastuniq:
   ```bash
   fastuniq -i fastuniq_input.txt -t q -o R1_dedup.fastq.gz -p R2_dedup.fastq.gz
   ```
   - **-i**: The list of files.
   - **-t q**: Data type (q = fastq).
   - **-o / -p**: Output files for R1 and R2 after removing duplicates.

### How to Choose Parameters
- **Check FastQC “Sequence Duplication Levels”**  
  - If duplication is very high (e.g., >50%), removing duplicates can significantly reduce your dataset but improve uniqueness.
  - If duplication is low (<10%), you might skip removing duplicates unless it’s required by your analysis.
- **Downstream Goals**  
  - Some pipelines prefer duplicate marking at the alignment stage (e.g., using `samtools markdup` or `Picard MarkDuplicates`), rather than removing them in FASTQ form. Clarify your analysis goals before deciding.

---

## Putting It All Together with FastQC

**Before** using SLIDINGWINDOW, FLASH, or fastuniq, you’ll typically run FastQC to see:
- Where quality declines (use **SLIDINGWINDOW** or other Trimmomatic parameters to trim).
- Whether reads overlap significantly (use **FLASH** to merge pairs if appropriate).
- Whether duplication is high (use **fastuniq** to remove duplicates if needed).

**After** applying these tools, it’s a good idea to run FastQC again to confirm:
- Quality scores are improved (no more low-quality tails).
- Adapter contamination is gone.
- Sequence duplication is reduced if you removed duplicates.
- Merged reads look reasonable (if you used FLASH).

---

## Example Workflow

1. **Run FastQC** on raw files:
   ```
   fastqc *.fastq.gz
   ```
2. **Trim** low-quality bases and adapters (Trimmomatic):
   ```
   trimmomatic PE -threads 4 \
     sample_R1.fastq.gz sample_R2.fastq.gz \
     sample_R1_paired.fastq.gz sample_R1_unpaired.fastq.gz \
     sample_R2_paired.fastq.gz sample_R2_unpaired.fastq.gz \
     ILLUMINACLIP:adapters.fa:2:30:10 SLIDINGWINDOW:4:20 MINLEN:50
   ```
3. **Merge Overlapping Reads** (FLASH), if insert size is smaller than read lengths:
   ```
   flash sample_R1_paired.fastq.gz sample_R2_paired.fastq.gz -o sample_merged -M 250
   ```
4. **Remove Duplicates** (fastuniq), if duplication is high:
   ```
   # 1) Create a file list
   echo "sample_R1_paired.fastq.gz" > fastuniq_input.txt
   echo "sample_R2_paired.fastq.gz" >> fastuniq_input.txt

   # 2) Run fastuniq
   fastuniq -i fastuniq_input.txt -t q -o sample_R1_dedup.fastq.gz -p sample_R2_dedup.fastq.gz
   ```
5. **Re-run FastQC** on the final FASTQ files to confirm improvements.

---

## Summary

- **SLIDINGWINDOW**: A Trimmomatic parameter that trims low-quality regions in a step-wise manner.
- **FLASH**: Merges paired-end reads that overlap, commonly used for short-insert or amplicon libraries.
- **fastuniq**: Removes duplicate reads to reduce bias from PCR over-amplification or optical duplicates.

By examining the relevant FastQC modules—**Per Base Sequence Quality**, **Sequence Duplication Levels**, and **Sequence Length Distribution**—you can decide which tools to use and how to set their parameters. Always validate your changes by running FastQC again to ensure your data quality has improved and you haven’t lost valuable information.
## Step 3: Trimming Single-End Reads

For single-end data (one FASTQ file per sample), here’s a typical **Trimmomatic** command:

```bash
trimmomatic SE -threads 4 \
  input_reads.fastq.gz \
  trimmed_reads.fastq.gz \
  ILLUMINACLIP:adapters.fa:2:30:10 MINLEN:36
```

- **SE**: Single-end mode.
- **-threads 4**: Use 4 threads (adjust as needed).
- **ILLUMINACLIP:adapters.fa:2:30:10**: Reference your `adapters.fa`, and set clipping parameters. The numbers (e.g., `2:30:10`) represent mismatches, palindrome clip threshold, and simple clip threshold, respectively.
- **MINLEN:36**: Discards reads that fall below 36 bases after trimming.

If you want to process **all** single-end FASTQ files in the directory, you can use a loop:

```bash
for file in *.fastq.gz
do
  echo "Trimming $file ..."
  trimmomatic SE -threads 4 "$file" "trimmed_$file" \
    ILLUMINACLIP:adapters.fa:2:30:10 MINLEN:36
done
```

---

## Step 4: Trimming Paired-End Reads

Paired-end data usually comes in files named something like `sample_R1.fastq.gz` and `sample_R2.fastq.gz`. The command is slightly different:

```bash
trimmomatic PE -threads 4 \
  sample_R1.fastq.gz sample_R2.fastq.gz \
  sample_R1_paired.fastq.gz sample_R1_unpaired.fastq.gz \
  sample_R2_paired.fastq.gz sample_R2_unpaired.fastq.gz \
  ILLUMINACLIP:adapters.fa:2:30:10 MINLEN:36
```

- **PE**: Paired-end mode.
- **sample_R1.fastq.gz sample_R2.fastq.gz**: Input files.
- **sample_R1_paired.fastq.gz sample_R1_unpaired.fastq.gz**: Output for R1 (paired and unpaired reads).
- **sample_R2_paired.fastq.gz sample_R2_unpaired.fastq.gz**: Output for R2 (paired and unpaired reads).
- **ILLUMINACLIP:adapters.fa:2:30:10**: Use the adapter sequences in `adapters.fa`.
- **MINLEN:36**: Discard reads that end up shorter than 36 bases.

To process **all** paired-end files in your directory:

```bash
for file in *_R1.fastq.gz
do
  base=$(basename "$file" _R1.fastq.gz)
  echo "Trimming $base ..."
  trimmomatic PE -threads 4 \
    "${base}_R1.fastq.gz" "${base}_R2.fastq.gz" \
    "${base}_R1_paired.fastq.gz" "${base}_R1_unpaired.fastq.gz" \
    "${base}_R2_paired.fastq.gz" "${base}_R2_unpaired.fastq.gz" \
    ILLUMINACLIP:adapters.fa:2:30:10 MINLEN:36
done
```

---

## Troubleshooting

1. **Adapter Content Remains High**  
   - Ensure you’re using the correct adapter file and the right parameters.
   - Check if your reads need more aggressive trimming (e.g., increasing mismatch thresholds).
2. **Reads Are Very Short**  
   - Check if you’re discarding too many bases or if your reads were already short.
   - Adjust `MINLEN` if needed.
3. **Quality Scores Still Low**  
   - Look into whether the data has inherent quality issues not fixable by trimming.
   - Consider re-checking the new trimmed FASTQ with FastQC to ensure improvement.

---

## Further Reading

- **[Trimmomatic Documentation](http://www.usadellab.org/cms/?page=trimmomatic)**
- **[Cutadapt Documentation](https://cutadapt.readthedocs.io/en/stable/)**
- **[FastQC Documentation](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/)**

---

### Final Notes

1. **Check Output Quality**  
   After trimming, run FastQC again to confirm adapter removal and improved base quality.
2. **Adapt Parameters**  
   Trimming parameters depend on your specific data (read length, quality distribution, adapter type).
3. **Consistent Folder Organization**  
   Store your trimmed files in a separate folder (e.g., `trimmed_data/`) or clearly label them (`trimmed_` prefix) to avoid confusion with raw reads.

By following these simple steps, you can ensure your data is free from adapters and low-quality regions, setting a solid foundation for reliable downstream analyses.
```

**Instructions:**
1. **Create a folder named `Trimming/`** in your repository.  
2. **Copy the above content** into a file called `README.md` inside that folder.  
3. **Adjust commands or file names** (e.g., references to `adapters.fa`, read length thresholds) according to your specific needs.  

You now have a **step-by-step, generalized guide** explaining how to trim FASTQ files (single-end or paired-end) based on FastQC results and adapter sequences.
