# Setup



## Text editor

While you can make a lot of quick edits on the command line with `vim`/`nano` etc., it's nice to have an easy text editor on your local machine to play around with. If you're on Mac TextEdit should already be available, but I'm a big fan of [Sublime](https://www.sublimetext.com/),

## Command line

Let's make some little edits to `~/.bashrc`:
```

```

## snpArcher
For this workshop we'll need to setup a designated conda environment. While one *could* use their standard conda environment for everything, I personally like to have specific environments for specific workflows to avoid compatbility issues and clashing between different projects.


First, we create an environment for this workshop where we'll setup all the software we need, including [SNPArcher](https://snparcher.readthedocs.io/en/latest/) and its requirements
```
conda create -c conda-forge -c bioconda -n snparcher "snakemake>=8" "python==3.11.4"
conda activate snparcher
conda install -c conda-forge mamba<2.0.0 #Incompatibilities arise between snakemake and newer mamba
```

Now, create a project directory (if you haven't already) and download snpArcher into your project directory:
```
mkdir -p workshop
cd workshop
git clone https://github.com/harvardinformatics/snpArcher.git
```

This test is **absolutely** necessary to run to ensure that `snpArcher` is set up correctly. It will probably take around ~30 min just because of the conda setup involved, not because the data is large. Thankfully, this will already have been done prior to the workshop. 

```
cd snpArcher
snakemake -d .test/ci/ --cores 1 --use-conda
```


## Misc. software 

1) Where possible, let's use `conda` to install some other useful utilities:

```
mamba install bedtools samtools bcftools ncbi-datasets-cli
```

2) For some later analyses, we'll want to use PopLDDecay. Download the latest tarball from [here](https://github.com/BGI-shenzhen/PopLDdecay?tab=readme-ov-file) and transfer it to your Downloads or Software directory on the server. Then:

```
tar -zxvf  PopLDdecayXXX.tar.gz
cd PopLDdecayXXX;
cd src;
make ; make clean
cd ../;ln -s ${PWD}/bin/PopLDDecay ~/bin/
```

3) At times we'll want to use [PLINK](https://www.cog-genomics.org/plink/). Once you download the appropriate binary, just place it in your `~/bin/` folder and make sure it runs. No installation required!


4) We'll also be using [ANGSD](https://www.popgen.dk/angsd/index.php/Main_Page) for some analyses of genotype likelihoods. To install:
```
#download htslib
git clone --recurse-submodules https://github.com/samtools/htslib.git;
#download angsd
git clone https://github.com/angsd/angsd.git;

#install htslib
cd htslib
make

#install angsd
cd ../angsd
make HTSSRC=../htslib

# Now add to path
find . -type f -executable | xargs -I {} ln -s $PWD/{} ~/bin/
```

## R
For all the R analyses here, we'll be working with relatively small pieces of data, and can therefore work locally as opposed to on the server. Depending on how we progress in this course, we may cover how to maintain R environments on the servers, and even how to use in conjunction with python.

But for now, you just need RStudio installed on your desktop. Open it up, and make sure you can install the following libraries:

```
## Miscellaneous plotting and data handling packaes
install.packages('ggplot2')
install.packages('tidyverse')
install.packages('data.table')
install.packages('R.utils')
install.packages('magrittr')
install.packages("devtools")

## Lostruct for local PCA
devtools::install_github("petrelharp/local_pca/lostruct")
```

