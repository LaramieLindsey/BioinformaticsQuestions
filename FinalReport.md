# Final Report for the Bioinformatics Questions
## Completed by: Laramie Lindsey
## Date: 10 May 2023

* NOTE: all data files are located in this repository.

### Questions 1
1) Can you use GATK haplotypecaller to call variants from the 3 given bam files? (focus on regions of BRCA1 and DMD exons only)
* I used GATK Best Practices as a guide.
* Prep the reference genome (download, index, and create sequence dictionary).

```wget https://ftp.ensembl.org/pub/grch37/current/fasta/homo_sapiens/dna/Homo_sapiens.GRCh37.dna.primary_assembly.fa.gz```

```samtools faidx ref.fasta```

```/home/laramie/gatk-4.4.0.0/gatk CreateSequenceDictionary -R ref.fasta```

* Create intervals file to only include exon coordinates for BRCA1 and DMD, respectively.

* Run the following command ```HaplotypeCaller``` on each sample bam file using the GRCh37 human genome as reference to generate GCVF files for each sample. 

```/home/laramie/gatk-4.4.0.0/gatk HaplotypeCaller -R ../ref.fasta -I ../SampleA.processed.bam -O /home/laramie/gatk/gatkProject/ProjectQs/Question1/SampleA.X.vcf -ERC GVCF -L DMD.intervals```

* Next, Combine GCVFs using the GenomicsDB command.

```/home/laramie/gatk-4.4.0.0/gatk GenomicsDBImport \
    -V SampleADMD.vcf \
    -V SampleBDMD.vcf \
    -V SampleCDMD.vcf \
    --genomicsdb-workspace-path my_database \
    --intervals DMD.intervals```
