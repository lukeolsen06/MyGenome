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
### Step size of 10 
```sbatch velvetoptimiser_noclean.sh UFVPY232 61 131 10```

### Step size of 2
```sbatch velvetoptimiser_noclean.sh UFVPY232 93 109 2```

## 6. Blast Search
### Ran a blastn search using the sequence in MoRepeats.fasta as the query and my genome as the database (subject)
```blastn -subject UFVPY232.fasta -query MoRepeats.fasta -out MoRepeats.UFVPY232.BLASTn0 -evalue 1e-20 -outfmt 0```

### Export a list of contigs that mostly comprise mitochondrial sequences:
```awk '$4/$3 > 0.9 {print $2 ",mitochondrion"}' MoMitochondrion.UFVPY232.BLAST > UFVPY232_mitochondrion.csv```
#### Results: [UFVPY232_mitochondrion.csv](/Results/UFVPY232_mitochondrion.csv)

### Use the script to find the ID and length of the longest contig in your assembly:
```perl SequenceLengths.pl UFVPY232_nh.fasta | sort -k2n```

## 6. Variant Calling
###  Identifying Genetic Variants between B71v2sh genome and my genome assembly
```sbatch CallVariants.sh ./UFVPY232_blast/```

## 7. Gene Prediction with Hidden Markov Models
### Running SNAP
```snap-hmm Moryzae.hmm UFVPY232_final.fasta > UFVP232-snap.zff```
#### Results: [UFVPY232-snap.zff](/Results/UFVPY232-snap.zff)

### Running Augustus
```augustus --species=magnaporthe_grisea --gff3=on --singlestrand=true --progress=true ../snap/UFVPY232_final.fasta > UFVPY232-augustus.gff3```
#### Results: [UFVPY232-augustus.gff3.zip](/Results/UFVPY232-augustus.gff3.zip)

### Running MAKER
```maker 2>&1 | tee maker.log```
#### Results: [UFVPY232-annotations.zip](/Results/UFVPY232-annotations.zip)


