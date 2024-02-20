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
java -jar...
```

# 3. Count number to forward reads remaining
```bash
grep -c '@' UFVPY232_1_paired.fastq
```
