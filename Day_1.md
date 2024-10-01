# Setup
bashrc stuff


To use [SNPArcher](https://snparcher.readthedocs.io/en/latest/) we'll need to setup a designated conda environment. While one *could* use their standard conda environment to install SNPArcher, I personally like to have specific environments for specific workflows to avoid compatbility issues and clashing between different projects.

Prior to the workshop, you should have conda setup on your cluster. After logging on, create a new environment and install the `snparcher` package:
```
conda create -c conda-forge -c bioconda -n snparcher "snakemake>=8" "python==3.11.4"
conda activate snparcher
```

