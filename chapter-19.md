---
title: "chapter-19"
author: "Alicia Sillers"
date: "2023-07-25"
output: 
  html_document:
    keep_md: TRUE
---





```r
library(rlang)
```

```
## Warning: package 'rlang' was built under R version 4.2.3
```

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
## ✖ purrr::%@%()         masks rlang::%@%()
## ✖ dplyr::filter()      masks stats::filter()
## ✖ purrr::flatten()     masks rlang::flatten()
## ✖ purrr::flatten_chr() masks rlang::flatten_chr()
## ✖ purrr::flatten_dbl() masks rlang::flatten_dbl()
## ✖ purrr::flatten_int() masks rlang::flatten_int()
## ✖ purrr::flatten_lgl() masks rlang::flatten_lgl()
## ✖ purrr::flatten_raw() masks rlang::flatten_raw()
## ✖ purrr::invoke()      masks rlang::invoke()
## ✖ dplyr::lag()         masks stats::lag()
## ✖ purrr::splice()      masks rlang::splice()
## ℹ Use the ]8;;http://conflicted.r-lib.org/conflicted package]8;; to force all conflicts to become errors
```

#19.4 Exercises

1. Given the following components:

```r
xy <- expr(x + y)
xz <- expr(x + z)
yz <- expr(y + z)
abc <- exprs(a, b, c)
```
Use quasiquotation to construct the following calls:

```r
#(x + y) / (y + z)
#-(x + z) ^ (y + z)
#(x + y) + (y + z) - (x + y)
#atan2(x + y, y + z)
#sum(x + y, x + y, y + z)
#sum(a, b, c)
#mean(c(a, b, c), na.rm = TRUE)
#foo(a = x + y, b = y + z)
```
Answer:

```r
expr(!!xy / !!yz)
```

```
## (x + y)/(y + z)
```

```r
expr(-(!!xz)^(!!yz))  
```

```
## -(x + z)^(y + z)
```

```r
expr(((!!xy)) + !!yz-!!xy)
```

```
## (x + y) + (y + z) - (x + y)
```

```r
expr(atan2(!!xy, !!yz))
```

```
## atan2(x + y, y + z)
```

```r
expr(sum(!!xy, !!xy, !!yz))
```

```
## sum(x + y, x + y, y + z)
```

```r
expr(sum(!!!abc))
```

```
## sum(a, b, c)
```

```r
expr(mean(c(!!!abc), na.rm = TRUE))
```

```
## mean(c(a, b, c), na.rm = TRUE)
```

```r
expr(foo(a = !!xy, b = !!yz))
```

```
## foo(a = x + y, b = y + z)
```


2. The following two calls print the same, but are actually different:

```r
(a <- expr(mean(1:10)))
```

```
## mean(1:10)
```

```r
#> mean(1:10)
(b <- expr(mean(!!(1:10))))
```

```
## mean(1:10)
```

```r
#> mean(1:10)
identical(a, b)
```

```
## [1] FALSE
```

```r
#> [1] FALSE
```
What’s the difference? Which one is more natural?       
In b, (1:10) is unquoted, so this part is evaluated and inlined while mean is not. In a, the whole expression is quoted. 

#19.6 Exercises

1. One way to implement exec() is shown below. Describe how it works. What are the key ideas?

```r
exec <- function(f, ..., .env = caller_env()) {
  args <- list2(...)
  do.call(f, args, envir = .env)
}
```
Answer: exec takes any arguments from ... and uses them as arguments for the given function, which it runs in the given environment. args is instantiated as a list of the arguments, with quasiquotation supported. do.call executes the function.    

2. Carefully read the source code for interaction(), expand.grid(), and par(). Compare and contrast the techniques they use for switching between dots and list behaviour.
Answer: interaction computes a factor which represents the interaction of the given factors. The results are unordered. expand.grid creates a data frame from all combinations of the supplied vectors or factors. par can be used to set or query graphical parameters.   

3. Explain the problem with this definition of set_attr()

```r
set_attr <- function(x, ...) {
  attr <- rlang::list2(...)
  attributes(x) <- attr
  x
}
#set_attr(1:10, x = 10)
#> Error in attributes(x) <- attr: attributes must be named
```
Answer: ... is supposed to take any argument, which set_attr will read as attributes of the first argument, x. In the example, x = 10 is given as an attribute but I think it is interpreted as the object instead because of its name. 

#19.7 Exercises

1. In the linear-model example, we could replace the expr() in reduce(summands, ~ expr(!!.x + !!.y)) with call2(): reduce(summands, call2, "+"). Compare and contrast the two approaches. Which do you think is easier to read?
Answer: with call2, "you pass the call elements (the function to call and the arguments to call it with) separately."  call2 reads the passed arguments as ... unless named with existing argument name.   

2. Re-implement the Box-Cox transform defined below using unquoting and new_function():

```r
bc <- function(lambda) {
  if (lambda == 0) {
    function(x) log(x)
  } else {
    function(x) (x ^ lambda - 1) / lambda
  }
}
```
Answer:

```r
bc2 <- function(lambda) {
  lambda <- enexpr(lambda)
  if (!!lambda == 0) {
    new_function(exprs(x = ), expr(log(x)))
  } else {
    new_function(exprs(x = ), expr((x ^ (!!lambda) - 1) / !!lambda))
  }
}
```

3. Re-implement the simple compose() defined below using quasiquotation and new_function():

```r
compose <- function(f, g) {
  function(...) f(g(...))
}
```
Answer:

```r
compose2 <- function(f, g) {
  f <- enexpr(f)
  g <- enexpr(g)
  new_function(exprs(... = ), expr((!!f)((!!g)(...))))
}
```

