# Practical-Analysis-of-Nanopore-Sequencing-Data-Using-Third-Generation-Sequencing

## Introduction:
With the decreasing cost of third-generation sequencing (TGS), many researchers and enthusiasts are becoming interested in analyzing TGS data. This project aims to provide a comprehensive guide to TGS data analysis, allowing users to quickly get hands-on experience with the entire TGS workflow.


#!/bin/bash

# Step 1: Quality Control (Option 1: nanoQC)
mkdir nanoQC
nanoQC -o nanoQC -f input_data_path.fastq.gz
NanoStat --fastq input_data_path.sra.fastq.gz --outdir statreports

# Step 2: Filtering (Option 1: Nanofilt)
gunzip -c input_data_path.fastq.gz | NanoFilt -q 7 -l 1000 --headcrop 50 --tailcrop 50 | gzip > clean.NanoFilt.fastq.gz

# Step 3: Alignment (Option 1: minimap2)
minimap2 -d ref.mmi ref.fa
minimap2 -a ref.mmi reads.fq > alignment.sam

# Step 4: Sorting
samtools sort -@ 8 -o bam -o s0137.sorted.bam s1037.sam
samtools index s0137.sorted.bam
samtools faidx ref.fa

# Step 5: Assembly (Option 1: Canu)
canu -p output_prefix -d output_dir genomeSize=5g maxThreads=96 -nanopore-raw input_data_path > canu.log

# Step 6: Assembly Quality Assessment (Quast)
quast.py -r ref.fa canu.fa miniasm.fa wtdbg2.cns.fa smartdenovo.fa -o quast

# Step 7: Assembly Result Polishing (Option 1: Medaka)
medaka_consensus -i raw_reads.fastq.gz -d assembly.fasta -o medaka_result -m r941_min_high_g360 -v medaka.vcf -t 24 > medaka.log

# Step 8: Assembly Result Polishing (Option 2: Pilon)
bwa index pilon.fasta
bwa mem pilon.fasta input_data_path.fastq.gz | samtools view -Sb - > input_data_path.bam
samtools sort -o input_data_path.sorted.bam input_data_path.bam
samtools index input_data_path.sorted.bam
java -Xmx32G -jar pilon.jar --genome pilon.fasta --bam input_data_path.sorted.bam --output pilon_polished.fasta --threads 24 --fix all --verbose

``` 































