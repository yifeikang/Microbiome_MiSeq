## Root direcotry

For this projec is named as "Ynsect-Fecal-microbial-2021", "Saoro2022"

Inside the project file, there are 4 files: data, results, src, OriginalFiles

Optional:
To keep src/ folder from getting clogged up with slurm\*.out files (hundreds of them every time you run a job array!), first make a directory inside src called slurm-out

Inside data folder, there are 2 folders: reference, raw-seq

Inside results folder, there are 2 folders: results, fastqc

```
mkdir Ynsect-Fecal-microbial-2021
cd Ynsect-Fecal-microbial-2021
mkdir src data results OriginalFiles
mkdir src/slurm-out # optional
mkdir data/reference data/raw-seq
cd results
mkdir dada2 fastqc
```

From Ynsect-Fecal-microbial-2021/ or Saoro2022/ type`tree`to confirm the stucture of the directory

Store a copy the fastq file from MiSeq under data/raw-seq

Always keep copy of the oringal files!

## Unzip files

Inside OriginalFiles/ folder, follow instructions from sequencing center's email to download files.

Submit slurm job by nano file to unzip folder. DO NOT unzip the file on the login node of biocluster! Only unzip one file at a time, and rename the folder before unzip another

```
#!/bin/bash
# ----------------SLURM Parameters----------------
#SBATCH -p normal
#SBATCH -n 24
#SBATCH --mem=700g
#SBATCH -N 1
#SBATCH --mail-user yifeik3@illinois.edu
#SBATCH --mail-type END, FAIL
#SBATCH -J unzip
# ----------------Commands------------------------

tar -xjf Yifei_PrimerSortedDemultiplexed_2022122.bz2
```

Rename the unziped folder "PrimerSortedDemultiplexed", and make a copy of the file into data/raw-seq

`cp -avr ~/Saro2022/OriginalFiles/PrimerSortedDemultiplexed/* ~/Saro2022/data/raw-seq/`

## Download reference database

First move the reference folder `cd data/reference/`

- Download dada2 reference database under data/reference:
  https://benjjneb.github.io/dada2/training.html

Download databases:

- Silva version 138.1:
  https://zenodo.org/record/4587955#.Y3VEnuzMKAk

Right click on silva_nr99_v138.1_train_set.fa.gz and select "Copy Link".

In the Biocluster terminal type “wget “, then paste link, delete the “?download=1” from the end of the name, and then hit enter

```
wget https://zenodo.org/record/4587955/files/silva_nr99_v138.1_train_set.fa.gz
```

Make sure you are on a login node to get interenet connection!

Next, do the same steps for silva_species_assignment_v138.1.fa.gz file

```
wget https://zenodo.org/record/4587955/files/silva_species_assignment_v138.1.fa.gz
```

- Next, follow instructions in the Quality Check step
