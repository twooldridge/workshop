# Setup

For this workshop we'll need to setup a designated conda environment. While one *could* use their standard conda environment for everything, I personally like to have specific environments for specific workflows to avoid compatbility issues and clashing between different projects.

So, first things first, let's install the python package `mamba` in our base environment. After we do this, we can use it to make all the conda installs MUCH faster

```
conda install -c conda-forge mamba
```

Then, we create an environment for this workshop where we'll setup all the software we need, including [SNPArcher](https://snparcher.readthedocs.io/en/latest/) and it's requirements
```
mamba create -c conda-forge -c bioconda -n snparcher "snakemake>=8" "python==3.11.4"
mamba activate snparcher
# Now we install some more handy tools
mamba install bedtools samtools ncbi-datasets-cli
```

Now, create a project directory and download snpArcher into your project directory:
```
mkdir -p workshop
cd workshop
git clone https://github.com/harvardinformatics/snpArcher.git
```

Quick test
```
cd snpArcher
snakemake -d .test/ci/ --cores 1 --use-conda
```


Let's do some other organizational things in the project folder before we start. Your philosophy on organization may differ, but here's what I do:
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

** Discussion session **








