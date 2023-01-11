## Organize sample fastq files for next step

- Because this project include samples from both fecal samples and cecal digesta samples, and I have different metadata files for them, I will put the fastq files of them in seperate folders for easy analysis for future steps
- I use cyberduck to move files, this can also be done using termial.
- Inside `dada/raw-seq` folder, create folder `Fecal-raw-seq` and `Cecal-raw-seq`, and move the fecal samples into `Fecal-raw-seq` folder, and cecal samples into `Cecal-raw-seq` folder. Note here I did not move the fecal w1_f8 sample to the fecal folder because there was no data of that animal in other weeks, and I'm not going to use this data for other analysis
- Also, create a result folder for cecal project `dir.create("results/dada2/cecal_result")`. Do the same for the fecal project `dir.create("results/dada2/cecal_result")`. This can also be done on cyberduck

## Log onto Biocluster2 and open R session

- Log on the the biocluster, and then log on to a new interactive node with the following command: `srun --pty /bin/bash`, then move to the working directory `cd Saro2022/`
- Load the R module: `module load R/4.1.2-IGB-gcc-8.2.0`
- Start running R: `R`
- When finish coding, quit R: `q()`
- For this project, the primer is `V4_515F_806R_New`, with forward primer length of 19, reverse primer length of 20. We will use these values later in the trimming part
- Dada2 suggests hard trimming when average quality scores are below 30. After looking at the MultiQC report, I choose to trim: forward read to length of 250, reverse reads to length of 200.

- These codes are for a 16S paired-end data set sequenced on an Illumina MiSeq platform. Reads are 300nt, and from the V4 region

## Load packages

- Install package, this only need to be done once

```
BiocManager::install('dada2')
install.packages('ggplot2')
```

- Load packages

```
library(dada2)
library(ggplot2)
```

## Sample trimming

### Import files into dada2 format

- Confirm that the FASTQ files are in the correct location

```
path <- "data/raw-seq/Cecal-raw-seq/"
list.files(path)
```

### Sort paired-end reads

- Remember, forward and reverse fastq filenames have format SAMPLENAME_R1.fastq and SAMPLENAME_R2.fastq

```
  fnF <- sort(list.files(path, pattern="\_R1.fastq", full.names = TRUE))
  fnR <- sort(list.files(path, pattern="\_R2.fastq", full.names = TRUE))
```

- Make sure the lengths are the same

```
length(fnF) == length(fnR)
```

### Extract sample names

- Assuming filenames have format: SAMPLENAME_R1.fastq, eg."V4_515F_New_V4_806R_New-C_L5_CTACATACTA_R2.fastq"
- Sub means substitute, replace the `"\_R1.fastq"` with `""`

`sample.file.names <- sub("\_R1.fastq", "", basename(fnF))`

