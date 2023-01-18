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
## <environment: 0x0000027aa5eabd18>
```

```r
f2 <- as_mapper(c("a", "b", "c"))
f2
```

```
## function (x, ...) 
## pluck_raw(x, list("a", "b", "c"), .default = NULL)
## <environment: 0x0000027aa9923dc0>
```

```r
f3 <- as_mapper(list("a", 2, c(3,4)))
f3
```

```
## function (x, ...) 
## pluck_raw(x, list("a", 2, c(3, 4)), .default = NULL)
## <environment: 0x0000027aa99d02c8>
```

```r
as_mapper(list(1, attr_getter("a")))
```

```
## function (x, ...) 
## pluck_raw(x, list(1, function (x) 
## attr(x, attr, exact = TRUE)), .default = NULL)
## <environment: 0x0000027aa9a91130>
```
attr_getter() allows you to extract attributes.   

2. map(1:3, ~ runif(2)) is a useful pattern for generating random numbers, but map(1:3, runif(2)) is not. Why not? Can you explain why it returns the result that it does?    
Answer: The first version works because runif() is written as a function to be evaluated with every iteration, whereas the second version passes runif(2) directly to map(), where it should be executed one time. In practice though, it does not seem to be executed at all. I don't know why.

```r
map(1:3, ~ runif(2))
```

```
## [[1]]
## [1] 0.4228497 0.3210340
## 
## [[2]]
## [1] 0.4284202 0.1402063
## 
## [[3]]
## [1] 0.9673101 0.5003308
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
