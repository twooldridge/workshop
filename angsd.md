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

We can also use ANGSD to generate estimates of population diversity, Fst, etc. Here's the rough order of operations:

1) Use `doSaf` to  create a 'site allele frequency likelihood file'
2) Use `realSFS` to generate an estimate of the site frequency spectrum
3) Use the outputs of 1) and 2) with `realSFS saf2theta` to calculate theta values
4) Use the output of 3) with `thetaStat print` or `thetaStat do_stat` to print theta values and selection stats, respectively.

> [!IMPORTANT]  
> ANGSD accepts many types of input for step 1 to work. In *theory*, we could use the same beagle file generated for the PCA exercise above. However, when not limited by time or resources, I like to start from the raw bam files again, as those contain much more information for ANGSD to generate its estimates from. The BEAGLE file is already a layer removed from the raw alignments, and so some info is necessarily lost.

Ok, with that note, here are the commands for steps 1-3, which will run fairly quickly for our toy dataset:
```
# Generate SAF. This runs in about 1-2 minutes with 10 cpus for our ~5MB scaffold
angsd -bam angsd/bamlist.txt -doSaf 1 -ref refs/JAJLRC010000027.1.fa -anc refs/JAJLRC010000027.1.fa -C 50 -minMapQ 30 -minQ 20 -GL 2 -P 10 -out angsd/abalone 

# Generate folded SFS - we can discuss folded v. unfolded at this point (or later). This runs in about 2-3 minutes with 10 cpus
realSFS angsd/abalone.saf.idx -P 10 -fold 1 > angsd/abalone.folded.sfs

# Generate thetas, runs in about a minute
realSFS saf2theta angsd/abalone.saf.idx -outname angsd/abalone -sfs angsd/abalone.folded.sfs -fold 1
```

Ok, assuming all of that ran smoothly, now we can move onto generate sum windowed-statistics to explore! Because we know *a priori* that abalone have a lot of sequence diversity, we can use some relatively small windows here. This will run almost instantly:

```
thetaStat do_stat angsd/abalone.thetas.idx -win 25000 -step 5000 -outnames angsd/abalone.25kb_by_5kb
```

Now we can visualize these results in R. Let's start with theta pi:

```
thetas = fread('~/Downloads/transfer/abalone.25kb_by_5kb.pestPG')
ggplot(thetas, aes(x = WinCenter, y = tP)) +
    geom_point() +
    theme_classic()
```

![image](https://github.com/user-attachments/assets/dd0d50f8-5a6d-4ce6-aea1-6ff4ff6f09b0)


** Exercises **: 
1) How do we rescale theta pi so that it's reflected as a percentage, as is often reported?
2) What are some simple solutions to make sure that the values here aren't driven by poor quality regions of the genome?

