---
title: "chapter 9"
output:
  html_document:
    keep_md: TRUE
date: "2023-01-11"
---




```r
library(purrr)
```


#9.2.6 Exercises

1. Use as_mapper() to explore how purrr generates anonymous functions for the integer, character, and list helpers. What helper allows you to extract attributes? Read the documentation to find out.    
Answer:

```r
f1 <- as_mapper(c(1,2,3))
f1
```

```
## function (x, ...) 
## pluck_raw(x, list(1, 2, 3), .default = NULL)
## <environment: 0x0000020a435abee8>
```

```r
f2 <- as_mapper(c("a", "b", "c"))
f2
```

```
## function (x, ...) 
## pluck_raw(x, list("a", "b", "c"), .default = NULL)
## <environment: 0x0000020a43698138>
```

```r
f3 <- as_mapper(list("a", 2, c(3,4)))
f3
```

```
## function (x, ...) 
## pluck_raw(x, list("a", 2, c(3, 4)), .default = NULL)
## <environment: 0x0000020a4376a8d0>
```

```r
as_mapper(list(1, attr_getter("a")))
```

```
## function (x, ...) 
## pluck_raw(x, list(1, function (x) 
## attr(x, attr, exact = TRUE)), .default = NULL)
## <environment: 0x0000020a43846708>
```
attr_getter() allows you to extract attributes.   

2. map(1:3, ~ runif(2)) is a useful pattern for generating random numbers, but map(1:3, runif(2)) is not. Why not? Can you explain why it returns the result that it does?    
Answer: The first version works because runif() is written as a function to be evaluated with every iteration, whereas the second version passes runif(2) directly to map(), where it should be executed one time. In practice though, it does not seem to be executed at all. I don't know why.

```r
map(1:3, ~ runif(2))
```

```
## [[1]]
## [1] 0.8657751 0.8935995
## 
## [[2]]
## [1] 0.35149646 0.04155022
## 
## [[3]]
## [1] 0.3271014 0.4236221
```

```r
map(1:3, runif(2))
```

```
## [[1]]
## [1] 1
## 
## [[2]]
## [1] 2
## 
## [[3]]
## [1] 3
```

3. Use the appropriate map() function to:   
a) Compute the standard deviation of every column in a numeric data frame.    
Answer:

```r
map_dbl(iris[,1:3], sd)
```

```
## Sepal.Length  Sepal.Width Petal.Length 
##    0.8280661    0.4358663    1.7652982
```

b) Compute the standard deviation of every numeric column in a mixed data frame. (Hint: you’ll need to do it in two steps.)   
Answer:

```r
numcol <- map_lgl(ToothGrowth, is.numeric)
map_dbl(ToothGrowth[numcol], sd)
```

```
##       len      dose 
## 7.6493152 0.6288722
```

c) Compute the number of levels for every factor in a data frame.   
Answer:

```r
factcol <- map_lgl(PlantGrowth, is.factor)
map_int(PlantGrowth[factcol], nlevels)
```

```
## group 
##     3
```

4. The following code simulates the performance of a t-test for non-normal data. Extract the p-value from each test, then visualize.   

```r
trials <- map(1:100, ~ t.test(rpois(10, 10), rpois(7, 10)))
```
Answer:

```r
p <- map_dbl(trials, "p.value")
hist(p)
```

![](chapter-9_files/figure-html/unnamed-chunk-8-1.png)<!-- -->


5. The following code uses a map nested inside another map to apply a function to every element of a nested list. Why does it fail, and what do you need to do to make it work?

```r
x <- list(
  list(1, c(3, 9)),
  list(c(3, 6), 7, c(4, 7, 6))
)

triple <- function(x) x * 3
#map(x, map, .f = triple)
map(x, function(x) map(x, triple))
```

```
## [[1]]
## [[1]][[1]]
## [1] 3
## 
## [[1]][[2]]
## [1]  9 27
## 
## 
## [[2]]
## [[2]][[1]]
## [1]  9 18
## 
## [[2]][[2]]
## [1] 21
## 
## [[2]][[3]]
## [1] 12 21 18
```

```r
#> Error in .f(.x[[i]], ...): unused argument (function (.x, .f, ...)
#> {
#> .f <- as_mapper(.f, ...)
#> .Call(map_impl, environment(), ".x", ".f", "list")
#> })
```
Answer: passing it into map caused it only to be executed once, whereas putting it into a function causes it to apply to everything (old code commented out and replaced by corrected code)   

6. Use map() to fit linear models to the mtcars dataset using the formulas stored in this list:

