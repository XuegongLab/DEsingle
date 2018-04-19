# DEsingle

### *Zhun Miao*
### *2017-06-06*

![Logo](https://github.com/miaozhun/DEsingle/blob/master/inst/DEsingle_LOGO.png?raw=true)


## Introduction

`DEsingle` is an R package for differential expression (DE) analysis of single-cell RNA-seq (scRNA-seq) data. It will detect differentially expressed genes between two groups of cells in a scRNA-seq raw read counts matrix.

`DEsingle` employs the Zero-Inflated Negative Binomial model for differential expression analysis. By estimating the proportion of real and dropout zeros, it not only detects DE genes at higher accuracy but also subdivides three types of differential expression with different regulatory and functional mechanisms.

For more information, please refer to the [original manuscript](https://www.biorxiv.org/content/early/2018/03/26/173997) by *Zhun Miao, Ke Deng, Xiaowo Wang and Xuegong Zhang*.


## Installation

To install `DEsingle` from Bioconductor:

```{r Installation from Bioconductor, eval = FALSE}
BiocInstaller::biocLite("DEsingle")
```

To install the developmental version from GitHub:

```{r Installation from GitHub, eval = FALSE}
devtools::install_github("miaozhun/DEsingle", build_vignettes = TRUE)
```

To load the installed `DEsingle` in R:

```{r Load DEsingle, eval = FALSE}
library(DEsingle)
```


## Input

`DEsingle` takes two inputs: `counts` and `group`.

The input `counts` is a scRNA-seq **raw read counts matrix** or a `SingleCellExperiment` object which contains the read counts matrix. The rows of the matrix are genes and columns are samples/cells.

The other input `group` is a vector of factor which specifies the two groups in the matrix to be compared, corresponding to the columns in `counts`.


## Test data

Users can load the test data in `DEsingle` by

```{r Load TestData}
library(DEsingle)
data(TestData)
```

The toy data `counts` in `TestData` is a scRNA-seq read counts matrix which has 200 genes (rows) and 150 cells (columns).

```{r counts}
dim(counts)
counts[1:6, 1:6]
```

The object `group` in `TestData` is a vector of factor which has two levels and equal length to the column number of `counts`.

```{r group}
length(group)
summary(group)
```


## Usage

### With read counts matrix input

Here is an example to run `DEsingle` with read counts matrix input:

```{r demo1, eval = FALSE}
# Load library and the test data for DEsingle
library(DEsingle)
data(TestData)

# Specifying the two groups to be compared
# The sample number in group 1 and group 2 is 50 and 100 respectively
group <- factor(c(rep(1,50), rep(2,100)))

# Detecting the differentially expressed genes
results <- DEsingle(counts = counts, group = group)

# Dividing the differentially expressed genes into 3 categories
results.classified <- DEtype(results = results, threshold = 0.05)
```

### With SingleCellExperiment input

The [`SingleCellExperiment`](http://bioconductor.org/packages/SingleCellExperiment/) class is a widely used S4 class for storing single-cell genomics data. `DEsingle` also could take the `SingleCellExperiment` data representation as input.

Here is an example to run `DEsingle` with SingleCellExperiment input:

```{r demo2, eval = FALSE}
# Load library and the test data for DEsingle
library(DEsingle)
library(SingleCellExperiment)
data(TestData)

# Convert the test data in DEsingle to SingleCellExperiment data representation
sce <- SingleCellExperiment(assays = list(counts = as.matrix(counts)))

# Specifying the two groups to be compared
# The sample number in group 1 and group 2 is 50 and 100 respectively
group <- factor(c(rep(1,50), rep(2,100)))

# Detecting the differentially expressed genes with SingleCellExperiment input sce
results <- DEsingle(counts = sce, group = group)

# Dividing the differentially expressed genes into 3 categories
results.classified <- DEtype(results = results, threshold = 0.05)
```


## Output

The output of `DEsingle` is a matrix containing the differential expression (DE) analysis results, whose rows are genes and columns contain the following items:

* `theta_1`, `theta_2`, `mu_1`, `mu_2`, `size_1`, `size_2`, `prob_1`, `prob_2`: MLE of the zero-inflated negative binomial distribution's parameters of group 1 and group 2.
* `total_mean_1`, `total_mean_2`: Mean of read counts of group 1 and group 2.
* `foldChange`: total_mean_1/total_mean_2.
* `norm_total_mean_1`, `norm_total_mean_2`: Mean of normalized read counts of group 1 and group 2.
* `norm_foldChange`: norm_total_mean_1/norm_total_mean_2.
* `chi2LR1`: Chi-square statistic for hypothesis testing of H0.
* `pvalue_LR2`: P value of hypothesis testing of H20 (Used to determine the type of a DE gene).
* `pvalue_LR3`: P value of hypothesis testing of H30 (Used to determine the type of a DE gene).
* `FDR_LR2`: Adjusted P value of pvalue_LR2 using Benjamini & Hochberg's method (Used to determine the type of a DE gene).
* `FDR_LR3`: Adjusted P value of pvalue_LR3 using Benjamini & Hochberg's method (Used to determine the type of a DE gene).
* `pvalue`: P value of hypothesis testing of H0 (Used to determine whether a gene is a DE gene).
* `pvalue.adj.FDR`: Adjusted P value of H0's pvalue using Benjamini & Hochberg's method (Used to determine whether a gene is a DE gene).
* `Remark`: Record of abnormal program information.
* `Type`: Types of DE genes. *DEs* represents differential expression status; *DEa* represents differential expression abundance; *DEg* represents general differential expression.
* `State`: State of DE genes, *up* represents up-regulated; *down* represents down-regulated.


## Parallelization

`DEsingle` integrates parallel computing function with [`BiocParallel`](http://bioconductor.org/packages/BiocParallel/) package. Users could just set `parallel = TRUE` in `DEsingle` to enable parallelization and leave the `BPPARAM` parameter alone.

```{r demo3, eval = FALSE}
# Load library
library(DEsingle)

# Detecting the differentially expressed genes in parallelization
results <- DEsingle(counts = counts, group = group, parallel = TRUE)
```

Advanced users could use a `BiocParallelParam` object from package `BiocParallel` to fill in the `BPPARAM` parameter to specify the parallel back-end to be used and its configuration parameters.

### For Unix and Mac users

The best choice for Unix and Mac users is to use `MulticoreParam` to configure a multicore parallel back-end:

```{r demo4, eval = FALSE}
# Load library
library(DEsingle)
library(BiocParallel)

# Set the parameters and register the back-end to be used
param <- MulticoreParam(workers = 18, progressbar = TRUE)
register(param)

# Detecting the differentially expressed genes in parallelization with 18 cores
results <- DEsingle(counts = counts, group = group, parallel = TRUE, BPPARAM = param)
```

### For Windows users

For Windows users, use `SnowParam` to configure a Snow back-end is a good choice:

```{r demo5, eval = FALSE}
# Load library
library(DEsingle)
library(BiocParallel)

# Set the parameters and register the back-end to be used
param <- SnowParam(workers = 8, type = "SOCK", progressbar = TRUE)
register(param)

# Detecting the differentially expressed genes in parallelization with 8 cores
results <- DEsingle(counts = counts, group = group, parallel = TRUE, BPPARAM = param)
```

See the [*Reference Manual*](https://bioconductor.org/packages/release/bioc/manuals/BiocParallel/man/BiocParallel.pdf) of [`BiocParallel`](http://bioconductor.org/packages/BiocParallel/) package for more details of the `BiocParallelParam` class.


## Visualization of results

Users could use the `heatmap()` function in `stats` or `heatmap.2` function in `gplots` to plot the heatmap of the DE genes DEsingle found, as we did in Figure S3 of the [*manuscript*](https://www.biorxiv.org/content/early/2018/03/26/173997).


## Interpretation of results

For the interpretation of results when `DEsingle` applied to real data, please refer to the *Three types of DE genes between E3 and E4 of human embryonic cells* part in the [*Supplementary Materials*](https://www.biorxiv.org/content/biorxiv/suppl/2018/03/26/173997.DC4/173997-1.pdf) of our [*manuscript*](https://www.biorxiv.org/content/early/2018/03/26/173997).


## Help

Use `browseVignettes("DEsingle")` to see the vignettes of `DEsingle` in R after installation.

Use the following code in R to get access to the help documentation for `DEsingle`:

```{r help1, eval = FALSE}
# Documentation for DEsingle
?DEsingle
```

```{r help2, eval = FALSE}
# Documentation for DEtype
?DEtype
```

```{r help3, eval = FALSE}
# Documentation for TestData
?TestData
?counts
?group
```

You are also welcome to contact the author by email for help.


## Author

*Zhun Miao* <<miaoz13@mails.tsinghua.edu.cn>>

MOE Key Laboratory of Bioinformatics; Bioinformatics Division and Center for Synthetic & Systems Biology, TNLIST; Department of Automation, Tsinghua University, Beijing 100084, China.

