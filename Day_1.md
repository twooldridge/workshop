# Setup
bashrc stuff


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




