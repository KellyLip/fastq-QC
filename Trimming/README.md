# Trimming FASTQ Files

This README file explains how to trim your FASTQ files based on:
1. **Quality Control Results** (from FastQC or another QC tool).
2. **Known Adapter Sequences** listed in `adapters.fa`.

By trimming reads, you remove low-quality regions and adapter contamination, improving the accuracy of downstream analyses such as alignment or assembly.

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
- **MINLEN** (discards excessively short reads)

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
## 4. MINLEN

### What Is It?

`MINLEN` is a Trimmomatic parameter that specifies the **minimum read length** required after all other trimming steps (e.g., removing adapters or low-quality bases). Any reads that end up shorter than this threshold are discarded entirely. This helps ensure that extremely short reads—which usually contain minimal usable information—do not pass into downstream analysis.

### Typical Syntax

When you run Trimmomatic, you might see a command like:
```bash
trimmomatic SE -threads 4 \
  input.fastq.gz \
  output_trimmed.fastq.gz \
  ILLUMINACLIP:adapters.fa:2:30:10 SLIDINGWINDOW:4:20 MINLEN:50
```

- **MINLEN:50** means any read that is fewer than 50 bases long after the trimming process is discarded.

### How to Choose the MINLEN Value

1. **Consider Your Original Read Length**  
   - If your reads are 150 bp (bases) each, dropping down to 50 bp still retains ~33% of the read length.  
   - If your reads are only 75 bp to start with, you might choose a lower threshold (e.g., MINLEN:36) to retain enough reads.

2. **Review “Sequence Length Distribution” in FastQC**  
   - If most reads remain fairly long even after trimming, you can afford a higher `MINLEN` (e.g., 50 or 75).  
   - If you see that many reads would drop below a certain length (e.g., 30–40 bp) after trimming poor-quality tails, you might opt for a slightly lower `MINLEN` to retain enough data, while still removing uninformative short reads.

3. **Downstream Application Requirements**  
   - **Alignment or Assembly**: Some aligners work better with reads above a certain length (e.g., 50 bp).  
   - **Coverage Considerations**: If discarding too many reads drastically reduces your coverage, you might want to choose a more lenient threshold.  
   - **Amplicon or Targeted Sequencing**: If you know your amplicons are short, you may need a lower `MINLEN`.

4. **Experiment and Validate**  
   - Run Trimmomatic with a chosen `MINLEN`, then re-run FastQC on the trimmed reads.  
   - Check if you still meet your desired coverage or read quality metrics.

### Example Command with MINLEN

**Single-End Example:**
```bash
trimmomatic SE -threads 4 \
  input.fastq.gz \
  output_trimmed.fastq.gz \
  ILLUMINACLIP:adapters.fa:2:30:10 SLIDINGWINDOW:4:20 MINLEN:50
```

**Paired-End Example:**
```bash
trimmomatic PE -threads 4 \
  sample_R1.fastq.gz sample_R2.fastq.gz \
  sample_R1_paired.fastq.gz sample_R1_unpaired.fastq.gz \
  sample_R2_paired.fastq.gz sample_R2_unpaired.fastq.gz \
  ILLUMINACLIP:adapters.fa:2:30:10 SLIDINGWINDOW:4:20 MINLEN:50
```

- Here, `MINLEN:50` discards any read (or read pair) that ends up below 50 bp after all other trimming operations.

---

## Summary

- **SLIDINGWINDOW**: A Trimmomatic parameter that trims low-quality regions in a step-wise manner.
- **FLASH**: Merges paired-end reads that overlap, commonly used for short-insert or amplicon libraries.
- **fastuniq**: Removes duplicate reads to reduce bias from PCR over-amplification or optical duplicates.
- **MINLEN** discards excessively short reads that remain after adapter/quality trimming.

---
## Workflow ##

Below is a comprehensive example workflow showing how to trim your FASTQ files using **Trimmomatic** with multiple parameters—**ILLUMINACLIP**, **SLIDINGWINDOW**, **MINLEN**—for both single-end and paired-end data. It also includes how to run these commands on **all FASTQ files** in a directory.

---

## Overview

1. **Single-End Workflow**  
   - Trims all single-end FASTQ files in the directory.  
2. **Paired-End Workflow**  
   - Trims all paired-end files (identified as `_R1` and `_R2`) in the directory.  
3. **Parameters Explained**  
   - **ILLUMINACLIP**: Removes adapter sequences using an adapter FASTA file (e.g., `adapters.fa`).  
   - **SLIDINGWINDOW**: Scans the reads in windows of a specified size and trims if the average quality falls below a threshold.  
   - **MINLEN**: Discards reads (or read pairs) that fall below a specified length after trimming.  

---

## 1. Single-End Trimming

If your files are **single-end** reads, you typically have one FASTQ file per sample (e.g., `sample.fastq.gz`):

**Shell Script / Terminal Commands**:

```bash
# Loop through all single-end FASTQ files ending in .fastq.gz
for file in *.fastq.gz; do
  echo "Processing file: $file"
  
  trimmomatic SE -threads 4 \
    "$file" \
    "trimmed_$file" \
    ILLUMINACLIP:adapters.fa:2:30:10 SLIDINGWINDOW:4:20 MINLEN:50
  
done
```

