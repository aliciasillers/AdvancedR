---
title: "chapter 10"
output: 
  html_document:
    keep_md: TRUE
date: "2023-02-05"
---




```r
library(rlang)
library(ggplot2)
library(scales)
library(e1071)
```

#10.2.6 Exercises

1. The definition of force() is simple:

```r
force
```

```
## function (x) 
## x
## <bytecode: 0x0000022c4084ac50>
## <environment: namespace:base>
```

```r
#> function (x) 
#> x
#> <bytecode: 0x7fe8519464b0>
#> <environment: namespace:base>
```
Why is it better to force(x) instead of just x?   
Answer: It is better to force x so that the value of x given to the function factory is evaluated and saved when the function factory is created and not just lazily evaluated when the manufactured function is read. This ensures that x is the consistently the correct value in the manufactured functions, even if it is bound to other values at other times.     

2. Base R contains two function factories, approxfun() and ecdf(). Read their documentation and experiment to figure out what the functions do and what they return.    
Answer: approxfun() returns a function performing linear or constant interpolation of the given data points. That function will return the interpolated values of the input x values. ecdf() returns a function that returns the percentiles of input x values. 

3. Create a function pick() that takes an index, i, as an argument and returns a function with an argument x that subsets x with i.

```r
#pick(1)(x)
# should be equivalent to
#x[[1]]

#lapply(mtcars, pick(5))
# should be equivalent to
#lapply(mtcars, function(x) x[[5]])
```
Answer:

```r
pick <- function(i){
  force(i)
  fun <- function(x) x[[i]]
}

lapply(mtcars, pick(5))
```

```
## $mpg
## [1] 18.7
## 
## $cyl
## [1] 8
## 
## $disp
## [1] 360
## 
## $hp
## [1] 175
## 
## $drat
## [1] 3.15
## 
## $wt
## [1] 3.44
## 
## $qsec
## [1] 17.02
## 
## $vs
## [1] 0
## 
## $am
## [1] 0
## 
## $gear
## [1] 3
## 
## $carb
## [1] 2
```

4. Create a function that creates functions that compute the ithcentral moment of a numeric vector. You can test it by running the following code:

```r
#m1 <- moment(1)
#m2 <- moment(2)

#x <- runif(100)
#stopifnot(all.equal(m1(x), 0))
#stopifnot(all.equal(m2(x), var(x) * 99 / 100))
```
Answer:

```r
moment2 <- function(i){
  force(i)
  function(v) moment(v, order=i, center=TRUE)
}

m1 <- moment2(1)
m2 <- moment2(2)

x <- runif(100)
stopifnot(all.equal(m1(x), 0))
stopifnot(all.equal(m2(x), var(x) * 99 / 100))

m1(x)
```

```
## [1] 6.661338e-18
```

```r
m2(x)
```

```
## [1] 0.07992705
```

5. What happens if you don’t use a closure? Make predictions, then verify with the code below.

```r
i <- 0
new_counter2 <- function() {
  i <<- i + 1
  i
}
```
Answer: Without a closure, this is just a single function and not a function factory. The function will still return how many times it has been called. However, since i is in the global environment, if it is bound to another value later, the function's count will be thrown off because it will reset to the new value + 1.

```r
new_counter2()
```

```
## [1] 1
```

```r
i <- 10
new_counter2()
```

```
## [1] 11
```

6. What happens if you use <- instead of <<-? Make predictions, then verify with the code below.

```r
new_counter3 <- function() {
  i <- 0
  function() {
    i <- i + 1
    i
  }
}
```
Answer: My prediction is that since <- always creates a binding in the current environment, using it instead of <<- to define i in the manufactured function will create a new object each time the function is called rather than rebinding the old object, so the counts will not be saved.

```r
counter3 <- new_counter3()
counter3()
```

```
## [1] 1
```

#10.3.4 Exercises

1. Compare and contrast ggplot2::label_bquote() with scales::number_format()    
Answer: label_bquote() creates labels for rows and columns using plotmath expressions. label_number() forces numerical labels to be in decimal format. 
