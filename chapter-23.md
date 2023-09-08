---
title: "chapter 23"
author: "Alicia Sillers"
date: "2023-09-08"
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
library(profvis)
```

```
## Warning: package 'profvis' was built under R version 4.2.3
```

```r
library(bench)
```

```
## Warning: package 'bench' was built under R version 4.2.3
```

# 23.2 Notes

A sampling profiler stops the execution of code every few milliseconds and records the call stack


```r
f <- function() {
  pause(0.1)
  g()
  h()
}
g <- function() {
  pause(0.1)
  h()
}
h <- function() {
  pause(0.1)
}
```

There are two ways to use profvis:

1. From the Profile menu in RStudio.

2. With profvis::profvis(). I recommend storing your code in a separate file and source()ing it in; this will ensure you get the best connection between profiling data and source code.   


```r
#source("profiling-example.R")
profvis(f())
```

```{=html}
<div class="profvis html-widget html-fill-item-overflow-hidden html-fill-item" id="htmlwidget-40de24df616f668ed0ce" style="width:100%;height:600px;"></div>
<script type="application/json" data-for="htmlwidget-40de24df616f668ed0ce">{"x":{"message":{"prof":{"time":[1,1,2,2,3,3,4,4,5,5,6,6,7,7,7,8,8,8,9,9,9,10,10,10,11,11,11,12,12,12,13,13,13,13,14,14,14,14,15,15,15,15,16,16,16,16,17,17,17,17,18,18,18,18,19,19,19,19,20,20,20,21,21,21,22,22,22,23,23,23,24,24,24,25,25,25],"depth":[2,1,2,1,2,1,2,1,2,1,2,1,3,2,1,3,2,1,3,2,1,3,2,1,3,2,1,3,2,1,4,3,2,1,4,3,2,1,4,3,2,1,4,3,2,1,4,3,2,1,4,3,2,1,4,3,2,1,3,2,1,3,2,1,3,2,1,3,2,1,3,2,1,3,2,1],"label":["pause","f","pause","f","pause","f","pause","f","pause","f","pause","f","pause","g","f","pause","g","f","pause","g","f","pause","g","f","pause","g","f","pause","g","f","pause","h","g","f","pause","h","g","f","pause","h","g","f","pause","h","g","f","pause","h","g","f","pause","h","g","f","pause","h","g","f","pause","h","f","pause","h","f","pause","h","f","pause","h","f","pause","h","f","pause","h","f"],"filenum":[null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null],"linenum":[null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null],"memalloc":[12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.21396636962891,12.23390960693359,12.23390960693359,12.23390960693359,12.23390960693359,12.23390960693359,12.23390960693359,12.23390960693359,12.23390960693359,12.23390960693359,12.23390960693359,12.23390960693359,12.23390960693359,12.23390960693359,12.23390960693359,12.23390960693359,12.23390960693359,12.23390960693359,12.23390960693359],"meminc":[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0.0199432373046875,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"filename":[null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null]},"interval":10,"files":[],"prof_output":"C:\\Users\\alici\\AppData\\Local\\Temp\\RtmpWozwGL\\file26a873a263e9.prof","highlight":{"output":["^output\\$"],"gc":["^<GC>$"],"stacktrace":["^\\.\\.stacktraceo(n|ff)\\.\\.$"]},"split":"h"}},"evals":[],"jsHooks":[]}</script>
```

If <GC> is taking a lot of time, it’s usually an indication that you’re creating many short-lived objects.

# 23.2 Exercises

1. Profile the following function with torture = TRUE. What is surprising? Read the source code of rm() to figure out what’s going on.

```r
f <- function(n = 1e5) {
  x <- rep(1, n)
  rm(x)
}
```


```r
#profvis(f(), torture = TRUE)
```
Answer: It takes a very long time to run due to garbage collection triggered by the rm() function

# 23.3 Notes

A microbenchmark is a measurement of the performance of a very small piece of code. They are useful for comparing small snippets of code for specific tasks.    

The bench package uses a high precision timer, making it possible to compare operations that only take a tiny amount of time


```r
x <- runif(100)
(lb <- bench::mark(
  sqrt(x),
  x ^ 0.5
))
```

```
## # A tibble: 2 × 6
##   expression      min   median `itr/sec` mem_alloc `gc/sec`
##   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
## 1 sqrt(x)       400ns    500ns  1121957.      848B        0
## 2 x^0.5         3.2µs    3.3µs   245589.      848B        0
```

```r
#> # A tibble: 2 x 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 sqrt(x)       865ns   1.05µs   679021.      848B        0
#> 2 x^0.5        3.78µs   4.17µs   203205.      848B        0
```

By default, bench::mark() runs each expression at least once (min_iterations = 1), and at most enough times to take 0.5 s (min_time = 0.5). It checks that each run returns the same value which is typically what you want microbenchmarking; if you want to compare the speed of expressions that return different values, set check = FALSE. It returns the results as a tibble, with one row for each input expression, and the following columns: min, mean, median, max, and itr/sec summarise the time taken by the expression. 


```r
plot(lb)
```

![](chapter-23_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

```r
#> Loading required namespace: tidyr
```
mem_alloc tells you the amount of memory allocated by the first run, and n_gc() tells you the total number of garbage collections over all runs. These are useful for assessing the memory usage of the expression.   

n_itr and total_time tells you how many times the expression was evaluated and how long that took in total. n_itr will always be greater than the min_iteration parameter, and total_time will always be greater than the min_time parameter.    

result, memory, time, and gc are list-columns that store the raw underlying data    


```r
lb[c("expression", "min", "median", "itr/sec", "n_gc")]
```

```
## # A tibble: 2 × 4
##   expression      min   median `itr/sec`
##   <bch:expr> <bch:tm> <bch:tm>     <dbl>
## 1 sqrt(x)       400ns    500ns  1121957.
## 2 x^0.5         3.2µs    3.3µs   245589.
```

```r
#> # A tibble: 2 x 4
#>   expression      min   median `itr/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl>
#> 1 sqrt(x)       865ns   1.05µs   679021.
#> 2 x^0.5        3.78µs   4.17µs   203205.
```

# 23.3 Exercises

1. Instead of using bench::mark(), you could use the built-in function system.time(). But system.time() is much less precise, so you’ll need to repeat each operation many times with a loop, and then divide to find the average time of each operation, as in the code below.

```r
n <- 1e6
system.time(for (i in 1:n) sqrt(x)) / n
```

```
##    user  system elapsed 
## 3.1e-07 6.0e-08 7.8e-07
```

```r
system.time(for (i in 1:n) x ^ 0.5) / n
```

```
##     user   system  elapsed 
## 2.19e-06 0.00e+00 3.70e-06
```
How do the estimates from system.time() compare to those from bench::mark()? Why are they different? 

```r
bm <- bench::mark(
  sqrt(x), 
  x ^ 0.5,
  iterations = n
)

summary(bm)
```

```
## # A tibble: 2 × 6
##   expression      min   median `itr/sec` mem_alloc `gc/sec`
##   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
## 1 sqrt(x)       400ns    500ns  1777224.      848B    37.3 
## 2 x^0.5         3.2µs    3.3µs   270550.      848B     7.31
```

Answer: The estimates seem fairly similar but bench::mark() shows slightly more time and is likely more accurate. 

2. Here are two other ways to compute the square root of a vector. Which do you think will be fastest? Which will be slowest? Use microbenchmarking to test your answers.

```r
x ^ (1 / 2)
```

```
##   [1] 0.6330939 0.6841759 0.6514051 0.9173671 0.6529203 0.6458101 0.4262499
##   [8] 0.5466660 0.9304812 0.7931319 0.4667304 0.9248078 0.5402295 0.4845056
##  [15] 0.3222795 0.8603851 0.9809463 0.4580857 0.8628819 0.9522074 0.4238919
##  [22] 0.7003353 0.3505987 0.9076652 0.8004939 0.9108112 0.4642704 0.4469432
##  [29] 0.6913394 0.7187836 0.1968360 0.5520977 0.6401418 0.5533677 0.7385895
##  [36] 0.8275366 0.6794045 0.0818988 0.7919822 0.8621291 0.7956597 0.4801029
##  [43] 0.9724597 0.6414684 0.8907956 0.7701590 0.6123027 0.8920488 0.8767728
##  [50] 0.5025477 0.3359136 0.3222450 0.5569836 0.2620650 0.7622921 0.3561705
##  [57] 0.8006015 0.2411205 0.7533490 0.8959298 0.9793712 0.7372915 0.2996736
##  [64] 0.4464465 0.9990532 0.8607937 0.8188232 0.8199310 0.8914353 0.6246701
##  [71] 0.5232043 0.8276059 0.9608024 0.8697883 0.7160723 0.2478287 0.9921029
##  [78] 0.8989842 0.9875969 0.5515156 0.3181277 0.9649773 0.5551811 0.5389485
##  [85] 0.9427362 0.8143339 0.6277501 0.4772217 0.4552120 0.1870186 0.9076631
##  [92] 0.6837952 0.9700832 0.5000996 0.3271646 0.1422122 0.6323699 0.9965869
##  [99] 0.8174670 0.5048674
```

```r
exp(log(x) / 2)
```

```
##   [1] 0.6330939 0.6841759 0.6514051 0.9173671 0.6529203 0.6458101 0.4262499
##   [8] 0.5466660 0.9304812 0.7931319 0.4667304 0.9248078 0.5402295 0.4845056
##  [15] 0.3222795 0.8603851 0.9809463 0.4580857 0.8628819 0.9522074 0.4238919
##  [22] 0.7003353 0.3505987 0.9076652 0.8004939 0.9108112 0.4642704 0.4469432
##  [29] 0.6913394 0.7187836 0.1968360 0.5520977 0.6401418 0.5533677 0.7385895
##  [36] 0.8275366 0.6794045 0.0818988 0.7919822 0.8621291 0.7956597 0.4801029
##  [43] 0.9724597 0.6414684 0.8907956 0.7701590 0.6123027 0.8920488 0.8767728
##  [50] 0.5025477 0.3359136 0.3222450 0.5569836 0.2620650 0.7622921 0.3561705
##  [57] 0.8006015 0.2411205 0.7533490 0.8959298 0.9793712 0.7372915 0.2996736
##  [64] 0.4464465 0.9990532 0.8607937 0.8188232 0.8199310 0.8914353 0.6246701
##  [71] 0.5232043 0.8276059 0.9608024 0.8697883 0.7160723 0.2478287 0.9921029
##  [78] 0.8989842 0.9875969 0.5515156 0.3181277 0.9649773 0.5551811 0.5389485
##  [85] 0.9427362 0.8143339 0.6277501 0.4772217 0.4552120 0.1870186 0.9076631
##  [92] 0.6837952 0.9700832 0.5000996 0.3271646 0.1422122 0.6323699 0.9965869
##  [99] 0.8174670 0.5048674
```
I think the first one will be fastest and the second will be slowest

```r
x <- runif(100)

bm <- bench::mark(
  sqrt(x),
  x^0.5,
  x^(1 / 2),
  exp(log(x) / 2)
)

bm[order(bm$median), ]
```

```
## # A tibble: 4 × 6
##   expression         min   median `itr/sec` mem_alloc `gc/sec`
##   <bch:expr>    <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
## 1 sqrt(x)          400ns    500ns  1874554.      848B      0  
## 2 x^0.5            3.2µs    3.3µs   261531.      848B      0  
## 3 x^(1/2)          3.4µs    3.6µs   260055.      848B     26.0
## 4 exp(log(x)/2)    9.3µs    9.5µs    99445.      848B      0
```

```r
#> # A tibble: 4 x 6
#>   expression         min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr>    <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 sqrt(x)          908ns    1.1µs   656785.      848B      0  
#> 2 exp(log(x)/2)   2.97µs   3.22µs   295051.      848B     29.5
#> 3 x^0.5           3.94µs   4.23µs   213837.      848B      0  
#> 4 x^(1/2)         4.08µs   4.45µs   204753.      848B      0
```
sqrt() is fastest and exp(log(x)/2) is slowest
