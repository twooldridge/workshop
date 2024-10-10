# Intro
When you have low coverage data (<5X), working with genotype likelihoods instead of variant calls (VCFs) may be preferred. At the least, you can verify some of your analyses on SNPs with the analogous likelihood-based approach. Fair warning, **there will be differences** between results from the two approaches - while this is expected to some extent, it has led to headaches for many bioinformaticians. At some point, you will have to use your own discretion and sanity checks to determine what differences are meaningful and which ones are trivial.

Now, with that warning aside, the main way we'll be working with genotype likelihoods is via [ANGSD](https://www.popgen.dk/angsd/index.php/ANGSD)
