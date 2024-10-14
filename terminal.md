# Setup

## Text editor

## Command line

## Misc. software 

For some later analyses, we'll want to use PopLDDecay. Download the latest tarball from [here](https://github.com/BGI-shenzhen/PopLDdecay?tab=readme-ov-file) and transfer it to you Downloads or Software directory on the server. Then:

```
tar -zxvf  PopLDdecayXXX.tar.gz
cd PopLDdecayXXX;
cd src;
make ; make clean
cd ../;ln -s bin/PopLDDecay ~/bin/
```

```

```

## R
For all the R analyses here, we'll be working with relatively small pieces of data, and can therefore work locally as opposed to on the server. Depending on how we progress in this course, we may cover how to maintain R environments on the servers, and even how to use in conjunction with python.

But for now, just open up an R script in R studio, and make sure you can load the following libraries:

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

