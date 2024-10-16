# Table of Contents

1. [snpArcher QC dashboard](#QC)<br><br>
2. [Exploring other snpArcher output](#Exploring-other-snpArcher-output)<br>
  2.1 [Genmap genome mask](#Genmap)<br>
  2.2 [Other alignment metrics](#Alignment-summary-metrics)<br>
  2.3 [Visualizing heterozygosity on a sample map](#Heterozygosity-on-a-map)<br><br>
3. [Working with VCF data](#Working-with-VCF-data)<br>
  3.1 [Quick vcf stats](#Quick-stats)<br>
  3.2 [Re-running PCA with subset of samples](#PCA-on-a-smaller-sample-set)<br>
  3.3 [LD decay](#LD-decay)<br>
  3.4 [Local PCA](#Local-pca-with-lostruct)

---
# QC

In your `snpArcher` `results/cracherodii` folder you should see something that looks like this:

<img width="908" alt="image" src="https://github.com/user-attachments/assets/e350f777-b5f2-4570-b883-fa531daf0700">

The first thing we'll take a look at is the overall QC data located in `QC/cracherodi_qc.html`. Let's transfer this from the cluster to our local machine, and examine in a browser. 

> [!TIP]
> **Organizational tip** for server -> local transfers.
>  I like to create a folder in my project directory called something like `transfer`, and put the file I want in that directory. Then, on my local machine, I can just always enter the same command:

```
scp -r ${USERNAME}@${SERVER}:${WORKDIR}/transfer ~/Downloads
```

Where you replace all those variables with your server-specific setup. 

Now, the results!

![image](https://github.com/user-attachments/assets/dd8f9fd5-e8b1-4384-a6ca-c4a7523b9483)

Okay, this dashboard is **extremely** useful for initial exploration of the data and making sure everything looks ok. For example, no weird outliers on the PCA. The reality is that you will likely end up re-running many of these analyses yourself as you continue to explore your data and remove individuals, variants, etc. So think of this as what the name implies: a QC tool. All the data that goes into the QC dashboard can also be found in the `QC` folder. For example, if you want a simple `.txt` file of sample depth, go to `QC/cracherodi.idepth`

Now, we'll take a few moments to walk through this dashboard and discuss what everything means.

Discussion topics:

1. PC x Depth
2. Mapping rate
3. Relatedness

---

# Exploring other snpArcher output

## Callable sites

## Coverage


### Genmap

Now, let's look a little deeper at what `snpArcher` generates that doesn't show up on the QC dashboard. One particularly useful piece of the pipeline is the [genmap](https://github.com/cpockrandt/genmap) output, which computes genome mappability, or the proportion of the genome to which good quality reads can map. `snpArcher` uses this internally for QC, but it can be very useful for you to use when interpreting and filtering the results of **other** analyses. Let's take a look in `R`:

```
genmap = 
  fread('~/Downloads/transfer/sorted_mappability.bg') %>%
  set_colnames(c('chrom','start','end','score'))
head(genmap)
               chrom start   end    score
1: JAJLRC010000027.1     0 22530 1.000000
2: JAJLRC010000027.1 22530 22583 0.500000
3: JAJLRC010000027.1 22583 22596 0.333333
4: JAJLRC010000027.1 22596 22607 0.250000
5: JAJLRC010000027.1 22607 22631 0.333333
6: JAJLRC010000027.1 22631 22637 0.500000
```

How much of the genome is perfectly mappable (score = 1)?

```
> genmap %>%
+     mutate(length = end - start - 1) %>%
+     group_by(score) %>%
+     summarize(T_length = sum(length)) %>%
+     arrange(desc(T_length)) %>%
+     mutate(pct = T_length/sum(T_length))
# A tibble: 12 × 3
    score T_length        pct
    <dbl>    <dbl>      <dbl>
 1 1       5268252 0.906     
 2 0.5      473989 0.0815    
 3 0.333     36214 0.00623   
 4 0.25      17514 0.00301   
 5 0.2        7330 0.00126   
 6 0.167      4134 0.000711  
 7 0.1        1520 0.000261  
 8 0.143      1407 0.000242  
 9 0.125      1256 0.000216  
10 0.0909     1056 0.000182  
11 0.111       846 0.000146  
12 0.0833       12 0.00000206
```


> **Question**:
> Why do I need to calculate length as `end - start - 1`? Why is the file formatted that way?


> **Exercise**:
> 
> Write a bed-formatted file of all regions with score == 1 that you could use for filtering other types of data

## Alignment summary metrics

The QC dashboard only reports some alignment stats. What if we wanted to know about the secondary mapping rate? These data are in the `summary_stats` folder. Once you have all the `*_AlnSumMets.txt` files in a place accessible by RStudio, try the following:

```
paths = list.files('~/Downloads/transfer/', pattern = 'AlnSum', full.names = T)
aln_stats =
  bind_rows(lapply(paths, 
                   function(PATH){fread(PATH) %>% mutate(sample = basename(PATH) %>% gsub('_AlnSumMets.txt','',.))
    })) %>%
  set_colnames(c('value','second_value','field','sample'))

head(aln_stats)
    value second_value                                     field sample
1: 295582            0 total (QC-passed reads + QC-failed reads)  ASI03
2: 289146            0                                   primary  ASI03
3:   6436            0                                 secondary  ASI03
4:      0            0                             supplementary  ASI03
5:   8522            0                                duplicates  ASI03
6:   8522            0                        primary duplicates  ASI03
```

> **Exercise**:
>
>  This format is kind of messy. How can we get the proportion of reads with secondary mappings?<br>

One possible solution:
<details>

```
## Formatting
secondary =
  aln_stats %>%

  ## Get rid of distracting columns and fields
  filter(field %in% c('total (QC-passed reads + QC-failed reads)','secondary')) %>%
  dplyr::select(value,field,sample) %>%

  ## 'Spread' and further clean the data
  pivot_wider(values_from = value, names_from = field) %>%
  set_colnames(c('sample','total','secondary')) %>%

  ## Calculate percentage
  mutate(pct_secondary = as.numeric(secondary)/as.numeric(total))

head(secondary)
 # A tibble: 6 × 4
  source   total  secondary pct_secondary
  <chr>    <chr>  <chr>             <dbl>
1 ASI03    295582 6436             0.0218
2 BSCO001C 357304 6026             0.0169
3 BSCO001J 368246 8394             0.0228
4 BSCO001O 353807 7899             0.0223
5 CAR01    332676 5134             0.0154
6 CAR11    105687 1513             0.0143

## Plotting
ggplot(secondary) +
    geom_bar(aes(x = sample, y = pct_secondary), stat = 'identity') +
    theme(axis.text.x = element_text(angle = 90))
```
![image](https://github.com/user-attachments/assets/ce4ff33b-2d49-4bba-914d-5674f92c023b)


</details>


## Heterozygosity on a map



---

# Working with VCF data

## Quick stats

Let's quickly inspect the VCFs to see what the differences are between them. We see that the `raw` folder still retains ALL snps and indels produced by the pipeline, but you can see some of the sites have been flagged by quality filters:

```
bgzip -dc cracherodi_raw.vcf.gz | grep -v "##"  | less -S****
```

<img width="1181" alt="image" src="https://github.com/user-attachments/assets/79b84c93-868c-45a1-b2ef-494c05f9d974">

 And those sites have been removed in the `filtered` vcf, which is what we'll use for additional downstream analyses.

<img width="1168" alt="image" src="https://github.com/user-attachments/assets/df5c72a1-da7d-439a-9eb0-37b016b3dfff">

> **Exercise**:
> 
> How do we quickly find, on the command line, the # of variants that are in a VCF?


## PCA on a smaller sample set
Let's say we're really interested in relationships just within our northern samples, and we want a PCA of just them. We could filter samples in the `QC/cracherodi.eigenvec` file, but the more correct approach, especially if we want to report these data, would be to re-create a PCA with just the SNPs for those samples. There are many ways to do this, but the general approach I like is:

1. Filter `VCF` files with `bcftools`
2. PCA with `plink`
3. Plot with `R`

So here is the easiest way to do that. Working from our project directory

```
mkdir -p PCA
## Select samples of interest. What is the purpose of tail -n+2?
tail -n+2 snpArcher/config/samples.csv | awk -F"," '($10 > 36) {print $1}' > PCA/north_samples.txt

## Filter VCF accordingly. 
bcftools view -S PCA/north_samples.txt -q 0.05:minor results/cracherodii/cracherodi_clean_snps.vcf.gz |\
bcftools +prune -n 1 -N rand -w 1kb -O z \
> PCA/north_samples_snps.vcf.gz && tabix PCA/north_samples_snps.vcf.gz
```

Okay, let's walk through the above filtering steps, and why we do them. How many variants are we left with?


Now, we can use `plink` for the PCA:

```
plink --vcf PCA/north_samples_snps.vcf.gz --allow-extra-chr --pca --out PCA/north_samples_snps
```

And visualize in `R`:

```
PCA = 
  fread('~/Downloads/transfer/north_samples_snps.eigenvec') %>%
  set_colnames(c('sample','dummy',sapply(seq(1,ncol(.)-2),function(NUM){paste0("PC",NUM)})))

```

![image](https://github.com/user-attachments/assets/f35eb52b-d392-409f-b9c1-72d30976f51b)

> **Exercise**:
> 
> How can we use the snpArcher output to generate the same plot, but with samples color-coded by sequence depth?

# LD Decay

A key aspect of becoming acquainted with any new popgen system is an understanding of how linkage disequilibrium, or the correlation between SNPs, is a function of physical distance along the chromosome. By understanding this, it can inform how we 'prune' SNPs for analyses like PCA and help identify SNP-SNP correlations that are abnormal (and potentially interesting!). 

To get a basic sense of this, we'll use the program `PopLDDecay`. Implementation is very simple:

```
mkdir -p LD
PopLDdecay -InVCF snpArcher/results/cracherodii/cracherodi_clean_snps.vcf.gz -OutStat LD/decay -MaxDist 100 -OutType 1
```

Hmm, seems like this might take a minute or two to run. Alternatively, we can submit this as a `slurm` job, and we'll check on the results later:


```
CMD="PopLDdecay -InVCF snpArcher/results/cracherodii/cracherodi_clean_snps.vcf.gz -OutStat LD/decay -MaxDist 100 -OutType 1"
sbatch \
--mem=5000 \
--time=00:20:00 \
-p long \
-N 1 -n 1 -c 1 \
-J decay \
-e logs/decay.e \
-o logs/decay.o \
--wrap="$CMD"
```

Once completed, let's take a look at the results file `LD/decay.stat.gz` in R:

```
ld = fread('~/Downloads/transfer/decay.stat.gz')
ggplot(ld) +
    geom_point(aes(x = `#Dist`, y = `Mean_r^2`))
```

![image](https://github.com/user-attachments/assets/ff8a73de-ad98-4177-b0d3-48e219a9670f)

**Exercise**: r^2 = 0.2 is often a threshold used for considering snps unlinked. We can see that LD decays below that quickly. If we want to be extra conservative, at what physical distance (bp) does r^2 drop below 0.10?

## Local-pca-with-lostruct

[Lostruct](https://github.com/petrelharp/local_pca) functions as an R package, so we'll run all analyses via R. With the size of this scaffold, we can actually get the first PC windows pretty easily!

```
library(lostruct)
## Read in VCF and do windowed PCA
snps <- vcf_windower("~/Downloads/transfer/cracherodi_clean_snps.vcf.gz",size=1e3,type='snp')
pcs <- eigen_windows(snps,k=2) # Calculate for each window
windows <- region(snps)() 

## Some windows have NA values, let's remove those
goodrows = !rowSums(!is.finite(pcs))
goodwindows = windows[goodrows,]
goodpcs = pcs[goodrows, ]

## Now calculate distance between windows
pcdist <- pc_dist(goodpcs,npc=2) # Distance between windows
chr.mds <- cmdscale(pcdist, eig = TRUE, k = 2)
pts = chr.mds$points %>% set_colnames(c('mds1','mds2'))

## Let's plot these results
mds_results_clean = bind_cols(goodwindows, pts) %>% mutate(midpoint = ((end-start)/2) + start)
ggplot(mds_results_clean) +
    geom_point(aes(x = midpoint, y = mds1)) +
    theme_classic()
```

![image](https://github.com/user-attachments/assets/489ae4c4-7f00-4f12-8d86-33bd553d3451)

No striking conclusions to draw here! Typically you would look at this on a whole genome level, and do some sort outlier analyses to identify the interesting parts.

**Question** This is just a first pass analysis. Can you imagine parameters that might be nice to change as we explore the data?
