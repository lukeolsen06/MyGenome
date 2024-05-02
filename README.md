# MyGenome (UFVPY232)
Analyses for CS485G genome assembly

## 1. Sequence quality assessment and trimming
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

### Ran trimmomatic
```bash
java -jar trimmomatic-0.38.jar PE -threads 2 -phred33 -trimlog trimlog.txt UFVPY232_1.fq UFVPY232_2.fq UFVPY232_1_paired.fastq UFVPY232_1_unpaired.fastq UFVPY232_2_paired.fastq UFVPY232_2_unpaired.fastq CROP:280 SLIDINGWINDOW:20:20 MINLEN:120
```

### Count of number of forward reads remaining
```bash
grep -c '@' UFVPY232_1_paired.fastq
```
Output: 8,813,356
### Count of the number of bases in cleaned reads (forward and reverse)
```bash
cat UFVPY232_1_paired.fastq UFVPY232_2_paired.fastq | awk 'NR%4==2 {total += length($0)} END {print total}'
```
Output: 2,640,919,623

## 2. Genome Assembly
### Submit assembly of step size 10 to SLURM
```sbatch velvetoptimiser_noclean.sh UFVPY232 61 131 10```

Results: [21-03-2024-15-42-45_Logfile.txt](/Results/21-03-2024-15-42-45_Logfile.txt)

### Submit assembly of step size 2 to SLURM
```sbatch velvetoptimiser_noclean.sh UFVPY232 93 109 2```

Results: [21-03-2024-16-03-49_Logfile.txt](/Results/21-03-2024-16-03-49_Logfile.txt)

## 3. Assessing genome completeness using BUSCO
```sbatch BuscoSingularity.sh UFVPY232/velvet_UFVPY232_93_109_2_noclean/UFVPY232_nh.fasta```

### Remove contigs less than 200 base pairs long
```perl CullShortContigs.pl UFVPY232_nh.fasta```

## 4. BLASTing genome against mitochondrial genome and the B71v2sh reference genome

### Mitochondrial genome
```blastn -query MoMitochondrion.fasta -subject UFVPY232_nh.fasta -evalue 1e-50 -max_target_seqs 20000 -outfmt '6 qseqid sseqid slen length qstart qend sstart send btop' -out MoMitochondrion.UFVPY232.BLAST```

[MoMitochondrion.UFVPY232.BLAST](/Results/MoMitochondrion.UFVPY232.BLAST)

### B71v2sh reference genome
```blastn -query B71v2sh_masked.fasta -subject UFVPY232_final.fasta -evalue 1e-50 -max_target_seqs 20000 -outfmt '6 qseqid sseqid qstart qend sstart send btop' -out B71v2sh.UFVPY232.BLAST```

[B71v2sh.UFVPY232.BLAST](/Results/B71v2sh.UFVPY232.BLAST)

### Export a list of contigs that mostly comprise mitochondrial sequences:
```awk '$4/$3 > 0.9 {print $2 ",mitochondrion"}' MoMitochondrion.UFVPY232.BLAST > UFVPY232_mitochondrion.csv```
#### Results: [UFVPY232_mitochondrion.csv](/Results/UFVPY232_mitochondrion.csv)

### Use the script to find the ID and length of the longest contig in your assembly:
```perl SequenceLengths.pl UFVPY232_nh.fasta | sort -k2n```

### Variant Calling
#### Identifying Genetic Variants between B71v2sh genome and my genome assembly
```sbatch CallVariants.sh ./UFVPY232_blast/```

## 5. Gene Prediction with Hidden Markov Models
### Running SNAP
```snap-hmm Moryzae.hmm UFVPY232_final.fasta > UFVP232-snap.zff```
#### Results: [UFVPY232-snap.zff](/Results/UFVPY232-snap.zff)

### Running Augustus
```augustus --species=magnaporthe_grisea --gff3=on --singlestrand=true --progress=true ../snap/UFVPY232_final.fasta > UFVPY232-augustus.gff3```
#### Results: [UFVPY232-augustus.gff3.zip](/Results/UFVPY232-augustus.gff3.zip)

### Running MAKER
```maker 2>&1 | tee maker.log```
```fasta_merge -d UFVPY232_final.maker.output/UFVPY232_final_master_datastore_index.log -o UFVPY232-genes.fasta```
#### Results: [UFVPY232-annotations.zip](/Results/UFVPY232-annotations.zip)

### Predicted Genes:
```grep -c '^>' UFVPY232-genes.fasta.all.maker.augustus.proteins.fasta```

Number of predicted proteins: 11,165


