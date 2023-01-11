## Organize sample fastq files for next step

- Because this project include samples from both fecal samples and cecal digesta samples, and I have different metadata files for them, I will put the fastq files of them in seperate folders for easy analysis for future steps

- I use cyberduck to move files, this can also be done using termial.

- Inside `dada/raw-seq` folder, create folder `Fecal-raw-seq` and `Cecal-raw-seq`, and move the fecal samples into `Fecal-raw-seq` folder, and cecal samples into `Cecal-raw-seq` folder. Note here I did not move the fecal w1_f8 sample to the fecal folder because there was no data of that animal in other weeks, and I'm not going to use this data for other analysis

## Log onto Biocluster2 and open R session

- Log on the the biocluster, and then log on to a new interactive node with the following command: `srun --pty /bin/bash`, then move to the working directory `cd Saro2022/`

- Load the R module: `module load R/4.1.2-IGB-gcc-8.2.0`

- Start running R: `R`

- For this project, the primer is `V4_515F_806R_New`, with forward primer length of 19bp, reverse primer length of 20bp. We will use these values later in the trimming part

- Dada2 suggests hard trimming when average quality scores are below 30. After looking at the MultiQC report, I choose to trim: forward read to length of 250, reverse reads to length of 200.

- R code in the trimming part

```
out <- filterAndTrim(fnF, # paths to the input forward reads
                     filtF, # paths to the output filtered forward reads
                     fnR, # paths to the input reverse reads
                     filtR, # paths to the output filtered reverse reads
                     trimLeft=c(19,20), # 19 reads of the forward primer, 20 reads for reverse reads
                     truncLen=c(250,200), # forward trim 250 and reverse trim 200
                     maxN=0, # not alloweing any N in the read, this is the default setting
                     maxEE=c(2,2),  # more than 2 errors in the forward read, and reverse reads, espectively is removed, this is recommended by dada2
                     rm.phix=TRUE, # remove phix reads
                     multithread=6) # the number of CPUs requested, default is FALSE, if use TRUE, will use all the CPUs, don't use TRUE on biocluster
head(out)
```
