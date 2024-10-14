# Intro
When you have low coverage data (<5X), working with genotype likelihoods instead of variant calls (VCFs) may be preferred. At the least, you can verify some of your analyses on SNPs with the analogous likelihood-based approach. Fair warning, **there will be differences** between results from the two approaches - while this is expected to some extent, it has led to headaches for many bioinformaticians. At some point, you will have to use your own discretion and sanity checks to determine what differences are meaningful and which ones are trivial.

Now, with that warning aside, the main way we'll be working with genotype likelihoods is via [ANGSD](https://www.popgen.dk/angsd/index.php/ANGSD). ANGSD takes many types of input, but for the most part we'll be providing it `.bam` files. 

# PCA
With ANGSD/PCAngsd, we can generate a PCA based directly on the bam files. To do this, we need to:
1. Convert bam files to beagle format
2. Use beagle format for PCA calculation

```
# STEP 1. Setup directory for angsd input/output.

mkdir -p angsd
ls snpArcher/results/cracherodii/bams/*final.bam > angsd/bamlist.txt

# The command to generate beagle output (-doGlf 2). There are MANY flags here for ANGSD, and a full explanation is beyond the scope of this workshop.
# This is probably going to take a minute to complete, so we'll test the command first, then cancel and restart as a SLURM job when it seems like things are running ok. 

CMD="angsd -C 50 -minMapQ 30 -minQ 20 -GL 2 -doGlf 2 -doMajorMinor 1 -doMaf 1 -SNP_pval 1e-6 -nThreads 10 -ref snpArcher/refs/JAJLRC010000027.1.fa -bam angsd/bamlist.txt -out angsd/abalone"
$CMD #Cancel once this is running fine
sbatch --mem=10000 --time=01:00:00 -p long -c 10 -J beagle -e logs/beagle.e -o logs/beagle.o --wrap="$CMD"

```
**Exercise**: How could we incorporate our `genmap` mappability bedgraph into the first angsd command? 

Now that Step 1 is done, on to Step 2. But it turns out that PCAngsd is a separate program that needs to be downloaded! Let's take care of this quickly. 

```
git clone https://github.com/Rosemeis/pcangsd.git
cd pcangsd/
pip3 install .

# Now we can run it! This is relatively fast with our toy dataset
pcangsd -b angsd/abalone.beagle.gz -o angsd/PCA -t 8 --iter 1000

```


If your run didn't converge, let me know. Otherwise, we can move on to plotting the results in R:

```
sample_order = read.table('~/Downloads/transfer/bamlist.txt') %>% mutate(sample = basename(V1) %>% gsub("_final.bam","",.))
C = as.matrix(read.table('~/Downloads/transfer/PCA.cov'))
e = eigen(C)
pca_results = cbind(e$vectors,sample_order)
ggplot(pca_results, aes(x = `1`,y = `2`)) +
    geom_label(aes(label = sample)) +
    geom_point(size = 4, alpha = 0.5) + 
    theme_classic()
```
![image](https://github.com/user-attachments/assets/9df6fde7-3869-4f22-b760-23d37512ee0a)



# Diversity and selection stats
