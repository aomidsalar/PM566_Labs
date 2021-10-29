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



## Problem 2

```r
fun1 <- function(n = 100, k = 4, lambda = 4) {
  x <- NULL
  for (i in 1:n)
    x <- rbind(x, rpois(k, lambda))
  return(x)
}
#fun1(5,10)
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
##       expr      min       lq     mean   median       uq      max neval
##     fun1() 17.87953 21.33259 12.35694 20.65903 20.54659 2.981957   100
##  fun1alt()  1.00000  1.00000  1.00000  1.00000  1.00000 1.000000   100
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
##        expr      min       lq     mean   median       uq   max neval
##     fun2(x) 8.207192 8.188059 6.626285 7.310173 7.668163 1.527   100
##  fun2alt(x) 1.000000 1.000000 1.000000 1.000000 1.000000 1.000   100
```

