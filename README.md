# *Pseudomonas* sp. OLsAu1

This is the bioinformatics repository that contains the code we use to assemble, annotate, and assign a species to the OLsAu1 strain.

This genome is available from NCBI (JAVGXC000000000) and corresponding to the article:

**Whole-genome sequence of *Pseudomonas yamanorum* OLsAu1 isolated from the edible wild ectomycorrhizal mushroom *Lactarius* sp. section Deliciosi**



The computational procedures were carried out by Rodolfo Ángeles and Diana Oaxaca at Kayab, CCG, UNAM cluster.

Genome sequencing was performed by Rosario Ramírez Mendoza.

The isolation of the strain was carried out by Juan José Almaraz Suárez.

The conceptualization of the project, the coordination of the work team and the review of the manuscript were carried out by Jesús Pérez Moreno.

February-August 2023



## Work flow



- [x] Assembly
  - [x] Cleaning
  - [x] Assembling
  - [x] Selecction and related

- [x] Annotation
  - [x] Prokka
  - [x] EggNog

- [x] Taxonomic placement
  - [x] pyani


## Assembly

### Cleaning

Ver la calidad de los datos crudos

```sh
mv data/raw
fastqc *.fastq 
```

| Filename                          | Rosario_S4_L001_R1_001.fastq |
| --------------------------------- | ---------------------------- |
| File type                         | Conventional base calls      |
| Encoding                          | Sanger / Illumina 1.9        |
| Total Sequences                   | 2,141,717                    |
| Sequences flagged as poor quality | 0                            |
| Sequence length                   | 35-151                       |
| %GC                               | 60                           |

| Filename                          | Rosario_S4_L001_R2_001.fastq |
| --------------------------------- | ---------------------------- |
| File type                         | Conventional base calls      |
| Encoding                          | Sanger / Illumina 1.9        |
| Total Sequences                   | 2,141,717                    |
| Sequences flagged as poor quality | 0                            |
| Sequence length                   | 35-151                       |
| %GC                               | 60                           |

Clean the raw reads with TrimGalore (v0.6.7)

As the sequencing libraries are small (2 million reads). I will clean in 5 ways

```sh
qsub -q Big.q@compute-1-11.local trim.jdl
```

`trim.jdl` content:

```sh
#$ -S /bin/bash
#$ -N trim
#$ -cwd
#$ -o trim.out
#$ -e trim.err
source /etc/bashrc
export LC_ALL=en_US.UTF-8

##RodolfoAngeles 19/02/23
cd ../data/raw

trim_galore --paired --illumina --fastqc -o ../clean/def *.fastq

trim_galore --paired --illumina --fastqc --length 75 -o ../clean/75 *.fastq

trim_galore --paired --illumina --fastqc --length 130 -o ../clean/130 *.fastq

trim_galore --paired --illumina --fastqc --length 75 --clip_R1 15 --clip_R2 15 --three_prime_clip_R1 2 --three_prime_clip_R2 2 -o ../clean/75clip *.fastq

trim_galore --paired --illumina --fastqc --length 130 --clip_R1 15 --clip_R2 15 --three_prime_clip_R1 2 --three_prime_clip_R2 2 -o ../clean/130clip *.fastq
```



### Assembling

Do the assembly whit SPAdes (v3.13.1)

```sh
qsub -q Big.q@compute-1-11.local assem.jdl
```

`assem.jdl` content:

```sh
#$ -S /bin/bash
#$ -N assem
#$ -cwd
#$ -o assem.out
#$ -e assem.err
source /etc/bashrc
export LC_ALL=en_US.UTF-8

#Trim_def
spades.py -1 ../data/clean/def/*val_1.fq -2 ../data/clean/def/*val_2.fq --careful -t 40 -o ../out/spades/trim_def.spades_def

spades.py -1 ../data/clean/def/*val_1.fq -2 ../data/clean/def/*val_2.fq --careful -k 127,117,107,97 -t 40 -o ../out/spades/trim_def.spades_kmer

#Trim_130
spades.py -1 ../data/clean/130/*val_1.fq -2 ../data/clean/130/*val_2.fq --careful -t 40 -o ../out/spades/trim_130.spades_def

spades.py -1 ../data/clean/130/*val_1.fq -2 ../data/clean/130/*val_2.fq --careful -k 127,117,107,97 -t 40 -o ../out/spades/trim_130.spades_kmer

#Trim_130clip
spades.py -1 ../data/clean/130clip/*val_1.fq -2 ../data/clean/130clip/*val_2.fq --careful -t 40 -o ../out/spades/trim_130clip.spades_def

spades.py -1 ../data/clean/130clip/*val_1.fq -2 ../data/clean/130clip/*val_2.fq --careful -k 127,117,107,97 -t 40 -o ../out/spades/trim_130clip.spades_kmer
#Trim_75
spades.py -1 ../data/clean/75/*val_1.fq -2 ../data/clean/75/*val_2.fq --careful -t 40 -o ../out/spades/trim_75.spades_def

spades.py -1 ../data/clean/75/*val_1.fq -2 ../data/clean/75/*val_2.fq --careful -k 75,65,55,45 -t 40 -o ../out/spades/trim_75.spades_kmer

#Trim_75clip
spades.py -1 ../data/clean/75clip/*val_1.fq -2 ../data/clean/75clip/*val_2.fq --careful -t 40 -o ../out/spades/trim_75clip.spades_def

spades.py -1 ../data/clean/75clip/*val_1.fq -2 ../data/clean/75clip/*val_2.fq --careful -k 75,65,55,45 -t 40 -o ../out/spades/trim_75clip.spades_kmer
```



