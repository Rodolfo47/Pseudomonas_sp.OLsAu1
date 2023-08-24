# *Pseudomonas* sp.

Rodolfo Ángeles y Diana Oaxaca

February-August 2023



## Aim and scope

Rosario Ramírez Mendoza and Jesús Pérez Moreno obtained the Mi-Seq sequencing of the OLsAu1 strain of *Pseudomonas* sp. previously approximated to *Pseudomonas azotoformans* (Pineda-Mendoza et al. 2019).

I'm going to generate the genome assembly and annotation of the strain. Later together we will decide the subsequent analyzes.

All the bioinformatics processes were carried out in the Kayab, CCG, UNAM cluster.

A *P. azotoformans* genome is reported (Fang et al. 2016).



## To do list

I will tick this list of activities and convert some ones into the sections of this binnacle

- [x] Create the working directories in Kayab and on my laptop
- [x] Upload sequencing data
- [x] Raw FastQC
- [x] Cleaning
- [x] Celan FastQC
- [x] Backup clean data
- [x] Assembling
- [x] QUAST
- [x] Assembly report
- [x] BUSCO
- [x] Annotation
- [ ] Genome descriptors table
- [ ] Annotation report
- [ ] Schedule second meeting



## Cleaning

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



## Assembling

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



## BUSCO



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

## Assembling results

Best assembly were obtained with default trimming parameters and default (autodetect k-mer size) assembling parameter.

Put the selected assembly in the res dir

```sh
cp ../out/spades/trim_def.spades_def/*.fasta ../res/assemby/
ln -s ../res/assemby/contigs.fasta ../res/assemby/Pseudomonas_sp_OLsAu1.fna
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

Terminar la tabla de reporte y compartir los resultados para agendar cita 



- Fang, Y., Wu, L., Chen, G., & Feng, G. (2016). Complete genome sequence of Pseudomonas azotoformans S4, a potential biocontrol bacterium. Journal of biotechnology, 227, 25-26.
- Pineda-Mendoza, D. Y., Almaraz, J. J., Lara-Hernandez, M. E., Arteaga-Garibay, R., & Silva-Rojas, H. V. (2019). Cepas de bacterias aisladas de esporomas de hongos ectomicorrízicos promueven el crecimiento vegetal. ITEA-Información Técnica Económica Agraria, 115(1), 4-17.
