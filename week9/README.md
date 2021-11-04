---
title: "Week 9 Lab"
author: "Audrey Omidsalar"
date: "10/29/2021"
output:
  html_document:
    toc: yes
    toc_float: yes
    keep_md: yes
  github_document:
  always_allow_html: true
---


## Problem 1
#### Give yourself a few minutes to think about what you just learned. List three examples of problems that you believe may be solved using parallel computing, and check for packages on the HPC CRAN task view that may be related to it.
1. Statistics on large data sets: I recently was working on a project where I had to run correlation tests on a data set with around 50 columns and 40,000 rows. I could have done this much more efficiently using parallel computing. (bigstatsr package)
2. Spatial Transcriptomics / Seurat Clustering: This is done on single cell RNA-Seq data to define clusters in various clusters based on their expression. This can be done in parallel with the 'future' and 'Seurat' packages.
3. Repetitive Tasks: Processes that need to be run regularly (ex. every day, every hour) with large amounts of data can be made more efficient using parallel computing (parallel package). For example, this could simplify climate modeling on a state-wide or national level.

## Problem 2


```r
fun1 <- function(n = 100, k = 4, lambda = 4) {
  x <- NULL
  for (i in 1:n)
    x <- rbind(x, rpois(k, lambda))
  return(x)
}
fun1(5,10)
```

```
##      [,1] [,2] [,3] [,4] [,5] [,6] [,7] [,8] [,9] [,10]
## [1,]    4    3    1    3    5    4    5    4    3     3
## [2,]    5    1    5    3    1    3    1    7    4     0
## [3,]    1    2    3    5    2    6    5    7    5     0
## [4,]    2    5    6    3    5    5    3    3    4     5
## [5,]    3    3    4    4    3    2    6    4    1     2
```

```r
fun1alt <- function(n = 100, k = 4, lambda = 4) {
  matrix(rpois(n*k, lambda), nrow = n, ncol = k)
}

# Benchmarking
microbenchmark::microbenchmark(
  fun1(),
  fun1alt(), unit = 'relative'
)
```

```
## Unit: relative
##       expr      min       lq     mean   median       uq       max neval
##     fun1() 16.69486 21.90543 11.41709 21.43762 22.18993 0.9687642   100
##  fun1alt()  1.00000  1.00000  1.00000  1.00000  1.00000 1.0000000   100
```

### 2. Find the Column Max

```r
# Data Generating Process (10 x 10,000 matrix)
set.seed(1234)
x <- matrix(rnorm(1e4), nrow=10)

# Find each column's max value
fun2 <- function(x) {
  apply(x, 2, max)
}

fun2alt <- function(x) {
  ## position of the max value per row of x
  idx <- max.col(t(x))
  ## get the max value
  x[cbind(idx, 1:ncol(x))]
}
#Do we get the same values?
all(fun2(x) == fun2alt(x))
```

```
## [1] TRUE
```

```r
# Benchmarking
microbenchmark::microbenchmark(
  fun2(x),
  fun2alt(x), unit = 'relative'
)
```

```
## Unit: relative
##        expr     min      lq     mean   median       uq       max neval
##     fun2(x) 10.4924 8.05049 5.854933 8.093899 7.967874 0.8741729   100
##  fun2alt(x)  1.0000 1.00000 1.000000 1.000000 1.000000 1.0000000   100
```

## Problem 3

```r
my_boot <- function(dat, stat, R, ncpus = 1L) {
  
  # Getting the random indices
  n <- nrow(dat)
  idx <- matrix(sample.int(n, n*R, TRUE), nrow=n, ncol=R)
 
  # Making the cluster using `ncpus`
  # STEP 1: GOES HERE create cluster
  cl <- makePSOCKcluster(ncpus)
  
  # STEP 2: GOES HERE prepare cluster
  clusterSetRNGStream(cl,123)
  clusterExport(cl, c("stat", "dat", "idx"), envir = environment())
    # STEP 3: THIS FUNCTION NEEDS TO BE REPLACES WITH parLapply
  ans <- parLapply(cl = cl, seq_len(R), function(i) {
    stat(dat[idx[,i], , drop=FALSE])
  })
  
  # Coercing the list into a matrix
  ans <- do.call(rbind, ans)
  
  # STEP 4: GOES HERE stop cluster
  stopCluster(cl)
  ans
  
}
```


```r
# Bootstrap of an OLS
my_stat <- function(d) coef(lm(y ~ x, data=d))

# DATA SIM
set.seed(1)
n <- 500; R <- 5e3

x <- cbind(rnorm(n)); y <- x*5 + rnorm(n)

# Checking if we get something similar as lm
ans0 <- confint(lm(y~x))
ans1 <- my_boot(dat = data.frame(x, y), my_stat, R = R, ncpus = 2L)
# You should get something like this
t(apply(ans1, 2, quantile, c(.025,.975)))
```

```
##                   2.5%      97.5%
## (Intercept) -0.1395732 0.05291612
## x            4.8686527 5.04503468
```

```r
##                   2.5%      97.5%
## (Intercept) -0.1372435 0.05074397
## x            4.8680977 5.04539763
ans0
```

```
##                  2.5 %     97.5 %
## (Intercept) -0.1379033 0.04797344
## x            4.8650100 5.04883353
```

```r
##                  2.5 %     97.5 %
## (Intercept) -0.1379033 0.04797344
## x            4.8650100 5.04883353
system.time(my_boot(dat = data.frame(x, y), my_stat, R = 4000, ncpus = 1L))
```

```
##    user  system elapsed 
##   0.116   0.020   6.129
```

```r
system.time(my_boot(dat = data.frame(x, y), my_stat, R = 4000, ncpus = 2L))
```

```
##    user  system elapsed 
##   0.184   0.030   3.829
```

