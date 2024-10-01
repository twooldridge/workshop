# Setup

For this workshop we'll need to setup a designated conda environment. While one *could* use their standard conda environment for everything, I personally like to have specific environments for specific workflows to avoid compatbility issues and clashing between different projects.

So, first things first, let's install the python package `mamba` in our base environment. After we do this, we can use it to make all the conda installs MUCH faster

```
conda install -c conda-forge mamba
```

Then, we create an environment for this workship where we'll setup all the software we need, including [SNPArcher](https://snparcher.readthedocs.io/en/latest/) and it's requirements
```
mamba create -c conda-forge -c bioconda -n snparcher "snakemake>=8" "python==3.11.4"
mamba activate snparcher
mamba install bedtools 
```

Now, create a project directory and download snpArcher into your project directory:
```
mkdir -p haliotis
cd haliotis
git clone https://github.com/harvardinformatics/snpArcher.git
```

Let's do some other organizational things in the project folder before we start. Your philosophy on organization may differ, but always aim for more details and intuitive labels. Here's what I do:
```
cd haliotis
mkdir -p data    #Where raw data goes
mkdir -p cmds    #Where scripts will go
mkdir -p logs    #Where output files will go
```
We'll add more folders as we proceed with analysis.