- The file name contains barcode, will edit next
- Get sample names

  ```
  sample.names <- sapply(strsplit(basename(fnF), "-"), `[`, 2)
  sample.trt <- sapply(strsplit(basename(sample.names), "_"), `[`,1)
  sample.num <- sapply(strsplit(basename(sample.names), "_"), `[`,2)
  sample.names <- paste(sample.trt,sample.num,sep ="\_")
  ```

### Quality Control

- Inspect read quality profiles using dada2.

Note: We already did this using FASTQC and MultiQC, but let's see what the dada2 version looks like

- We can use ggsave() to save it to a specific place and filename

```
plotQualityProfile(fnF, aggregate = TRUE)
ggsave("results/dada2/cecal_result/forward-quality-profile.pdf")
plotQualityProfile(fnR, aggregate = TRUE)
ggsave("results/dada2/cecal_result/reverse-quality-profile.pdf")
```

### Filtering the sequences

- Create filtered filenames and paths (in filtered/ subdirectory) for our output files

  `dir.create("results/dada2/cecal_result/filtered")`

```
filtF <- file.path("results/dada2/cecal_result/filtered", paste0(sample.names, "_F_filt.fastq.gz"))
filtR <- file.path("results/dada2/cecal_result/filtered", paste0(sample.names, "_R_filt.fastq.gz"))
```

- Assign names to these file paths

```
  names(filtF) <- sample.names
  names(filtR) <- sample.names
```

- Perform filtering

```
out <- filterAndTrim(fnF, # paths to the input forward reads
filtF, # paths to the output filtered forward reads
fnR, # paths to the input reverse reads
filtR, # paths to the output filtered reverse reads
trimLeft=c(19,20), # 19 reads of the forward primer, 20 reads for reverse reads
truncLen=c(250,200), # forward trim 250 and reverse trim 200
maxN=0, # not alloweing any N in the read, this is the default setting
maxEE=c(2,2), # more than 2 errors in the forward read, and reverse reads, espectively is removed, this is recommended by dada2
rm.phix=TRUE, # remove phix reads
multithread=10) # the number of CPUs requested, default is FALSE, if use TRUE, will use all the CPUs, don't use TRUE on biocluster

head(out)

```

### Learn Error Rates

- This step learns the error rates of your sequences for more accurate sample inference in the next step.
- This step is the second most computationally intensive. This may not run on your personal computer.
- NOTE: On Windows set multithread=FALSE.

````errF <- learnErrors(filtF, multithread=10)
errR <- learnErrors(filtR, multithread=10) # For multithread=10, takes 15min to run 39 samples```
````

- Plot errors to make sure everything looks as expected.

If you used the NovaSeq platform, and the error distribution has large dips in comparison to the theoretical red line, then check the recommendations for this here: https://github.com/benjjneb/dada2/issues/791#issuecomment-617891477

```
plotErrors(errF, nominalQ=TRUE)
ggsave("results/dada2/cecal_result/forward-error-distribution.pdf")
plotErrors(errR, nominalQ=TRUE)
ggsave("results/dada2/cecal_result/reverse-error-distribution.pdf")
```

### Dereplicate samples

- Dereplication combines all identical reads into into “unique sequences” with a corresponding “abundance”: the number of reads with that unique sequence. This reduces computation time by eliminating redundant comparisons. This part could require a lot of RAM.

```
derepF <- derepFastq(filtF, verbose = TRUE)
derepR <- derepFastq(filtR, verbose = TRUE)
```

### Sample Inference

- This is the core algorithm of dada2 that will infer real sequence variants from our unique sequences using the error models we generated to guide its decisions. See the dada2 publication for more details: https://doi.org/10.1038/nmeth.3869

This step takes about 10-15min for 39 samples

````dadaF <- dada(derepF, err=errF, multithread=10)
dadaR <- dada(derepR, err=errR, multithread=10)```
````

### Merge paired reads

- This is where we'll finally merge the paired-end reads together into one read, which will further improve accuracy. Note that any unmerged reads will be lost. It is possible to rescue these unmerged reads (see the returnRejects option), but proceed with caution. Also note that it is possible to artificially merge reads that don't overlap using the justConcatenate option.

```
mergers <- mergePairs(
  dadaF, filtF,
  dadaR, filtR,
  verbose=TRUE
  # minOverlap = 200 # optional, set the minimum overlap of our PE reads to 200
)

# Inspect the merger data.frame from the first sample
head(mergers[[1]])
```

- Make a sequence table

  ```
  seqtab <- makeSequenceTable(mergers)
  dim(seqtab)
  # Inspect distribution of sequence lengths (should be around size of V4 region)
  table(nchar(getSequences(seqtab)))

  ```

### Remove chimeras

- Chimeras are a mistake that happens during PCR where to different sequences merge together to form a hybrid sequence. This are easier to identify after all filtering has happened.
- This is not unusual for chimeras number to be large (eb. ~20%), and sometimes they might make up a majority. The number of non-chimeras greater than 40% is generallly fine

```
seqtab.nochim <- removeBimeraDenovo(seqtab, method="consensus", multithread=10, verbose=TRUE)
dim(seqtab.nochim)
sum(seqtab.nochim)/sum(seqtab) # 86%, this is good

# Now we need to export these data so we can access them later.
write.table(seqtab.nochim,
            file = "results/dada2/cecal_result/seqtab_nochim.txt",
            quote = FALSE,
            sep = "\t",
)
```

### Summarize results

- Create a summary table of sequences lost at each filtering step. Confirm that there was a gradual loss of sequence reads at each step. If there was a drastic loss, consider adjusting parameters at that step.

```
getN <- function(x) sum(getUniques(x))
track <- cbind(out, sapply(dadaF, getN), sapply(dadaR, getN), sapply(mergers, getN), rowSums(seqtab.nochim))
# If processing a single sample, remove the sapply calls: e.g. replace sapply(dadaFs, getN) with getN(dadaFs)
colnames(track) <- c("input", "filtered", "denoisedF", "denoisedR", "merged", "nonchim")
rownames(track) <- sample.names
head(track)
```

## Assign taxonomy

- We assign taxonomy in two steps using the Silva 138.1 database. This is the most computationally intensive step, and requires a decent amount of memory/RAM. There's good chance this won't run on a laptop for a full data set.

### Step 1: Assign taxonomy through genus

```
taxa <- assignTaxonomy(seqtab.nochim,
"data/reference/silva_nr99_v138.1_train_set.fa.gz",
multithread=10
)
```

### Step 2: Assign species if 100% match

```
taxa <- addSpecies(taxa,
"data/reference/silva_species_assignment_v138.1.fa.gz"
)
```

### Preview of results

```
taxa.print <- taxa # Removing sequence row names for display only
rownames(taxa.print) <- NULL
head(taxa.print)
```

### Save final data

```
# write to file
write.table(taxa,
file = "results/dada2/cecal_result/taxa.txt",
quote = FALSE,
sep = "\t",
)

# save as RDS file
saveRDS(track, file = "results/dada2/cecal_result/summary-of-dada2.RDS")
saveRDS(seqtab.nochim, file = "results/dada2/cecal_result/seqtab_final.RDS")
saveRDS(taxa, file = "results/dada2/cecal_result/taxonomy_final.RDS")

```
