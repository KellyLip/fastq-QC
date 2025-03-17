# FastQC Usage Guide

FastQC is an essential tool for performing quality control on raw sequencing data. This guide explains how to run FastQC, describes the modules in the FastQC report, and shows you how to process all FASTQ files in your directory with one command.

## Installation

You can install FastQC using conda or download it from the [FastQC website](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/).

To install via conda, run:

```bash
conda install fastqc
```

## Running FastQC on FASTQ Files

To run FastQC on one FASTQ file (either uncompressed or gzipped), simply use:

```bash
fastqc your_file.fastq
```

or, if the file is gzipped:

```bash
fastqc your_file.fastq.gz
```

To run FastQC on all gzipped FASTQ files in your current directory and send the output to a specific folder (here, `fastqc_results`), use the following command:

```bash
fastqc -o fastqc_results -t 2 *.fastq.gz
```

**Explanation:**

- `-o fastqc_results`: Directs FastQC to store its output in the folder named `fastqc_results`. If the folder does not exist, FastQC will create it.
- `-t 2`: Uses 2 threads to speed up processing.
- `*.fastq.gz`: Tells FastQC to run on all files ending with `.fastq.gz` in the current directory.

You can run this command in your terminal from the directory containing your FASTQ files.

## Understanding FastQC Report Modules

After FastQC finishes processing, it will generate an HTML report along with a zipped file containing detailed results. Here are the main modules you will see in the report:

1. **Basic Statistics:**  
   Provides an overview of the file, including the file name, total sequences, sequence length, and GC content.

2. **Per Base Sequence Quality:**  
   Displays quality scores for each base position across all reads. This module helps identify if there is a decline in quality toward the ends of reads.

3. **Per Sequence Quality Scores:**  
   Shows the distribution of overall quality scores across all sequences, allowing you to detect if a subset of reads has unusually low quality.

4. **Per Base Sequence Content:**  
   Illustrates the proportion of each nucleotide (A, T, G, C) at each base position. Uniform distribution is expected, though some variation may occur naturally.

5. **Per Sequence GC Content:**  
   Compares the observed GC content distribution with the expected (usually normal) distribution. Significant deviations may indicate contamination or bias.

6. **Per Base N Content:**  
   Plots the percentage of ambiguous ‘N’ bases at each position. High percentages can signal sequencing errors.

7. **Sequence Length Distribution:**  
   Shows the distribution of read lengths. This is particularly useful if your data comes from multiple protocols or if there are unexpected length variations.

8. **Sequence Duplication Levels:**  
   Indicates how often identical sequences appear in your data, which may be due to PCR duplicates or highly expressed regions.

9. **Overrepresented Sequences:**  
   Lists sequences that appear more frequently than expected, which might indicate contamination, adapter sequences, or PCR artifacts.

10. **Adapter Content:**  
    Identifies the presence of adapter sequences that may need to be trimmed before further analysis.

11. **K-mer Content:**  
    Analyzes the frequency of short nucleotide sequences (k-mers) to detect any unexpected patterns that might indicate biases or contamination.

## Troubleshooting

- **FastQC Command Not Found:**  
  Verify that FastQC is installed and that its installation path is included in your system’s PATH variable.

- **Output Folder Issues:**  
  Ensure that the output directory (`fastqc_results`) is writable or that you have permission to create it.

- **Processing All Files:**  
  Confirm that your FASTQ files are named with the `.fastq.gz` extension so that the wildcard (`*.fastq.gz`) matches them. If your files are uncompressed, change the command accordingly.
