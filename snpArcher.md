# Table of contents

1. [Setting up a `snpArcher` run](#setting-up-a-snparcher-run)
2. [`snpArcher` config file](#snparcher-config-file)
3. [Running `snpArcher`](#running-snparcher)
4. [Discussion](#discussion) 

# Setting up a `snpArcher` run

Ok, the following assumes that you have already set up a `snpArcher` Conda environment and have successfully executed the example datasets (see [setup](https://github.com/twooldridge/workshop/blob/main/terminal.md)). Let's do some other organizational things in the project folder before we start. Your philosophy on the organization of your working directories may differ, but there has to be one. The idea is that you can understand and localize your analyses easily. Here's an example of the way we will be organizing the data for the workshop:

```
mkdir -p data      # Where raw data goes
mkdir -p refs      # For reference assembly and associated files (e.g. blast database)
mkdir -p pops      # Metadata for samples and sequencing, mainly in the form of plain .txt files
mkdir -p cmds      # Where scripts will go
mkdir -p logs      # Where output files will go
mkdir -p downloads # Self-explanatory
```

We'll add more folders as we proceed with the analysis. Now, let's talk about the toy data I've prepared so we can run through analyses in real time. To do this, we're only including 20 individual black abalones of low-moderate coverage, and I've extracted the reads that map to one of the smaller scaffolds (`JAJLRC010000027.1`) to reduce file size and run time. 

> [!TIP]
> - Generally, when testing something new, **start small!**
> - I like to pick a favorite scaffold or region for a reference genome I work with before I do something on the whole genome. This will make troubleshooting easier and save you time before scaling up!

You should have already downloaded the `FASTQ` data we'll use. This isn't the 'rawest' data format, so the reads would be cleaner than expected. They are example reads I've subsampled from the data in the 2024 paper. Once you have them downloaded from Google Drive, move them to the `downloads` folder.

```
scp -r ~/Downloads/fastq.tar.gz ${USER}@${SERVER}:${WORKDIR}/downloads/
```

Download the black abalone reference genome directly from NCBI. This should take less than a minute.

```
datasets download genome accession GCA_022045235.1 \
  --include genome --filename downloads/cracherodii.zip
unzip -o downloads/cracherodii.zip
```


Some data organization: 


```
## First, the reference assembly
cd refs
cp ../ncbi_dataset/data/GCA_022045235.1/GCA_022045235.1_xgHalCrac1.p_genomic.fna cracherodii.fa
samtools faidx cracherodii.fa

## Now create a reference genome consisting of just the scaffold we'll be working with for this workshop.
samtools faidx cracherodii.fa JAJLRC010000027.1 > JAJLRC010000027.1.fa
samtools faidx JAJLRC010000027.1.fa
cd -
rm -rf ncbi_dataset README.md md5sum.txt

## Now, the fastq data. We'll try something a bit fancier to organize the read files
tar -xvf downloads/fastq.tar.gz
ls downloads/*fastq* | xargs -I {} cp downloads/fastq/{} data/

## Finally, the sample file:
cp downloads/samples.csv pops/
cp downloads/samples.csv snpArcher/config/samples.csv
```

There! Everything should be nice and organized now. 🎉


# `snpArcher` config file:

To actually run snpArcher, we not only need to setup the dependencies, but we need to download the snpArcher folder from github which contains all the scripts for running it. 
```
git clone https://github.com/harvardinformatics/snpArcher.git
cd snpArcher
head config/config.yaml
```

`snpArcher` needs a `samples.csv` file with all the relevant data (see below). We can create this in Excel and export it, or use the one I made for this class, which we just placed in `snpArcher/config`:


<img width="725" alt="image" src="https://github.com/user-attachments/assets/4f8faecb-269b-451f-91ca-498764683849">


Now, let's look more deeply at the parameters in the config file. You can quickly view the file first:


```
less -S config/config.yaml
```


<img width="1049" alt="image" src="https://github.com/user-attachments/assets/6b58680a-652b-48a6-9389-732f27ab46f2">


Let's walk through what we need to change in the config file. Under **`must change`**. We only need to modify:


```
final_prefix: "cracherodii"
generate_trackhub: False
```


For the **`might change`**, there are a few interesting parameters.

Let's discuss the filtering options:

- `maf: 0` -- SNPs with MAF below this value will be filtered from the final clean VCFs. Set to 0 to disable.
  - I like to set MAF to `0` and do all filtering downstream.
- `scaffolds_to_exclude: ""` -- comma separated, no spaces list of scaffolds to exclude from final clean VCFs. Set to blank to disable.
  - For example, in a selection scan, you might want to include even the lowest frequency
variants as long as they pass the other filtering criteria.
This doesn't apply to our toy dataset, but for whole genome data, you might want to exclude
the abalone mitogenome or perhaps other scaffolds with known problems.
We'll just set this to `""`, indicating that everything should be included.

- Filtering based on coverage: This can be considered a key feature of the pipeline. There are many ways to configure this, depending on your data type and how strict you
would like to be. I like to set my criteria based on standard deviation:


```
## If cov_filter is True, use these parameters to control how coverage filtering is done
## Three options are provided for coverage-based filtering. The default option is to just filter
## regions of the genome with mean coverage below a minimal threshold (default = 1), with a very large upper limit
## To use this option, set the variables below to the lower absolute mean coverage limit and upper absolute mean coverage limit, 
## and make sure all other coverage variables are empty

cov_threshold_lower: 
cov_threshold_upper: 

## Alternatively, filtering can be done based on standard deviations 
## (assumes a Poisson distribution, so stdev_cov equals the square root of the mean coverage),
## where regions of the genome with mean coverage < X standard deviations or > X standard deviations are removed.
## To use this option, set the variables below to the desired X
## and make sure all other coverage variables are empty

cov_threshold_stdev: 2

## Finally, filtering can be done based on the absolute scaling of the mean, 
## where regions of the genome with mean coverage < (global mean coverge / X) or > (global mean coverge * X) are removed.
## To use this option, set the variable below to the desired X
## and make sure all other coverage variables are empty

cov_threshold_rel:
```


# Running `snpArcher`


To execute on SLURM, we'll need to also make some changes in the SLURM config file in `profiles/slurm/config.yaml`. 


> [!CAUTION]
> Please consult your cluster admin if you have questions about tweaking these things. You will want to scale with consideration for other users and the size of the dataset.

![image](https://github.com/user-attachments/assets/5d4bd11d-6c9f-4e2f-87e1-7b1b1cdd3083)


First, we'll keep `jobs` at 100; since we only have 20 samples and one scaffold, there aren't any instances where having more than 20 jobs running in parallel will be helpful. When you start dealing with more samples and whole genome data, you can increase this parameter, but please keep in mind how many resources you're asking of the servers and what other people need to do.


We'll change `retries` to 1, `slurm_partition` to what your cluster admin would recommend, and let's just set a default `runtime` of 360 minutes (6 hours) for this test dataset. 


Now, let's go through all the options for the number of threads (CPUs) a process might use. I'll highlight the options where increasing the number of threads might make a difference in runtime:


```
genmap: 8
bwa_map: 8
dedup: 8
```


While some of the options could be increased from 1 to 2, for the vast majority of these, you won't see a significant increase in performance. Therefore, I set these at 1 so that the SLURM job requests fewer resources of the cluster and is less likely to stay `PENDING` for very long.


Below is a LONG list of configurable parameters for each pipeline step. We'll leave these unmodified for now, but this will be good to revisit if you see that some jobs need more resources. Anything specified here overrides the previous parameters at the top of the config file. For example, if `bam2gvcf` keeps timing out and needs more runtime to finish, go and uncomment that section and specify a new runtime variable:


<img width="747" alt="image" src="https://github.com/user-attachments/assets/e7b8ce93-29f9-4fa5-8e7b-6c70b2884ade">


Now, we're ready to run everything. The `snakemake` job that controls it all will run in a screen session you can check in on.


```
screen -S varcal
cd snpArcher
conda activate snparcher
snakemake -d ./ --workflow-profile profiles/slurm  --cores 1 --use-conda   # --cores 1 because we only need 1 core for the head job
```

Once we've ensured everything is set up and running, we can use the remaining time for questions and call it a day. The results will be ready by tomorrow!


# Discussion

Let's discuss the new abalone data you will analyze soon and consider whether there are `snpArcher` parameters we need to adjust for.

1) How many samples?
2) Breadth of coverage?
3) Size of reference genome? Scaffolds in reference genome? Quality?
4) Will we combine these data with another dataset (older, other population, etc.)? 








