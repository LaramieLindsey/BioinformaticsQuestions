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
* Following best practices, I used DepthOfCoverage from the GATK toolkit to create files. See below code: 
```
/home/laramie/gatk-4.4.0.0/gatk \
   DepthOfCoverage \
   -R ref.fasta \
   -O DMDCoverageSampleA \
   -I SampleA.processed.bam \
   -L DMD.intervals
```
* I did my best to interpret the output files. From my understanding the average depth of coverage of DMD exons for each sample is as follows:

| Sample ID | Mean Coverage | No coverage present? | # of areas with no coverage |
|-----------|---------------|----------------------|-----------------------------|
| SampleA   | 13.96         | Yes                  | 1                           |
| SampleB   | 8.50          | Yes                  | 6                           |
| SampleC   | 8.54          | Yes                  | 1                           |

Based on these results, I would conclude that SampleB most likely has a DMD exon deletion. SampleA and SampleC may have an exon deletion but I need to spend more time understaning how to interpret these results.

### Question 4
4) Check whether any of the 2 samples are from the same family. (Hint: mother and son)

* This question was a little harder for me. I used vcftools to create plink files. Then edited the .ped file to only include the 6 mandatory fields.
```
vcftools --vcf DMDGenotyped.vcf --plink --out plink
```
* I used CalculateGenotypePosteriors next using the command below: 
```
/home/laramie/gatk-4.4.0.0/gatk CalculateGenotypePosteriors -V DMDGenotyped.vcf -O familyFINAL.vcf -ped plinkDMD.ped
```
* However, I want to be honest and say that I am unsure how to interpret these results. I have no prior knowledge of the metadata for the samples. My guess would be that you can determine relatedness based on the number of shared variants. However, with a small dataset (exons only resulted in few variant sites) I am not sure I can confidently report relatedness. I would be more confident in interpreting the results by using the full vcf dataset, however, I do not have the compute resources to do so.

### Question 5
5) Are there any hard-clipping reads in the bam files? If yes, can you remove them?

* I believe there are hard clipped reads in the BAM file. I do not know the best way to remove them. One option I found was to use bamutils, but this option seems to remove all clipped reads (hard and soft). The other option I found was to use some awk code combined with samtools code. I tried this option but I am not 100% confident in the results. I also saw the ClipReads option on GATK but I don't fully understand all the options and if it can be used to remove reads based on hard clipping.

```
#extract name of clipped reads:
samtools view SampleA.processed.bam | awk '$6 ~ /H/{print $1}' | sort -k1,1 | uniq > names.txt

# sort original bam on names:
samtools view SampleA.processed.bam | sort -k1,1 > tmp.sam

# get header
samtools view -H SampleA.processed.bam > new.sam

# outer-join (I use ctrl+v+tab to explicitly get the tab character)
join -t '   ' -v 1 -1 1 -2 1 tmp.sam names.txt >> new.sam

# convert to bam
samtools view -bS new.sam > SampleA_noHCs.bam
```

### Summary
Overall, I recognize that I don't have a full grasp on variant calling. However, I hope I have shown that I am willing to learn and improve upon my skills. I plan to spend time learning the Best Practices for GATK and completing tutorials. 
