### Basename sample file

Inside src/ folder, store fastqc.sh file and sample name file "basenames-295-full.txt", note here 295 is the total number of samples in the file

- Notice that the name in the basenames files has to be unique if using wild-card such as \* or ?.
  For instance, if you have animal ID 1-1 and you use 1-1\*, it will get 1-1, and 1-10, 1-11, etc

- Get basenames

  cd to raw-seq folder, then run `ls *R1.fastq > ../../src/basenames-196.txt`. This will input all the file names in raw-seq folder into the basenames-196.txt, but now all the name ends with "R1.fastq", which will be remove by the following command

  ```
  cd ../../src
  sed -i 's/R1.fastq//' basenames-196.txt
  ```

  The resulting basenames file will have sample ID + bar code information. You should run the next step inside `src/' folder.

  In terminal created nano file by typing `nano`, then copy the following file and edit information for your samples as needed.

  Submit job by running `sbatch fastqc.sh`

### Fastqc

fastqc.sh

- For Saro project:

```

#!/bin/bash
#SBATCH -N 1
#SBATCH -n 12
#SBATCH --mail-user=yifeik3@illinois.edu
#SBATCH --mail-type=END,FAIL
#SBATCH -J raw-fastqc
#SBATCH --array=1-196%5
#SBATCH -D /home/n-z/yifeik3/Saro2022/src/slurm-out/
#SBATCH -p normal

## MAKE SURE TO REPLACE THE 'X's ABOVE WITH YOUR INFORMATION (there are two places to do this)

cd ..

#Load the program needed; name may change in future so check each time with 'module avail FastQC'
# on command line
module load FastQC/0.11.8-Java-1.8.0_152

#Set up variable to pull base file names and make output directory
line=$(sed -n -e "$SLURM_ARRAY_TASK_ID p" basenames-196.txt)

#Can use echo statements to put into .out files to help check on progress and diagnose
# locations of problems
echo "starting fastqc"
fastqc -o ../results/fastqc/ ../data/raw-seq/${line}R1.fastq ../data/raw-seq/${line}R2.fastq
echo "fastqc finished"

```

- For Ynsect project:

```
#!/bin/bash
#SBATCH -N 1
#SBATCH -n 12
#SBATCH --mail-user=yifeik3@illinois.edu
#SBATCH --mail-type=END,FAIL
#SBATCH -J raw-fastqc
#SBATCH --array=1-295%5
#SBATCH -D /home/n-z/yifeik3/Ynsect2021_R/Ynsect-Fecal-microbial-2021/src/slurm-out
#SBATCH -p normal

## MAKE SURE TO REPLACE THE 'X's ABOVE WITH YOUR INFORMATION (there are two places to do this)

cd ..

#Load the program needed; name may change in future so check each time with 'module avail FastQC'
# on command line
module load FastQC/0.11.8-Java-1.8.0_152

#Set up variable to pull base file names and make output directory
line=$(sed -n -e "$SLURM_ARRAY_TASK_ID p" basenames-295-full.txt)

#Can use echo statements to put into .out files to help check on progress and diagnose
# locations of problems
echo "starting fastqc"
#fastqc -o ../results/fastqc/ ../data/raw-seq/fecal-dataset/${line}R?.fastq
fastqc -o ../results/fastqc/ ../data/raw-seq/fecal-dataset/${line}R1.fastq ../data/raw-seq/fecal-dataset/${line}R2.fastq
echo "fastqc finished"

```

Some comments on the code above:

```
#SBATCH --array=1-295%5
```

Here %5 means run 5 job at a time from the 295 jobs

```
#SBATCH -D /home/n-z/yifeik3/Ynsect2021_R/Ynsect-Fecal-microbial-2021/src/slurm-out
```

- D to set the direcotry for the slurm out files. Notice there is a `cd ..` command to move to back to the src/ folder since the direcotry is set to be /slurm.out

- Fastqc itself can actually be given multiple fastq files and you could run it as a single job without using a job array (#SBATCH –array line in fastqc.sh) or setting up a line variable by doing:

`fastqc -o ../results/fastqc/ ../data/raw-seq/fecal-dataset/*.fastq`

However, this will run each of the 295 \* 2 = 590 fastq files sequentially instead of in parallel. You also will need to pull each pair of samples using the line variable in other scripts. So here is what I suggest doing: take the full name of just the R1.fastq files and put them into a basenames file, then remove the R1.fastq part. Codes for this are:

```
cd /home/n-z/yifeik3/Ynsect2021_R/Ynsect-Fecal-microbial-2021/data/raw-seq/fecal-dataset/
ls *R1.fastq > ../../../src/basenames-295-full.txt
cd ../../../src/
sed -i 's/R1.fastq//' basenames-295-full.txt
```

### MultiQC report

- Log on to an interactive node to do work
  `srun --pty /bin/bash`

Then your location will change from "yifeik3@biologin-1 src" to "yifeik3@compute-7-2 src"

- Remove all modules & then load the MultiQC module

To check the most up to date module, use `module avail multiqc`, The output with (D) is Default Module.

Here it is "MultiQC/1.11-IGB-gcc-8.2.0-Python-3.7.2"

```
module purge
module load MultiQC/1.11-IGB-gcc-8.2.0-Python-3.7.2
```

- Move to the correct folder and then run MultiQC on the folder containing the FASTQC files you just generated

```
cd ~/Ynsect2021_R/Ynsect-Fecal-microbial-2021/results/fastqc/
multiqc .
```

- For the Saro2022 project

```
cd ~/Saro2022/results/fastqc/
multiqc .
```

Then download the MultiQC html report from cyberduck.

Under "Sequence Quality Histograms", pick the position for score greater than 30 (score 30 means Base call accuracy 99.9%).

The green lines are forward reads, and red lines are reverse reads, typically forwards reads has higher quality than reverse reads.

Therefore, usually the cutoff for forward is 250, and reverse is 200. Will use these two numbers for downstream trimming.

- Moving to R code in the next step
