**Random notes for things to add when relevant**
- use of screen
- xargs/parallel
- sed, awk, cut, tr
- vim/nano, less

# Setup

For this workshop we'll need to setup a designated conda environment. While one *could* use their standard conda environment for everything, I personally like to have specific environments for specific workflows to avoid compatbility issues and clashing between different projects.


First, we create an environment for this workshop where we'll setup all the software we need, including [SNPArcher](https://snparcher.readthedocs.io/en/latest/) and it's requirements
```
conda create -c conda-forge -c bioconda -n snparcher "snakemake>=8" "python==3.11.4"
conda activate snparcher
conda install -c conda-forge mamba<2.0.0 #Incompatibilities arise between snakemake and newer mamba
# Now we install some more handy tools
mamba install bedtools samtools ncbi-datasets-cli
```

Now, create a project directory and download snpArcher into your project directory:
```
mkdir -p workshop
cd workshop
git clone https://github.com/harvardinformatics/snpArcher.git
```

This test is **absolutely** necessary to run to ensure that snpArcher is set up correctly. It will probably take around ~30 min just because of the conda setup involved, not because the data is large.
```
cd snpArcher
snakemake -d .test/ci/ --cores 1 --use-conda
```


Now that *that's* done, let's do some other organizational things in the project folder before we start. Your philosophy on organization may differ, but here's what I do:
```
mkdir -p data    #Where raw data goes
mkdir -p refs    #For reference assembly and associated files (e.g. blast database)
mkdir -p pops    #Metadata for samples and sequencing, mainly in the form of plain .txt files
mkdir -p cmds    #Where scripts will go
mkdir -p logs    #Where output files will go
mkdir -p downloads # Self explanatory
```
We'll add more folders as we proceed with analysis. Now let's download the toy data we need for this course. I've prepared these in advance so that we can run through analyses in real time. To do this, we're only including 20 individual black abalone of low-moderate coverage, and I've extracted the reads that map to one of the smaller scaffolds (`JAJLRC010000058.1`) to reduce file size and run time. 

In general, whenever you're testing something new, start small! I like to pick a favorite scaffold or region for a reference genome I work with before I do something on the whole genome. This will make it easier to troubleshoot and save you time before scaling up!

```
# Download fastq data we'll be using. This isn't the 'rawest' data format
{from google drive}
# Download black abalone reference genome straight from NCBI. This should complete in less than a minute
datasets download genome accession GCA_022045235.1 --include genome --filename downloads/cracherodii.zip;unzip -o downloads/cracherodii.zip
```

Some organization:

```
# First, the reference assembly
cd refs
cp ../ncbi_dataset/data/GCA_022045235.1/GCA_022045235.1_xgHalCrac1.p_genomic.fna cracherodii.fa
samtools faidx cracherodii.fa
cd -
rm -rf ncbi_dataset README.md md5sum.txt

# Now, the fastq data. We'll try something a bit fancier
tar -xvf downloads/fastq.tar.gz
ls downloads/fastq | xargs -I {} cp downloads/fastq/{} data/ 
```

There! Everything should be nice and organized now.

```
cd snpArcher
head config/config.yaml
```

We see that snpArcher needs a 'samples.csv' file with all the relevant data. We can create this in excel and port it over, or use the one I made for this class

<img width="725" alt="image" src="https://github.com/user-attachments/assets/4f8faecb-269b-451f-91ca-498764683849">

So, from the `downloads` folder, copy the file `workshop_samples.csv` over to snpArcher/config/samples.csv.
Now, let's walk through some of the parameters in the config file. You can quickly view the file first:
```
less -S config/config.yaml
```

<img width="1049" alt="image" src="https://github.com/user-attachments/assets/6b58680a-652b-48a6-9389-732f27ab46f2">

Let's walk through what we need to change in the config file. Under 'must change'. we only need to modify:

```
final_prefix: "cracherodii"
generate_trackhub: False
```

For the 'might change', there are a bunch of potential things:
```

```

To execute on slurm, we'll need to also make some changes in the slurm config file in `profiles/slurm/config.yaml`. Please consult with your cluster admin if you have questions about tweaking these things. You will want to scale with consideration for other users and the size of the dataset.

![image](https://github.com/user-attachments/assets/5d4bd11d-6c9f-4e2f-87e1-7b1b1cdd3083)

First, we'll keep `jobs` at 100, since we only have 20 samples and one scaffold, there aren't any instances where it will be useful to have more than 20 jobs running at the same time. When you start to deal with more samples and whole genome data you can increase this parameter, but *please* keep in mind how many resources you're asking of the servers and what other people need to do.

We'll change `retries` to 1, `slurm_partition` to what your cluster admin would recommend, and let's just set a default `runtime` of 360 minutes (6 hours) for this test dataset. 

Now, let's go through all the options for the # of threads (cpus) a process might use. I'll highlight the optiuons where increasing the number of threads might actually make a difference in runtime
```
genmap: 8
bwa_map: 8
dedup: 8
```

While some of the options could be increased from 1 to 2, for the vast majority of these you won't really see a big increase in performance. Therefore, I set these at 1, so that the slurm job requests fewer resources of the cluster and is less likely to stay `PENDING` for very long

Now, below this is a LONG list of configurable parameters for each step of the pipeline. We'll leave these unmodified for now, but this will be good to revisit if you're seeing thtat some jobs need more slurm resources. Anything specified here overrides the previous parameters at the top of the config file. For example, if `bam2gvcf` keeps timing out and needs more runtime to finish, go and uncomment that section and specify a new runtime variable:

<img width="747" alt="image" src="https://github.com/user-attachments/assets/e7b8ce93-29f9-4fa5-8e7b-6c70b2884ade">

Now, we're ready to run everything. The snakemake job that controls it all will be run in a screen session that you can check in on.

```
screen -S varcal
cd snpArcher
conda activate snparcher
snakemake -d ./ --workflow-profile profiles/slurm  --cores 1 --use-conda   # --cores 1 because we only need 1 core for the head job
```

Once we've ensured that everything is up and running, we can use the remaining time for any questions and then call it a day. The results will be ready by tomorrow!

** Discussion session **








