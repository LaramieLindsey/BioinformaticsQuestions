# Final Report for the Bioinformatics Questions
## Completed by: Laramie Lindsey
## Date: 10 May 2023

* NOTE: all data files are located in this repository.

### Question 1
1) Can you use GATK haplotypecaller to call variants from the 3 given bam files? (focus on regions of BRCA1 and DMD exons only)
* I used GATK Best Practices as a guide.
* Prep the reference genome (download, index, and create sequence dictionary).

```
wget https://ftp.ensembl.org/pub/grch37/current/fasta/homo_sapiens/dna/Homo_sapiens.GRCh37.dna.primary_assembly.fa.gz
```

```
samtools faidx ref.fasta
```

```
/home/laramie/gatk-4.4.0.0/gatk CreateSequenceDictionary -R ref.fasta
```

* Create intervals file to only include exon coordinates for BRCA1 and DMD, respectively.

* Run the following command ```HaplotypeCaller``` on each sample bam file using the GRCh37 human genome as reference to generate GCVF files for each sample. 

```
/home/laramie/gatk-4.4.0.0/gatk HaplotypeCaller \
-R ../ref.fasta \
-I ../SampleA.processed.bam \
-O /home/laramie/gatk/gatkProject/ProjectQs/Question1/SampleA.X.vcf \
-ERC GVCF -L DMD.intervals
```

* Next, Combine GCVFs using the ```GenomicsDBImport``` command.
```
/home/laramie/gatk-4.4.0.0/gatk GenomicsDBImport \
    -V SampleADMD.vcf \
    -V SampleBDMD.vcf \
    -V SampleCDMD.vcf \
    --genomicsdb-workspace-path mydb \
    --intervals DMD.intervals
```

* Genotype the GCVFs from the GenomicsDB created in the previous step.
```
/home/laramie/gatk-4.4.0.0/gatk GenotypeGCVFs \
   -R ref.fasta \
   -V gendb://mydb \
   -O DMDGenotyped.vcf
   -L DMD.intervals
```
### Question 2
2) Can you remove 1bp insertions from the VCF files?

* I checked the files for insertions. The BRCA1 VCF file did not contain any insertions, but the DMD VCF file did contain a few 1bp insertions. 
* This may not be the best solution, but I used the SelectVariants to output a file with insertions removed, using the command below. 
```
/home/laramie/gatk-4.4.0.0/gatk SelectVariants \
     -R ref.fasta \
     -V DMDGenotyped.vcf \
     --select-type-to-include SNP \
     -L DMD.intervals \
     -O DMD_no1bpinsertions.vcf
```
* I would like to know if there is a better/more accurate way of removing the 1bp insertions. For the sake of time, I decided to use the SelectVariants command.

### Question 3
3) What is the genomic coverage of each DMA exon in these samples? Do you see any DMD deletion in any samples?
* I am assuming this is referring to DMD exons.
* Following best practices, I used DepthOfCoverage from the GATK toolkit to create files. See below code: 
```
/home/laramie/gatk-4.4.0.0/gatk \
   DepthOfCoverage \
   -R ref.fasta \
   -O DMDCoverageSampleA \
   -I SampleA.processed.bam \
   -L DMD.intervals
```
* I did my best to interpret the output files. 