```r
formulas <- list(
  mpg ~ disp,
  mpg ~ I(1 / disp),
  mpg ~ disp + wt,
  mpg ~ I(1 / disp) + wt
)
```
Answer:

```r
map(formulas, lm, data = mtcars)
```

```
## [[1]]
## 
## Call:
## .f(formula = .x[[i]], data = ..1)
## 
## Coefficients:
## (Intercept)         disp  
##    29.59985     -0.04122  
## 
## 
## [[2]]
## 
## Call:
## .f(formula = .x[[i]], data = ..1)
## 
## Coefficients:
## (Intercept)    I(1/disp)  
##       10.75      1557.67  
## 
## 
## [[3]]
## 
## Call:
## .f(formula = .x[[i]], data = ..1)
## 
## Coefficients:
## (Intercept)         disp           wt  
##    34.96055     -0.01772     -3.35083  
## 
## 
## [[4]]
## 
## Call:
## .f(formula = .x[[i]], data = ..1)
## 
## Coefficients:
## (Intercept)    I(1/disp)           wt  
##      19.024     1142.560       -1.798
```

7. Fit the model mpg ~ disp to each of the bootstrap replicates of mtcars in the list below, then extract the R^2 of the model fit (Hint: you can compute the R^2 with summary().)

```r
bootstrap <- function(df) {
  df[sample(nrow(df), replace = TRUE), , drop = FALSE]
}

bootstraps <- map(1:10, ~ bootstrap(mtcars))
```
Answer:

```r
fitmodel <- map(bootstraps, ~ lm(mpg ~ disp, data = mtcars))
modsum <- map(fitmodel, summary)
map(modsum, "r.squared")
```

```
## [[1]]
## [1] 0.7183433
## 
## [[2]]
## [1] 0.7183433
## 
## [[3]]
## [1] 0.7183433
## 
## [[4]]
## [1] 0.7183433
## 
## [[5]]
## [1] 0.7183433
## 
## [[6]]
## [1] 0.7183433
## 
## [[7]]
## [1] 0.7183433
## 
## [[8]]
## [1] 0.7183433
## 
## [[9]]
## [1] 0.7183433
## 
## [[10]]
## [1] 0.7183433
```

#9.4.6 Exercises

1. Explain the results of modify(mtcars, 1).

```r
modify(mtcars,1)
```

```
##    mpg cyl disp  hp drat   wt  qsec vs am gear carb
## 1   21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 2   21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 3   21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 4   21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 5   21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 6   21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 7   21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 8   21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 9   21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 10  21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 11  21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 12  21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 13  21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 14  21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 15  21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 16  21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 17  21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 18  21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 19  21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 20  21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 21  21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 22  21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 23  21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 24  21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 25  21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 26  21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 27  21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 28  21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 29  21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 30  21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 31  21   6  160 110  3.9 2.62 16.46  0  1    4    4
## 32  21   6  160 110  3.9 2.62 16.46  0  1    4    4
```
Answer: It replaces all the rows of data with the data from the first row. Output must be same type/size of input.    

2. Rewrite the following code to use iwalk() instead of walk2(). What are the advantages and disadvantages?

```r
temp <- tempfile()
dir.create(temp)

cyls <- split(mtcars, mtcars$cyl)
paths <- file.path(temp, paste0("cyl-", names(cyls), ".csv"))
walk2(cyls, paths, write.csv)

dir(temp)
```

```
## [1] "cyl-4.csv" "cyl-6.csv" "cyl-8.csv"
```
Answer:

```r
temp2 <- tempfile()
dir.create(temp2)

cyls <- split(mtcars, mtcars$cyl)
iwalk(cyls, ~ write.csv(.x, file.path(temp2, paste0("cyl-", .y, ".csv"))))

dir(temp2)
```

```
## [1] "cyl-4.csv" "cyl-6.csv" "cyl-8.csv"
```
A bit more concise with iwalk but also maybe a bit harder to understand.    

3. Explain how the following code transforms a data frame using functions stored in a list.

```r
trans <- list(
  disp = function(x) x * 0.0163871,
  am = function(x) factor(x, labels = c("auto", "manual"))
)

nm <- names(trans)
mtcars[nm] <- map2(trans, mtcars[nm], function(f, var) f(var))
```
Compare and contrast the map2() approach to this map() approach:

```r
mtcars[nm] <- map(nm, ~ trans[[.x]](mtcars[[.x]]))
```
Answer: The first block of code alters two columns of data to mtcars. map2() is used to read both functions from trans one at a time and apply each one to the column of the same name. The map() approach also seems effective but it is much less simple to understand what is happening.   

4. What does write.csv() return, i.e. what happens if you use it with map2() instead of walk2()?

