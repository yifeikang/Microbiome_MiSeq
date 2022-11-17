The root file for the project is named as "Ynsect-Fecal-microbial-2021"

Inside the project file, there are 3 files: data, results, src

Optional:
To keep src/ folder from getting clogged up with slurm\*.out files (295 of them every time you run a job array!), first make a directory inside src called slurm-out

Inside data folder, there are 2 folders: reference, raw-seq

Inside results folder, there are 2 folders: results, fastqc

```
mkdir Ynsect-Fecal-microbial-2021
cd Ynsect-Fecal-microbial-2021
mkdir src data results
# mkdir src/slurm-out
mkdir data/reference data/raw-seq
cd results
mkdir dada2 fastqc
```

From Ynsect-Fecal-microbial-2021/, type`tree`to confirm the stucture of the directory

Store a copy the fastq file from MiSeq under data/raw-seq

Always keep copy of the oringal files!

- Download dada2 reference database under data/reference
  https://benjjneb.github.io/dada2/training.html

Download databases:

Silva version 138.1

https://zenodo.org/record/4587955#.Y3VEnuzMKAk

Right click on silva_nr99_v138.1_train_set.fa.gz and select "Copy Link".

In the Biocluster terminal type “wget “, then paste link, delete the “?download=1” from the end of the name, and then hit enter

```
wget https://zenodo.org/record/4587955/files/silva_nr99_v138.1_train_set.fa.gz
```

Make sure you are on a login node!

Next, do the same steps for silva_species_assignment_v138.1.fa.gz file

```
wget https://zenodo.org/record/4587955/files/silva_species_assignment_v138.1.fa.gz
```

Follow instruction in the Quality Check file next
