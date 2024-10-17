# Table of Contents

1. [snpArcher QC dashboard](#QC-dashboard)<br><br>
2. [Exploring other snpArcher output](#Exploring-other-snpArcher-output)<br>
  2.1 [Breakdown of output](#Where-are=the-results?)<br>
  2.2 [Genome masks and callable sites](#Callable-sites)<br>
  2.3 [Other alignment metrics](#Alignment-summary-metrics)<br>
  2.4 [SNP QC metrics](#SNP-QC-metrics)<br><br>
3. [Using the VCF output](#Working-with-VCF-data)<br>
  3.1 [Quick vcf stats](#Quick-stats)<br>
  3.2 [More complex filtering](#Filtering)<br>
  3.3 [Re-running PCA with subset of samples](#PCA-on-a-smaller-sample-set)<br>
  3.4 [LD decay](#LD-decay)<br>
  3.5 [Local PCA](#Local-pca-with-lostruct)

---
# QC dashboard

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

---

# Exploring snpArcher output

## Where are the results?

Here is a quick list of all the content in the results folder, with my own ranking of how important each thing is:

```
Key:
# IMPORTANT
## KINDA IMPORTANT
### MOSTLY IGNORE

bams/                                   # The final bam files used as input for all variant calling
callable_sites/                           ## Sequencing cov. files used to identify good/bad regions.
cracherodi_callable_sites.bed           # Mask of 'good' regions of the genome used as a filter
cracherodi_clean_indels.vcf.gz              ### Filtered vcf of just indels
cracherodi_clean_indels.vcf.gz.tbi          ### ^Index
cracherodi_clean_snps.vcf.gz                ### Filtered vcf of just snps
cracherodi_clean_snps.vcf.gz.tbi            ### ^Index
cracherodi_filtered.vcf.gz              # Primary vcf used for all downstream analyses
cracherodi_filtered.vcf.gz.csi          # ^Index
cracherodi_raw.vcf.gz                     ## Unfiltered vcf, hot off of variant calling - useful for sanity checking
cracherodi_raw.vcf.gz.tbi                 ## ^Index
data/                                       ### Files related to the reference genome
filtered_fastqs/                            ### Fastq files filtered by fastp
genmap/                                     ### Mappability maps
genmap_index/                               ### Files for genmap to run
genomics_db_import/                         ### Genomics DB workspace - GATK step between haplotype caller and cohort variant calling 
gvcfs_norm/                                 ### Whole-genome GVCFs, output of haplotype caller
interval_gvcfs/                             ### Interval-level GVCFs to stitch together ^
intervals/                                  ### Files related to genomic intervals
postprocess/                                ### Misc. files. Ignore.
QC/                                     # QC dashboard and all files used to create it
summary_stats/                          ## Files for each sample detailing fastp, alignment, and coverage metrics
```




## Callable sites

A key QC aspect of snpArcher is that it determines what regions of the genome are 'callable' based on 1) sequencing coverage, and 2) mappability to the reference genome. Will discuss each of these in turn.

### Coverage

In the `callable_sites` directory there are a whole bunch of files. Essentially, what snpArcher is doing behind the scenes is determining coverage metrics for each sample (`*mosdepth*txt`), and using these parameters to determine which regions of the genome have too few or too many reads. Such regions will be used to filter out snps and excluded for some of the calculations (e.g. heterozygosity). Remember, this is a parameter we set in `config.yaml`. Let's take a look at `callable_sites/cracherodi_callable_sites_cov.bed`:

```
JAJLRC010000027.1	7	2691
JAJLRC010000027.1	2723	2980
JAJLRC010000027.1	3048	3074
JAJLRC010000027.1	3120	3358
JAJLRC010000027.1	3434	3494
JAJLRC010000027.1	3496	3529
JAJLRC010000027.1	3530	5242
JAJLRC010000027.1	5272	5292
JAJLRC010000027.1	5309	5310
JAJLRC010000027.1	5311	5392
```

These are all the 'good' based on sequence coverage. Regions that are separated by less than 100bp are **merged**. This is also a parameter we had set in the `config.yaml` file! To see this in action, compare the first few lines of to what we see in the main results directory under `cracherodi_callable_sites.bed`:

```
JAJLRC010000027.1	7	5392
JAJLRC010000027.1	5636	5796
JAJLRC010000027.1	5949	16442
```

And there you have it! That's how the coverage-based mask works. 


### Genmap

Now, the second piece of the puzzle - the mappability mask created by [genmap](https://github.com/cpockrandt/genmap). This program determines  which regions of the genome to which good quality reads can map. As we now know, `snpArcher` uses this internally for QC, but it can be very useful for you to use when interpreting and filtering the results of **other** analyses. 

Let's look at the first few lines of the genmap results in `genmap/sorted_mappability.bg `:

```
JAJLRC010000027.1	0	22530	1
JAJLRC010000027.1	22530	22583	0.5
JAJLRC010000027.1	22583	22596	0.333333
JAJLRC010000027.1	22596	22607	0.25
JAJLRC010000027.1	22607	22631	0.333333
JAJLRC010000027.1	22631	22637	0.5
JAJLRC010000027.1	22637	23160	1
JAJLRC010000027.1	23160	23213	0.5
JAJLRC010000027.1	23213	23226	0.333333
JAJLRC010000027.1	23226	23237	0.25
```

Here, the good regions have a mapping score of '1' (4th column), indicating that 150bp kmers in that region are unique, and not found elsewhere in the genome. snpArcher uses these regions to create the 2nd callable sites mask, which is in `callable_sites/cracherodi_callable_sites_map.bed`

```
JAJLRC010000027.1	0	22530
JAJLRC010000027.1	22637	23160
JAJLRC010000027.1	23267	66863
JAJLRC010000027.1	67231	67511
JAJLRC010000027.1	67879	128758
JAJLRC010000027.1	128935	129079
JAJLRC010000027.1	129193	130698
JAJLRC010000027.1	130964	133138
JAJLRC010000027.1	133284	133761
JAJLRC010000027.1	133876	137261
```

Now, as you might be guessing, the final callable sites file `cracherodi_callable_sites.bed` is the combination of the `cov` and `map` files. We'll talk about that more in a bit, but let's look more closely at what the genmap results say about the reference genome.

We'll do this in `R`:

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

> **Exercise**:
> 
> How much do the coverage and map masks overlap? 

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

## SNP QC metrics

If we're curious about what SNPs got filtered and what didn't, we can take advantage of the file `QC/cracherodi_snpqc.txt` which is already nicely formatted with some of the key metrics used for SNP filtering, at least by snpArcher. 

> [!IMPORTANT]
> This file doesn't represent all SNPs in the clean vcfs, but just the snps that are present in `QC/cracherodi.pruned.vcf.gz`. This "pruned" vcf is the one used for PCA and some of the other stats were you don't want SNPs linked by LD. Regardless, this file is still a good representation of variant characteristics in our data

Let's take a look:

```
CHROM  POS  ID  AF  QUAL  ReadPosRankSum  FS  SOR  MQ  MQRankSum
JAJLRC010000027.1	26	.	0.536	410.83	0.967	0	2.833	52.24	-0.431
JAJLRC010000027.1	77	.	0.067	62.42	.	0	2.303	60	.
JAJLRC010000027.1	153	.	0.071	51.31	.	0	0.693	49.82	.
JAJLRC010000027.1	168	.	0.094	72.08	-0.674	0	0.223	54	-0.674
JAJLRC010000027.1	211	.	0.233	462.71	0.967	3.256	0.61	53.88	0
```
By default, snpArcher removes SNPs with QUAL < 30, ReadPosRankSum < -8.0, FS > 60, SOR > 3, MQ < 40, and MQRankSum < -12.5. A couple of these rules change for indels.

It can be interesting to explore the distributions of some of these metrics, and get a sense of overall data quality. Let's load this in R and visualize:

```
snpqc =
  ## Read and format
  fread('~/Downloads/transfer/cracherodi_snpqc.txt') %>%
  set_colnames(c('chrom','pos','ID','AF','QUAL','ReadPosRankSum','FS','SOR','MQ','MQRankSum')) %>%
  dplyr::select(!ID) %>%
  ## Change missing values '.' to NA
  mutate(across(!chrom,~as.numeric(.))) %>%
  ## Reshape data frame
  pivot_longer(-c(chrom,pos)) 

## Now plot
ggplot(snpqc) +
  geom_density(aes(x = value)) +
  facet_wrap(~name, scales = 'free')
```
![image](https://github.com/user-attachments/assets/ed08dfc6-3f8a-430c-a5bd-c84758090472)

As you can see by these distributions, these represent the metrics of SNPs that were already filtered. If we want a report on the raw data, we'll have to do some extra work (see 'More complex filtering').

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

## Filtering

Ok, let's return to our earlier exercise where we looked at some of the metrics of our SNP calls. Let's say we want to take a deeper dive into what SNPs failed our filters, and look at a few other metrics in the INFO field as well. `bcftools` is a great way to accomplish this:

```
bcftools query -f '%CHROM\t%POS\t%ID\t%INFO/AF\t%QUAL\t%INFO/ReadPosRankSum\t%INFO/FS\t%INFO/SOR\t%INFO/MQ\t%INFO/MQRankSum\t%INFO/QD\t%INFO/ExcessHet\n' cracherodi_raw.vcf.gz > raw_vcf_stats.txt
```

Now, we can perform a similar exercise as before and plot the distributions of these metrics:

```
snpqc =
  ## Read and format
  fread('~/Downloads/transfer/raw_vcf_stats.txt') %>%
  set_colnames(c('chrom','pos','ID','AF','QUAL','ReadPosRankSum','FS','SOR','MQ','MQRankSum','QD','ExcessHet')) %>%
  dplyr::select(!ID) %>%
  ## Change missing values '.' to NA
  mutate(across(!chrom,~as.numeric(.))) %>%
  ## Reshape data frame
  pivot_longer(-c(chrom,pos)) 

## Now plot
  ggplot(snpqc) +
      geom_density(aes(x = value)) +
      facet_wrap(~name, scales = 'free')
```

![image](https://github.com/user-attachments/assets/7b1eda97-cea3-42f7-9a8d-9c0acb83de40)

Based on these results, we can definitely see how some of the filters snpArcher implements, for excluding variants with MQ<40, will remove a small proportion of variants. While we haven't done any explicit calculations yet, this would suggest that overall we're not losing a bunch of data to filters. 

You might imagine a scenario where, after seeing these, you want to
a) Implement stricter filters (higher MQ?)
b) Implement additional filters. 
      - People will often remove sites with QD < 2. If I had to guess, that wouldn't have much of an effect here
      - Implement a hardy-weinberg based filter. The small bump in allele frequency at 0.5 is likely an artifact due to mapping to homologs. If we were very concerned, we could try to reduce that by  removing variants violating some HWE threshold. However, you need to be careful with the nature of your dataset and the downstream analysis. Filtering based on HWE might not make sense if there is serious population structure, or you're doing a selection scan wwehre you want to identify such loci. 

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
