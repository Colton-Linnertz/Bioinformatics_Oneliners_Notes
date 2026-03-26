# Bioinformatics Oneliners & Notes

**[A collection of command-line examples to process and analyze genomics data]**

[......Refer to the Docs for full details on each tool..............][biostar](https://www.biostars.org/p/142545/)

## [BAM Processing]

### Align BAM:
##### # ideal for 30x Whole Genome Sequencing  
##### # use 'samblaster' to mark duplicates for downstream analysis  
##### # place sample name in BAM header to merge with later sequencing data from the same sample  
```bash
bwa mem -t 12 -M -R "@RG\tLB:"${sample}"\tID:"${fastq1}"\tSM:"${sample}"\tPL:Illumina" "${REFgenome}" "${fastq1}" "${fastq2}" | samblaster -M | samtools view -bhS - > "${sample}.bam"
```

### Sort BAM:
```bash
java -jar -Xmx48g -XX:ParallelGCThreads=12 /path/to/picard.jar SortSam INPUT="${sample}.bam" OUTPUT="${sample}.sorted.bam" SORT_ORDER=coordinate VALIDATION_STRINGENCY=LENIENT TMP_DIR=${TEMP}
```

### Index BAM:
```bash
java -jar -Xmx48g -XX:ParallelGCThreads=12 /path/to/picard.jar BuildBamIndex INPUT="${sample}.sorted.bam" OUTPUT="${sample}.sorted.bam.bai" VALIDATION_STRINGENCY=LENIENT TMP_DIR=${TEMP}
```

### BAM alignment stats:
##### # make directory: /stats/ to output all stats
```bash
samtools flagstat "${sample}.bam" > "./stats/${sample}.flagstats"
```

### bam2fasta:
##### # fasta header will be the QNAME (Query Name)
```bash
samtools view "${sample}.bam" | awk '{OFS="\t"; print ">"$1"\n"$10}' - > "${sample}.fa"
```

### bam2fastq:
```bash
samtools fastq "${sample}.bam" > "${sample}.fastq" && gzip "${sample}.fastq"
```

### Merge sorted BAMs from the same sample
##### # merge multiple lanes or different sequencing runs
##### # run this as a script instead of on the command-line
```bash
java -jar -Xmx24g -XX:ParallelGCThreads=12 /path/to/picard.jar MergeSamFiles \
	      I="${sample}_${lane1}.sorted.bam" \
	      I="${sample}_${lane2}.sorted.bam" \
	      I="${sample}_${lane3}.sorted.bam" \
	      I="${sample}_${lane4}.sorted.bam" \
	      O="${BAMs}/${sample}.bam" \
	      SORT_ORDER=coordinate \
	      VALIDATION_STRINGENCY=LENIENT \
	      TMP_DIR="${TEMP}" \
	      MAX_RECORDS_IN_RAM=2000000 \
	      CREATE_INDEX=true	
```