### BUSCO

```sh
qsub -q Big.q@compute-1-11.local 03_busco.jdl
```

```sh
#$ -S /bin/bash
#$ -N busco
#$ -cwd
#$ -o busco.out
#$ -e busco.err
source /etc/bashrc
export LC_ALL=en_US.UTF-8
conda activate busco

for assem in $(ls ../out/spades/); do
	busco -i ../out/spades/$assem/contigs.fasta -l pseudomonadales_odb10 -o ../out/busco/$assem -m genome -f ;
done
```

### Assembling results

Best assembly were obtained with default trimming parameters and default (autodetect k-mer size) assembling parameter.

Put the selected assembly in the res dir

```sh
cp ../out/spades/trim_def.spades_def/*.fasta ../res/assemby/
ln -s ../res/assemby/contigs.fasta ../res/assemby/Pseudomonas_sp_OLsAu1.fna
```

## Sequencing coverage

Mapping clean reads using bowtie (version 2.3.5.1)

```sh
#Coverage estimation with bowtie2
##Build index
bowtie2-build ../../data/Pseudomonas_sp_OLsAu1.fna OLsAu1.fna
## Mapping
bowtie2 -p 22 -x OLsAu1.fna -1 ../../data/clean/def/Rosario_S4_L001_R1_001_val_1.fq -2 ../../data/clean/def/Rosario_S4_L001_R2_001_val_2.fq -S OLsAu1.sam

#2137703 reads; of these:
#  2137703 (100.00%) were paired; of these:
#    93723 (4.38%) aligned concordantly 0 times
#    2027987 (94.87%) aligned concordantly exactly 1 time
#    15993 (0.75%) aligned concordantly >1 times
#    ----
#    93723 pairs aligned concordantly 0 times; of these:
#      64845 (69.19%) aligned discordantly 1 time
#    ----
#    28878 pairs aligned 0 times concordantly or discordantly; of these:
#      57756 mates make up the pairs; of these:
#        28533 (49.40%) aligned 0 times
#        24284 (42.05%) aligned exactly 1 time
#        4939 (8.55%) aligned >1 times
#99.33% overall alignment rate

##Calculation
	
	#2137703 (100.00%) were paired;
	#99.33% overall alignment rate
	# So:
	# paired seq * alignment rate
echo "2137703 * 0.9933" | bc -l 
	# Result =  2123380.3899 aligned paired reads
	# So:
	# (aligned paired reads * read length * pair) / genome size
echo "(2123380.3899 * 151 * 2) / 6884230" | bc -l 
	# Result = 93.14925238549554561657 genome seq coverage

```



## Get some values for the table 1



