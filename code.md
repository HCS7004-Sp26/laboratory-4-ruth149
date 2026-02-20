## Login in OSC, either by using SSH in your terminal, a command line instanc ein your web browser, or using VSCode from OSC
```shell
ssh ruthsarah@pitzer.osc.edu
Su6Er5Om6Ut4R%
```
## Check where you are and make sure you create your working directory in the right place
```shell
cd /fs/scratch/PAS3260/SarahR/
mkdir Lab4
```

## Copy files to your working directory, for example:
```shell
cp /fs/scratch/PAS3260/Jonathan/Lab_4/* Lab4
```

## Go to your working directory and execute:
```shell
cd Lab4
ls -l
ls -Llh
ls -lh
```
### What do you see?
    # A fasta file for the reference sequence
    # 1 fastq file, zipped and unzipped
    # 1 bam file and 1 sam file (probably the fastq file aligned to the reference)
    # 1 vcf file (probably variation btw reference seq and the fastq file)
    # 1 gff3 file (annotation file)
    # 1 md5 file (verify data integrity)
## There is a md5 file, what are md5 files for?     
    #The md5 file uses and algorithm to convert a file into a string of 32 characters. If the files become changed or corrupted, this will result in a different string. By computing the string for our current file and comparing it to the md5 file, we can check that they were copied correctly.
## Let's work with this file:
```shell
md5sum -c Lab5.md5 > Checking.txt 
cat Lab5.md5
cat Checking.txt
grep "FAIL" Checking.txt

## Changing file contents to experiment with md5 files
cp ref_Run1CB3334.fasta myDNA.fasta
nano myDNA.fasta #deleted first nucleotide
md5sum myDNA.fasta >> Lab5.md5 #add md5 sum from myDNA.fasta to the md5 file
md5sum -c Lab5.md5 > Checking_post.txt #recheck values
mv Rice2.gff3 Rice.gff3 #changing the file name prevents md5sum from working

#creating an md5 file (write all md5 sums and put it in a new file)
md5sum ref_Run1CB3334.fasta Rice.gff3 Run1CB3334.bam Run1CB3334.fastq Run1CB3334.fastq.gz Run1CB3334.sam Run1CB3334.vcf > new.md5
ls #check that file was created
cat new.md5 #check contents

# Inspect the files
```
## Now, let's explore the files a little:
```shell
head ref_Run1CB3334.fasta
head Rice.gff3
head Run1CB3334.fastq
head Run1CB3334.vcf
head Run1CB3334.fastq.gz
head Run1CB3334.bam
```
### Do you see something weird?
    #the zip file (.gz) and the .bam file are not human readable
## For `Run1CB3334.fastq.gz` try:
```shell
zcat Run1CB3334.fastq.gz | head
```

## Let's check the fastq file in a little more detail
```shell
head -100 Run1CB3334.fastq | less
```

### How would you count the number of reads in the fastq file?
    #count the number of lines as divide by 4
```shell
wc -l Run1CB3334.fastq #count the number of lines
```
# Is it? Try:
zcat Run1CB3334.fastq.gz | echo "$((`wc -l` / 4))" #unzip the file, count the number of lines, divide by four and print that to the screen
```
Why the difference?
### Let's download some data from NCBI
```
module spider sratoolkit
module spider sratoolkit/3.0.2
module load sratoolkit/3.0.2
# What is sratoolkit?
    #software for downloading and working with sequence data from the NCBI sequence read archive (SRA)
fastq-dump --gzip --split-files --readids --origfmt ERR3638927  #download zipped fastq file
```
(a faster option is fasterq-dump, how can you look for information about this command?)
fasterq-dump --help
```shell
zcat ERR3638927_1.fastq.gz | head
```
## Let's download a SAM file
```shell
sam-dump --output-file ERR3638927.sam ERR3638927
ls
```
To manipulate the SAM file we will need samtools:
```shell
module spider samtools
module spider samtools/1.21
module load samtools/1.21
```
Then, we can make a sorted BAM file:
```shell
samtools view -bS ERR3638927.sam | samtools sort -o ERR3638927_sorted.bam
```
Index it:
```shell
samtools index ERR3638927_sorted.bam
```
Make it a FASTA file:
```shell
samtools fasta ERR3638927_sorted.bam > ERR3638927.fasta
head ERR3638927.fasta
```
And index it:
```shell
samtools faidx ERR3638927.fasta
```
## Finally, let's check the VCF file
```shell
head -100 Run1CB3334.vcf | less
module load vcftools/0.1.16
vcftools --vcf Run1CB3334.vcf
vcftools --vcf Run1CB3334.vcf --maf 0.05
vcftools --vcf Run1CB3334.vcf --maf 0.05 --recode --out Run1CB3334_Filtered_maf05
vcftools --vcf Run1CB3334_Filtered_maf05.recode.vcf
```
## Can we check the VCF file using a container as in Lab 3?
```shell
module unload vcftools/0.1.16
vcftools --help
apptainer pull vcftools.sif docker://quay.io/biocontainers/vcftools:0.1.16--pl5321hdcf5f25_9
apptainer exec vcftools.sif vcftools --help
apptainer exec vcftools.sif vcftools --vcf Run1CB3334.vcf --maf 0.05 --recode --out Run1CB3334_Filtered_maf05_from_container
apptainer exec vcftools.sif vcftools --vcf Run1CB3334_Filtered_maf05_from_container.recode.vcf
```
## How could we use a container of samtools?
```shell
module unload samtools/1.10
samtools --help
apptainer pull samtools.sif docker://
apptainer exec samtools.sif samtools --help
```
Followign the same structure as VCFtools, define the commands to sort and index fasta and BAM files using the samtools container
