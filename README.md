# G3C

## What is G3C?

G3C stands for **G**enetic **C**orrelation heterogeneity test for **C**ausality. It is a tool for establishing causal relationships between complex traits, for which GWAS have been conducted. It utilizes modeling and partition of estimated genetic correlations. The method is described in:

**Shen et al. (2017) Heterogeneity in genetic correlation detects causal factors for complex traits. _Submitted_.**

## What data is required?

In order to use G3C for testing causality, you need genome-wide association study (GWAS) summary statistics for a chosen *exposure* phenotype in **one** population, and GWAS summary statistics for a chosen *outcome* phenotype in **two** populations. These populations can have different levels of overlapping. 

## What tools are required?

* Python/LDSC: [Install](https://github.com/bulik/ldsc)
* R: [Install](http://www.r-project.org)
* G3C: Download and install the R package for
    - Windows: **[G3C.shen.se/G3C_1.1.zip](http://G3C.shen.se/G3C_1.1.zip) **
    - Mac OS: **[G3C.shen.se/G3C_1.1.tgz](http://G3C.shen.se/G3C_1.1.tgz) **
    - Linux: **[G3C.shen.se/G3C_1.1.tar.gz](http://G3C.shen.se/G3C_1.1.tar.gz) **

## How to use?

### Step 1: Estimation of genetic correlations

Before the test, please use [LD Score regression tool LDSC](https://github.com/bulik/ldsc/wiki/Heritability-and-Genetic-Correlation) to munge your summary statistics and estimate the genetic correlation between the exposure and outcome phenotypes using the two sources that you have.

You obtain **three** munged summary statistics files and **two** genetic correlation estimation log files, which
will be passed onto the next step.

### Step 2a: A single command to test the causal relationship

For users unfamiliar with R or programming languages, we provide an easy-to-use single command line version of **G3C**. After installation of the required tools above, simply download the R wrapper file: **G3C.shen.se/G3C.R **

Let's illustrate the usage using an example, where *education attainment (EA)* is regarded as exposure and *body mass index (BMI)* as outcome. Example files are ready for [download](https://www.dropbox.com/sh/to86shthsi5h2th/AAAvb4AitqYORu6baWtKl3_sa?dl=0). These are the files produced by **LDSC** using GWAS summary statistics.

The most naive way of testing the causal relationship is as follows, where only the genetic correlation estimation results are used. This is to assume that the two genetic correlation estimates are independent. Therefore, it is only applicable when you are certain that your **two** populations for the outcome phenotype GWAS are nearly **independent**.

Place **G3C.R** in the same folder as your files produced by **LDSC**, and run:

```
Rscript G3C.R "rg1.log='EDU_BMI1.log' rg2.log='EDU_BMI2.log'"
```

We can see that the resulted p-value is small, indicating a causal effect of EA on BMI. You may also name your exposure and outcome phenotypes as follows.

```
Rscript G3C.R "rg1.log='EDU_BMI1.log' rg2.log='EDU_BMI2.log' exposure='EA' outcome='BMI'"
```

Note you can also print all the output into a file, by specifying **outfile='FILENAME'** as an additional input parameter. 

We are uncertain how much overlap the two BMI GWAS populations have. **G3C** is able to account for any overlap by estimating the correlation between the two genetic correlation estimates. All you need is to specify the munged summary statistics produced by **LDSC**. This will make the computation a bit more time consuming.

```
Rscript G3C.R "rg1.log='EDU_BMI1.log' rg2.log='EDU_BMI2.log' exposure.sumstats='EDU.sumstats.gz' outcome.sumstats1='BMI1.sumstats.gz' outcome.sumstats2='BMI2.sumstats.gz' exposure='EA' outcome='BMI'"
```

As we can see, due to the overlap between the two BMI GWAS populations, there is a positive correlation between the two genetic correlation estimates. After accounting for such correlation, the power of detecting causality is improved. Note that when the two genetic correlation estimates are negatively correlated, the naive test above would produce more false positives.

### Step 2b: Testing the causal relationship in R

For R users, simply load the **G3C** package in R:

```
require(G3C)
```

A help document can be called via:

```
?g3c
```

Let's illustrate the use of the **G3C** package using the same examples as above. The only difference is that we directly call the R functions.

First, we run the estimation procedure assuming the two populations where BMI GWAS were performed are independent. As above, only the genetic correlation estimation log files are needed. Let's also specify the trait names.

```
setwd('~/Dropbox/G3CExampleData')
res <- g3c(rg1.log = 'EDU_BMI1.log', 
           rg2.log = 'EDU_BMI2.log', 
           exposure = 'Education Attainment', 
           outcome = 'BMI')
```

As you can read in the function help document, the assigned results to object **res** contains a list of variables, including:

* **rg.diff** The estimated difference between two genetic correlation estimates.
* **se** The standard error of **rg.diff**.
* **p.value** The p-value testing the null hypothesis of **rg.diff** = 0.
* **r.rg** The estimated correlation between two genetic correlation estimates.

```
res
```

Second, let's account for the possible overlap between the two BMI GWAS populations, so that the standard error of the estimate can be properly adjusted.

```
setwd('~/Dropbox/G3CExampleData')
res <- g3c(rg1.log = 'EDU_BMI1.log', 
           rg2.log = 'EDU_BMI2.log', 
           exposure.sumstats = 'EDU.sumstats.gz', 
           outcome.sumstats1 = 'BMI1.sumstats.gz',
           outcome.sumstats2 = 'BMI2.sumstats.gz',
           exposure = 'Education Attainment', 
           outcome = 'BMI')
```

When it is known that the two GWAS populations for the outcome phenotype are independent, the first run is recommended, as it takes no time to process the munged summary association data. 
