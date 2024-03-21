# MyGenome
Analyses for CS485G genome assembly

## 1. Analysis of sequence quality
The F1 and R1 sequence datasets were analyzed using FASTQC:
```bash
ssh -Y leol225@leol225.cs.uky.edu
cd MyGenome
fastqc &
```
Load F1 and R1 datasets into GUI interface.
Take screen shots of outuput files:
![F1Screenshot.png](/data/F1screenshot.png)
![R1Screenshot.png](/data/R1Screenshot.png)

## 2. Ran trimmomatic
```bash
java -jar trimmomatic-0.38.jar PE -threads 2 -phred33 -trimlog trimlog.txt UFVPY232_1.fq UFVPY232_2.fq UFVPY232_1_paired.fastq UFVPY232_1_unpaired.fastq UFVPY232_2_paired.fastq UFVPY232_2_unpaired.fastq CROP:280 SLIDINGWINDOW:20:20 MINLEN:120
```

## 3. Count number of forward reads remaining
```bash
grep -c '@' UFVPY232_1_paired.fastq
```
## 4. Count number of forward bases
```bash
awk 'NR%4==2 {total += length($0)} END {print total}' UFVPY232_1_paired.fastq
```

## 5. Assemble Genome
# Step size of 10 
sbatch velvetoptimiser_noclean.sh UFVPY232 61 131 10
sbatch velvetoptimiser_noclean.sh UFVPY232 93 109 2
