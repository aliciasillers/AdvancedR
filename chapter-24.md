---
title: "chapter-24"
author: "Alicia Sillers"
date: "2023-09-27"
output: 
  html_document:
    keep_md: TRUE
---




```r
library(tidyverse)
```

```
## Warning: package 'tidyverse' was built under R version 4.2.3
```

```
## Warning: package 'ggplot2' was built under R version 4.2.3
```

```
## Warning: package 'tibble' was built under R version 4.2.3
```

```
## Warning: package 'dplyr' was built under R version 4.2.3
```

```
## Warning: package 'lubridate' was built under R version 4.2.3
```

```
## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
## ✔ dplyr     1.1.2     ✔ readr     2.1.4
## ✔ forcats   1.0.0     ✔ stringr   1.5.0
## ✔ ggplot2   3.4.2     ✔ tibble    3.2.1
## ✔ lubridate 1.9.2     ✔ tidyr     1.3.0
## ✔ purrr     1.0.1     
## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::filter() masks stats::filter()
## ✖ dplyr::lag()    masks stats::lag()
## ℹ Use the ]8;;http://conflicted.r-lib.org/conflicted package]8;; to force all conflicts to become errors
```

```r
library(bench)
```

```
## Warning: package 'bench' was built under R version 4.2.3
```

# 24.2 Notes

When tackling a bottleneck, you’re likely to come up with multiple approaches. Write a function for each approach, encapsulating all relevant behaviour. This makes it easier to check that each approach returns the correct result and to time how long it takes to run. 

# 24.3 Notes

Places to look for existing solutions: CRAN task views, reverse dependencies of Rcpp (as listed on its CRAN page)

# 24.4 Notes

Using functions tailored to specific purposes can improve speed.    

rowSums(), colSums(), rowMeans(), and colMeans() are faster than equivalent invocations that use apply() because they are vectorised     

vapply() is faster than sapply() because it pre-specifies the output type.    

If you want to see if a vector contains a single value, any(x == 10) is much faster than 10 %in% x because testing equality is simpler than testing set inclusion.   

apply() always turns its input into a matrix. Not only is this error prone (because a data frame is more general than a matrix), it is also slower.    

read.csv(): specify known column types with colClasses. (Also consider switching to readr::read_csv() or data.table::fread() which are considerably faster than read.csv().)   

unlist(x, use.names = FALSE) is much faster than unlist(x).

# 24.5 Notes

Vectorizing your code is not just about avoiding for loops, although that’s often a step. Vectorizing is about taking a whole-object approach to a problem, thinking about vectors, not scalars. It makes many problems simpler. Instead of having to think about the components of a vector, you only think about entire vectors. The loops in a vectorized function are written in C instead of R. Loops in C are much faster because they have much less overhead. Vectorization means finding the existing R function that is implemented in C and most closely applies to your problem. Vectorization has a downside: it is harder to predict how operations will scale.    

Vectorized subsetting can lead to big improvements in speed. If x is a vector, matrix or data frame then x[is.na(x)] <- 0 will replace all missing values with 0.    

If you’re extracting or replacing values in scattered locations in a matrix or data frame, subset with an integer matrix.

# 24.5 Exercises

1. The density functions, e.g., dnorm(), have a common interface. Which arguments are vectorised over? What does rnorm(10, mean = 10:1) do?

```r
?dnorm
```

```
## starting httpd help server ... done
```
Answer: They are vectorized over the argument, mean, and sd. rnorm(10, mean = 10:1) generates 10 numbers from sets of numbers with different means, decreasing from 10 to 1.   

2. Compare the speed of apply(x, 1, sum) with rowSums(x) for varying sizes of x.

```r
x <- data.frame(c1 = c(1,2), c2 = c(1,2))
bench::mark(
  rowSums(x),
  apply(x, 1, sum)
)
```

```
## # A tibble: 2 × 6
##   expression            min   median `itr/sec` mem_alloc `gc/sec`
##   <bch:expr>       <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
## 1 rowSums(x)           27µs   34.5µs    16238.    98.5KB     15.7
## 2 apply(x, 1, sum)   39.2µs   41.5µs    21405.        0B     12.9
```

```r
x <- data.frame(c1 = c(1,2,3,4,5,6,7,8), c2 = c(1,2,3,4,5,6,7,8), c3 = c(1,2,3,4,5,6,7,8), c4 = c(1,2,3,4,5,6,7,8), c5 = c(1,2,3,4,5,6,7,8), c6 = c(1,2,3,4,5,6,7,8), c7 = c(1,2,3,4,5,6,7,8), c8 = c(1,2,3,4,5,6,7,8))
bench::mark(
  rowSums(x),
  apply(x, 1, sum)
)
```

```
## # A tibble: 2 × 6
##   expression            min   median `itr/sec` mem_alloc `gc/sec`
##   <bch:expr>       <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
## 1 rowSums(x)         51.1µs   53.2µs    16947.      560B     14.6
## 2 apply(x, 1, sum)     70µs   73.6µs    11640.    1.09KB     12.4
```

```r
x <- as.data.frame(matrix (runif (n=1000, min=1, max=20), nrow=50))
bench::mark(
  rowSums(x),
  apply(x, 1, sum)
)
```

```
## # A tibble: 2 × 6
##   expression            min   median `itr/sec` mem_alloc `gc/sec`
##   <bch:expr>       <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
## 1 rowSums(x)          103µs    110µs     6842.    9.31KB     10.5
## 2 apply(x, 1, sum)    171µs    183µs     4867.   38.41KB     12.7
```


3. How can you use crossprod() to compute a weighted sum? How much faster is it than the naive sum(x * w)?

```r
n <- 1:10
x <- rnorm(n * 1e6)
    bench::mark(
      sum = sum(x * x),
      crossprod = crossprod(x, x)[[1]]
    )
```

```
## # A tibble: 2 × 6
##   expression      min   median `itr/sec` mem_alloc `gc/sec`
##   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
## 1 sum           300ns    300ns  2613696.        0B      0  
## 2 crossprod     600ns    700ns   824381.    2.22KB     82.4
```
Answer: crossprod() is about twice as fast as sum(x*w)

#24.6 Notes

A pernicious source of slow R code is growing an object with a loop. Whenever you use c(), append(), cbind(), rbind(), or paste() to create a bigger object, R must first allocate space for the new object and then copy the old object to its new home    

Modifying an object in a loop, e.g., x[i] <- y, can also create a copy, depending on the class of x. Section 2.5.1 discusses this issue in more depth and gives you some tools to determine when you’re making copies.