```sh
#Features (Prokka): 6254
grep -c ">" Pseudomonas_sp_OLsAu1.ffn
#CDS (Prokka): 6188
awk -F '\t' '$2 == "CDS" { print $2 }' Pseudomonas_sp_OLsAu1.tsv | wc -l
#Proteins (Prokka): 6188
grep -c ">" Pseudomonas_sp_OLsAu1.faa
#Hypothetical proteins (Prokka): 2353
grep -c "hypothetical protein" Pseudomonas_sp_OLsAu1.faa
#COGs (Prokka):2555
grep -v "locus_tag" Pseudomonas_sp_OLsAu1.tsv | awk -F '\t' '{ print $6 }' | grep -vc " ^$ "
#EC (Prokka):1962
grep -v "locus_tag" Pseudomonas_sp_OLsAu1.tsv | awk -F '\t' '{ print $5 }' | grep -vc " ^$ "

#Proteins (EggNog):6407
grep -c ">" MM_d_a4j77c.emapper.genepred.fasta
#Hypothetical proteins (EggNog): 145 + 373 = 518
grep -v "#" MM_d_a4j77c.emapper.annotations.tsv | awk -F '\t' ' {print $8}' | grep -c "Protein of unknown function"
grep -v "#" MM_d_a4j77c.emapper.annotations.tsv | awk -F '\t' ' {print $8}' | grep -c "-"
#Annotated COGs (EggNog):5724
grep -v "#" MM_d_a4j77c.emapper.annotations.tsv | awk -F '\t' '$7 != "-" {print $7}' | wc -l
#Annotated KOs (EggNog):3697
grep -v "#" MM_d_a4j77c.emapper.annotations.tsv | awk -F '\t' '$12 != "-" { print $12 }' | wc -l
#CAZy (EggNog):50
grep -v "#" MM_d_a4j77c.emapper.annotations.tsv | awk -F '\t' '$19 != "-" { print $19 }' | wc -l
#PFAM (EggNog):5628
grep -v "#" MM_d_a4j77c.emapper.annotations.tsv | awk -F '\t' '$21 != "-" { print $21 }' | wc -l
#genome density: 6407 *1000000 / 6884230 = 930.67 
```



```
grep -v "#" MM_d_a4j77c.emapper.annotations.tsv | awk -F '\t' '{ print $1, $12, $13, $14, $15, $16, $18 }' | grep "ko:" | wc -l
```

## Annotation

Annotating the genome with Prokka (v1.14.6)

```sh
qsub -q Big.q@compute-1-11.local 04_prokka.jdl
```

```sh
#$ -S /bin/bash
#$ -N prokka
#$ -cwd
#$ -o prokka.out
#$ -e prokka.err
source /etc/bashrc
export LC_ALL=en_US.UTF-8
conda activate prokka

prokka --prefix Pseudomonas_sp_OLsAu1 --kingdom Bacteria --genus Pseudomonas --strain OLsAu1 --usegenus --cpus 40 --outdir ../out/prokka/Pseudomonas_sp_OLsAu1 ../res/assemby/Pseudomonas_sp_OLsAu1.fna
```

### EggNog

New annotations with EggNog (emapper-2.1.9: online server)

| Job name    | User                 | Date created | File                      | Sequences (type)      | Total length | Min. hit e-value | Min. hit bit-score | Min. % of identity | Min. % of query cov. | Min. % of subject cov. | Tax. scope | Orth. restrictions | GO evidence    |      |
| ----------- | -------------------- | ------------ | ------------------------- | --------------------- | ------------ | ---------------- | ------------------ | ------------------ | -------------------- | ---------------------- | ---------- | ------------------ | -------------- | ---- |
| MM_d_a4j77c | rodolfo.angeles. ... | 03/19/23     | Pseudomonas_sp_OLsAu1.fna | 624 (contigs_genomic) | 6884269      | 0.001            | 60                 | 40                 | 20                   | 20                     | auto       | all                | non-electronic |      |



Reference:

**eggNOG-mapper v2: functional annotation, orthology assignments, and domain prediction at the metagenomic scale.**
*Carlos P. Cantalapiedra, Ana Hernandez-Plaza, Ivica Letunic, Peer Bork, and Jaime Huerta-Cepas.* 2021
[Molecular Biology and Evolution, msab293, https://doi.org/10.1093/molbev/msab293](https://doi.org/10.1093/molbev/msab293)



## Taxonomic placement

### ANI

pyani (0.2.11)

```sh
qsub -q Big.q@compute-1-13.local 05_pyani.jdl
```



```sh
#$ -S /bin/bash
#$ -N ani
#$ -cwd
#$ -o ani.out
#$ -e ani.err
source /etc/bashrc
export LC_ALL=en_US.UTF-8

##RodolfoAngeles 2/03/23

#average_nucleotide_identity.py -i ../data -m ANIm -g --gmethod seaborn --labels labels.tab -o ../out/pyani/pyani.v1

#average_nucleotide_identity.py -i ../data -m ANIm -g --labels labels.tab -o ../out/pyani/pyani.v2

#average_nucleotide_identity.py -i ../data -m ANIm -g --gmethod seaborn --labels labels.tab -o ../out/pyani/pyani.v3

average_nucleotide_identity.py -i ../data -m ANIm -g --labels labels.tab -o ../out/pyani/pyani.v4
```