### Explanation

- **trimmomatic SE**: Runs Trimmomatic in single-end mode.
- **-threads 4**: Uses 4 CPU threads (adjust according to your system).
- **"$file"**: The input FASTQ file (loop variable).
- **"trimmed_$file"**: The output FASTQ file name is prefixed with `trimmed_`.
- **ILLUMINACLIP:adapters.fa:2:30:10**:  
  - Uses `adapters.fa` to remove adapter sequences.  
  - Parameters `2:30:10` can be adjusted based on how strictly you want to match/truncate adapters.
- **SLIDINGWINDOW:4:20**:  
  - A sliding window of size 4 bases.  
  - Trims when the average quality in that window falls below Q20.
- **MINLEN:50**:  
  - Discards reads that end up shorter than 50 bases after trimming.

---

## 2. Paired-End Trimming

If your files are **paired-end**, each sample typically has two FASTQ files, named something like `sample_R1.fastq.gz` and `sample_R2.fastq.gz`.

**Shell Script / Terminal Commands**:

```bash
# Loop through all R1 FASTQ files ending in _R1.fastq.gz
for file in *_R1.fastq.gz; do
  # Extract the base name (everything before _R1.fastq.gz)
  base=$(basename "$file" _R1.fastq.gz)
  
  echo "Processing sample: $base"
  
  trimmomatic PE -threads 4 \
    "${base}_R1.fastq.gz" "${base}_R2.fastq.gz" \
    "${base}_R1_paired.fastq.gz" "${base}_R1_unpaired.fastq.gz" \
    "${base}_R2_paired.fastq.gz" "${base}_R2_unpaired.fastq.gz" \
    ILLUMINACLIP:adapters.fa:2:30:10 SLIDINGWINDOW:4:20 MINLEN:50

done
```

### Explanation

- **trimmomatic PE**: Runs Trimmomatic in paired-end mode.
- **"${base}_R1.fastq.gz" "${base}_R2.fastq.gz"**: Input R1 and R2 files for the same sample.
- **"${base}_R1_paired.fastq.gz" "${base}_R1_unpaired.fastq.gz"**: Outputs for R1 (paired reads that remain after trimming and unpaired if one mate is discarded).
- **"${base}_R2_paired.fastq.gz" "${base}_R2_unpaired.fastq.gz"**: The corresponding outputs for R2.
- **ILLUMINACLIP:adapters.fa:2:30:10 SLIDINGWINDOW:4:20 MINLEN:50**: Same usage as single-end, just applied to both R1 and R2.

---

## 3. Parameters in Detail

1. **ILLUMINACLIP:adapters.fa:2:30:10**  
   - **adapters.fa**: A FASTA file containing adapter sequences you want to remove. This could be Nextera, TruSeq, or any custom adapter file.  
   - The numbers (`2:30:10`) represent:
     - **2**: Maximum mismatch count allowed in the adapter sequence.
     - **30**: Palindrome clip threshold.
     - **10**: Simple clip threshold.

2. **SLIDINGWINDOW:4:20**  
   - **Window size of 4 bases**.  
   - **Average quality threshold of 20** (Q20 ≈ 99% base call accuracy).

3. **MINLEN:50**  
   - Any read that ends up shorter than 50 bases after the above trimming steps is discarded.  
   - Adjust based on your desired minimum read length.

---

## 4. Checking Your Results

After running these scripts, you will have new FASTQ files labeled either `trimmed_` (single-end) or `_paired`/`_unpaired` (paired-end). To confirm improvements:

1. **Re-run FastQC** on the trimmed FASTQ files.
2. Check if:
   - **Adapter contamination** is gone or significantly reduced.  
   - **Per Base Sequence Quality** is higher and more uniform.  
   - **Sequence Length Distribution** still meets your downstream needs.

---

## Optional Steps

- **Merging Overlapping Reads (FLASH)**: If you have short inserts that overlap significantly, you can use [FLASH](https://ccb.jhu.edu/software/FLASH/) on the paired reads after trimming:
  ```bash
  flash trimmed_sample_R1_paired.fastq.gz trimmed_sample_R2_paired.fastq.gz -o sample_merged -M 250
  ```
  - **-M 250** sets the maximum overlap to 250 bases (adjust as needed).

- **Removing Duplicate Reads (fastuniq)**: If FastQC shows high duplication, you might use [fastuniq](https://sourceforge.net/projects/fastuniq/) to remove exact duplicates in paired-end data.

---

## Summary

1. **Single-End**: Loop over files matching `*.fastq.gz` and run `trimmomatic SE`.  
2. **Paired-End**: Loop over files matching `*_R1.fastq.gz` and run `trimmomatic PE` on each pair.  
3. **Parameters**:  
   - **ILLUMINACLIP** to remove adapter sequences.  
   - **SLIDINGWINDOW** to trim low-quality regions dynamically.  
   - **MINLEN** to discard reads below a certain length.  

With these examples, you can efficiently trim all FASTQ files in your directory. Always review your FastQC reports to guide parameter decisions and confirm that trimming improves your data quality.