```r
map2(cyls, paths, write.csv)
```

```
## $`4`
## NULL
## 
## $`6`
## NULL
## 
## $`8`
## NULL
```
Answer: it returns NULL

#9.6.3 Exercises

1. Why isn’t is.na() a predicate function? What base R function is closest to being a predicate version of is.na()?   
Answer: is.na() is not a predicate function because it returns a TRUE or FALSE for each value, whereas a predicate function only returns a single TRUE or FALSE. anyNA() would probably be the closest to being a predicate version of is.na().

2. simple_reduce() has a problem when x is length 0 or length 1. Describe the source of the problem and how you might go about fixing it.

```r
simple_reduce <- function(x, f) {
  out <- x[[1]]
  for (i in seq(2, length(x))) {
    out <- f(out, x[[i]])
  }
  out
}

x1 <- c(1)
x2 <- c(1,2,3,4)

simple_reduce(x2, mean)
```

```
## [1] 1
```
Answer: The source of the problem comes from seq(2, length(x)) because it will try to go the the second position in x even when there is not one. This could be fixed by using a map variant that only applies the function when length of x in greater than 1. 

```r
simple_reduce2 <- function(x, f) {
  out <- x[[1]]
  fun <- function(a,b){a <- f(a,b)}
  modify_if(out, length(x)>1, fun(out, x))
  out
}

#simple_reduce2(x2, mean)
```


3. Implement the span() function from Haskell: given a list x and a predicate function f, span(x, f) returns the location of the longest sequential run of elements where the predicate is true. (Hint: you might find rle() helpful.)    
Answer:

```r
l <- c(1,2,2,3,4,5,5,5,5,6,7)
lv <- as.vector(l, mode="any")
span <- function(x,f){
  a <- c()
  map_if(x, f, a <- c(a,x))
  rleout <- rle(a)
  rleout
}
span(lv, is.numeric)
```

```
## Run Length Encoding
##   lengths: int [1:7] 1 2 1 1 4 1 1
##   values : num [1:7] 1 2 3 4 5 6 7
```

4. Implement arg_max(). It should take a function and a vector of inputs, and return the elements of the input where the function returns the highest value. For example, arg_max(-10:5, function(x) x ^ 2) should return -10. arg_max(-5:5, function(x) x ^ 2) should return c(-5, 5). Also implement the matching arg_min() function.   
Answer:

```r
arg_max <- function(x, f){
  out <- c()
  for (i in 1:length(x)){
    out <- c(out, f(x[[i]]))
  }
  maxn <- max(out)
  ind <- which(out %in% maxn)
  x[ind]
}

arg_max(-10:5, function(x) x^2) 
```

```
## [1] -10
```

```r
arg_min <- function(x, f){
  out <- c()
  for (i in 1:length(x)){
    out <- c(out, f(x[[i]]))
  }
  minn <- min(out)
  ind <- which(out %in% minn)
  x[ind]
}

arg_min(-10:5, function(x) x^2) 
```

```
## [1] 0
```

5. The function below scales a vector so it falls in the range [0, 1]. How would you apply it to every column of a data frame? How would you apply it to every numeric column in a data frame?

```r
scale01 <- function(x) {
  rng <- range(x, na.rm = TRUE)
  (x - rng[1]) / (rng[2] - rng[1])
}
```
Answer: 

```r
map_if(mtcars, is.numeric, scale01)
```

```
## $mpg
##  [1] 0.4510638 0.4510638 0.5276596 0.4680851 0.3531915 0.3276596 0.1659574
##  [8] 0.5957447 0.5276596 0.3744681 0.3148936 0.2553191 0.2936170 0.2042553
## [15] 0.0000000 0.0000000 0.1829787 0.9361702 0.8510638 1.0000000 0.4723404
## [22] 0.2170213 0.2042553 0.1234043 0.3744681 0.7191489 0.6638298 0.8510638
## [29] 0.2297872 0.3957447 0.1957447 0.4680851
## 
## $cyl
##  [1] 0.5 0.5 0.0 0.5 1.0 0.5 1.0 0.0 0.0 0.5 0.5 1.0 1.0 1.0 1.0 1.0 1.0 0.0 0.0
## [20] 0.0 0.0 1.0 1.0 1.0 1.0 0.0 0.0 0.0 1.0 0.5 1.0 0.0
## 
## $disp
##  [1] 0.22175106 0.22175106 0.09204290 0.46620105 0.72062859 0.38388626
##  [7] 0.72062859 0.18857570 0.17385882 0.24070841 0.24070841 0.51060115
## [13] 0.51060115 0.51060115 1.00000000 0.97006735 0.92017960 0.01895735
## [19] 0.01147418 0.00000000 0.12222499 0.61586431 0.58094288 0.69568471
## [25] 0.82040409 0.01970566 0.12272387 0.05986530 0.69817910 0.18433525
## [31] 0.57345972 0.12446994
## 
## $hp
##  [1] 0.20494700 0.20494700 0.14487633 0.20494700 0.43462898 0.18727915
##  [7] 0.68197880 0.03533569 0.15194346 0.25088339 0.25088339 0.45229682
## [13] 0.45229682 0.45229682 0.54063604 0.57597173 0.62897527 0.04946996
## [19] 0.00000000 0.04593640 0.15901060 0.34628975 0.34628975 0.68197880
## [25] 0.43462898 0.04946996 0.13780919 0.21554770 0.74911661 0.43462898
## [31] 1.00000000 0.20141343
## 
## $drat
##  [1] 0.52534562 0.52534562 0.50230415 0.14746544 0.17972350 0.00000000
##  [7] 0.20737327 0.42857143 0.53456221 0.53456221 0.53456221 0.14285714
## [13] 0.14285714 0.14285714 0.07834101 0.11059908 0.21658986 0.60829493
## [19] 1.00000000 0.67281106 0.43317972 0.00000000 0.17972350 0.44700461
## [25] 0.14746544 0.60829493 0.76958525 0.46543779 0.67281106 0.39631336
## [31] 0.35944700 0.62211982
## 
## $wt
##  [1] 0.28304781 0.34824853 0.20634109 0.43518282 0.49271286 0.49782664
##  [7] 0.52595244 0.42879059 0.41856303 0.49271286 0.49271286 0.65379698
## [13] 0.56686269 0.57964715 0.95551010 1.00000000 0.97980056 0.17565840
## [19] 0.02608029 0.08233188 0.24341601 0.51316799 0.49143442 0.59498849
## [25] 0.59626694 0.10790079 0.16031705 0.00000000 0.42367681 0.32140118
## [31] 0.52595244 0.32395807
## 
## $qsec
##  [1] 0.23333333 0.30000000 0.48928571 0.58809524 0.30000000 0.68095238
##  [7] 0.15952381 0.65476190 1.00000000 0.45238095 0.52380952 0.34523810
## [13] 0.36904762 0.41666667 0.41428571 0.39523810 0.34761905 0.59166667
## [19] 0.47857143 0.64285714 0.65595238 0.28214286 0.33333333 0.10833333
## [25] 0.30357143 0.52380952 0.26190476 0.28571429 0.00000000 0.11904762
## [31] 0.01190476 0.48809524
## 
## $vs
##  [1] 0 0 1 1 0 1 0 1 1 1 1 0 0 0 0 0 0 1 1 1 1 0 0 0 0 1 0 1 0 0 0 1
## 
## $am
##  [1] manual manual manual auto   auto   auto   auto   auto   auto   auto  
## [11] auto   auto   auto   auto   auto   auto   auto   manual manual manual
## [21] auto   auto   auto   auto   auto   manual manual manual manual manual
## [31] manual manual
## Levels: auto manual
## 
## $gear
##  [1] 0.5 0.5 0.5 0.0 0.0 0.0 0.0 0.5 0.5 0.5 0.5 0.0 0.0 0.0 0.0 0.0 0.0 0.5 0.5
## [20] 0.5 0.0 0.0 0.0 0.0 0.0 0.5 1.0 1.0 1.0 1.0 1.0 0.5
## 
## $carb
##  [1] 0.4285714 0.4285714 0.0000000 0.0000000 0.1428571 0.0000000 0.4285714
##  [8] 0.1428571 0.1428571 0.4285714 0.4285714 0.2857143 0.2857143 0.2857143
## [15] 0.4285714 0.4285714 0.4285714 0.0000000 0.1428571 0.0000000 0.0000000
## [22] 0.1428571 0.1428571 0.4285714 0.1428571 0.0000000 0.1428571 0.1428571
## [29] 0.4285714 0.7142857 1.0000000 0.1428571
```

#9.7.3 Exercises

1. How does apply() arrange the output? Read the documentation and perform some experiments.    
Answer: returns a matrix or array 

2. What do eapply() and rapply() do? Does purrr have equivalents?   
Answer: eapply() applies a function to named values from an environment and returns the results as a list. rapply() applies a function to elements of x and returns a list of the results with flexibility in how they are stored.

3. Challenge: read about the fixed point algorithm. Complete the exercises using R.   
Answer: can't pull up the fixed point algorithm page
