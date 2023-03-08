---
title: "chapter-13"
output: 
  html_document:
    keep_md: TRUE
date: "2023-02-27"
---



```r
library(sloop)
```


#13.2 Notes

an S3 object has to have a class attribute

```r
f <- factor(c("a", "b", "c"))

typeof(f)
```

```
## [1] "integer"
```

```r
attributes(f)
```

```
## $levels
## [1] "a" "b" "c"
## 
## $class
## [1] "factor"
```

unclass() strips the class attribute

```r
unclass(f)
```

```
## [1] 1 2 3
## attr(,"levels")
## [1] "a" "b" "c"
```

sloop::ftype() can tell you if a function is generic or not. A generic function defines an interface, which uses a different implementation depending on the class of an argument. The implementation for a specific class is called a method, and the generic finds that method by performing method dispatch.

```r
s3_dispatch(print(f))
```

```
## => print.factor
##  * print.default
```

Generally, you can identify a method by the presence of . in the function name, but there are a number of important functions in base R that were written before S3, and hence use . to join words. If you’re unsure, check with sloop::ftype()

```r
ftype(t.test)
```

```
## [1] "S3"      "generic"
```

```r
ftype(t.data.frame)
```

```
## [1] "S3"     "method"
```

#13.2.1 Exercises

1. Describe the difference between t.test() and t.data.frame(). When is each function called?   
Answer: t.test() is a generic function, while t.data.frame() is not. t.test() is called by the command t.test() while t.data.frame() is called by the command t().    

2. Make a list of commonly used base R functions that contain . in their name but are not S3 methods.    
Answer: as.numeric, as.character, as.logical, is.numeric, is.na, read.csv, write.csv, data.frame, Sys.date, file.path

3. What does the as.data.frame.data.frame() method do? Why is it confusing? How could you avoid this confusion in your own code?

```r
as.data.frame.data.frame
```

```
## function (x, row.names = NULL, ...) 
## {
##     cl <- oldClass(x)
##     i <- match("data.frame", cl)
##     if (i > 1L) 
##         class(x) <- cl[-(1L:(i - 1L))]
##     if (!is.null(row.names)) {
##         nr <- .row_names_info(x, 2L)
##         if (length(row.names) == nr) 
##             attr(x, "row.names") <- row.names
##         else stop(sprintf(ngettext(nr, "invalid 'row.names', length %d for a data frame with %d row", 
##             "invalid 'row.names', length %d for a data frame with %d rows"), 
##             length(row.names), nr), domain = NA)
##     }
##     x
## }
## <bytecode: 0x00000259a81be8f0>
## <environment: namespace:base>
```
Removes class attributes other than data.frame

4. Describe the difference in behaviour in these two calls.

```r
set.seed(1014)
some_days <- as.Date("2017-01-31") + sample(10, 5)

mean(some_days)
```

```
## [1] "2017-02-06"
```

```r
#> [1] "2017-02-06"
mean(unclass(some_days))
```

```
## [1] 17203.4
```

```r
#> [1] 17203
```
Answer: I get an error when I try to run this code myself, but based on the commented output, it seems that mean(some_days) with the class attribute calculates mean between dates, while the unclassed version converts the dates to numeric values and calculates the mean of those values.   

5. What class of object does the following code return? What base type is it built on? What attributes does it use?

```r
x <- ecdf(rpois(100, 10))
x
```

```
## Empirical CDF 
## Call: ecdf(rpois(100, 10))
##  x[1:18] =      2,      3,      4,  ...,     18,     19
```

```r
typeof(x)
```

```
## [1] "closure"
```

```r
attributes(x)
```

```
## $class
## [1] "ecdf"     "stepfun"  "function"
## 
## $call
## ecdf(rpois(100, 10))
```
Answer: This has class attributes and a call attributes. The class is stepfun and the base type is closure.   

6.What class of object does the following code return? What base type is it built on? What attributes does it use?

```r
x <- table(rpois(100, 5))
x
```

```
## 
##  1  2  3  4  5  6  7  8  9 10 
##  7  5 18 14 15 15 14  4  5  3
```

```r
typeof(x)
```

```
## [1] "integer"
```

```r
attributes(x)
```

```
## $dim
## [1] 10
## 
## $dimnames
## $dimnames[[1]]
##  [1] "1"  "2"  "3"  "4"  "5"  "6"  "7"  "8"  "9"  "10"
## 
## 
## $class
## [1] "table"
```
Answer: This has a class attribute. The class is table and the base type is integer.

#13.3 Notes


```r
# Create and assign class in one step
x <- structure(list(), class = "my_class")

# Create, then set class
x <- list()
class(x) <- "my_class"
```


```r
class(x)
```

```
## [1] "my_class"
```

```r
#> [1] "my_class"
inherits(x, "my_class")
```

```
## [1] TRUE
```

```r
#> [1] TRUE
inherits(x, "your_class")
```

```
## [1] FALSE
```

```r
#> [1] FALSE
```

Constructors should: Be called new_myclass(); have one argument for the base object, and one for each attribute; check the type of the base object and the types of each attribute


```r
new_Date <- function(x = double()) {
  stopifnot(is.double(x))
  structure(x, class = "Date")
}

new_Date(c(-1, 0, 1))
```

```
## [1] "1969-12-31" "1970-01-01" "1970-01-02"
```

```r
#> [1] "1969-12-31" "1970-01-01" "1970-01-02"
```

Validators


```r
validate_factor <- function(x) {
  values <- unclass(x)
  levels <- attr(x, "levels")

  if (!all(!is.na(values) & values > 0)) {
    stop(
      "All `x` values must be non-missing and greater than zero",
      call. = FALSE
    )
  }

  if (length(levels) < max(values)) {
    stop(
      "There must be at least as many `levels` as possible values in `x`",
      call. = FALSE
    )
  }

  x
}
```

Helpers should: Have the same name as the class; e.g. myclass(), finish by calling the constructor, and the validator, if it exists; create carefully crafted error messages tailored towards an end-user; have a thoughtfully crafted user interface with carefully chosen default values and useful conversions



```r
POSIXct <- function(year = integer(), 
                    month = integer(), 
                    day = integer(), 
                    hour = 0L, 
                    minute = 0L, 
                    sec = 0, 
                    tzone = "") {
  ISOdatetime(year, month, day, hour, minute, sec, tz = tzone)
}

POSIXct(2020, 1, 1, tzone = "America/New_York")
```

```
## [1] "2020-01-01 EST"
```

```r
#> [1] "2020-01-01 EST"
```

#13.3 Exercises

1. Write a constructor for data.frame objects. What base type is a data frame built on? What attributes does it use? What are the restrictions placed on the individual elements? What about the names?

```r
new_data.frame <- function(..., row.names=NULL, check.names=TRUE) {
  x <- list(...)
  if(is.null(row.names)==FALSE && is.integer(row.names)==FALSE && is.character(row.names)==FALSE){stop()}
  stopifnot(is.logical(check.names))
  structure(x, row.names=NULL, check.names=TRUE, class = "data.frame")
}
```

2. Enhance my factor() helper to have better behaviour when one or more values is not found in levels. What does base::factor() do in this situation?

```r
#factor <- function(x = character(), levels = unique(x)) {
#  ind <- match(x, levels)
#  validate_factor(new_factor(ind, levels))
#}

factor(c("a", "a", "b"))
```

```
## [1] a a b
## Levels: a b
```

```r
#> [1] a a b
#> Levels: a b
```
base::factor() takes each unique value from x, orders them, and coerces them to characters

3. Carefully read the source code of factor(). What does it do that my constructor does not?

```r
factor
```

```
## function (x = character(), levels, labels = levels, exclude = NA, 
##     ordered = is.ordered(x), nmax = NA) 
## {
##     if (is.null(x)) 
##         x <- character()
##     nx <- names(x)
##     if (missing(levels)) {
##         y <- unique(x, nmax = nmax)
##         ind <- order(y)
##         levels <- unique(as.character(y)[ind])
##     }
##     force(ordered)
##     if (!is.character(x)) 
##         x <- as.character(x)
##     levels <- levels[is.na(match(levels, exclude))]
##     f <- match(x, levels)
##     if (!is.null(nx)) 
##         names(f) <- nx
##     if (missing(labels)) {
##         levels(f) <- as.character(levels)
##     }
##     else {
##         nlab <- length(labels)
##         if (nlab == length(levels)) {
##             nlevs <- unique(xlevs <- as.character(labels))
##             at <- attributes(f)
##             at$levels <- nlevs
##             f <- match(xlevs, nlevs)[f]
##             attributes(f) <- at
##         }
##         else if (nlab == 1L) 
##             levels(f) <- paste0(labels, seq_along(levels))
##         else stop(gettextf("invalid 'labels'; length %d should be 1 or %d", 
##             nlab, length(levels)), domain = NA)
##     }
##     class(f) <- c(if (ordered) "ordered", "factor")
##     f
## }
## <bytecode: 0x00000259a48bc2e8>
## <environment: namespace:base>
```
It coerces data types whereas my_factor only checks data types    

4. Factors have an optional “contrasts” attribute. Read the help for C(), and briefly describe the purpose of the attribute. What type should it have? Rewrite the new_factor() constructor to include this attribute.   
Answer: the "contrasts" attribute can be a matrix or function. It determines the type constrasts applied

5. Read the documentation for utils::as.roman(). How would you write a constructor for this class? Does it need a validator? What might a helper do?      
Answer: the constructor would check that the input is one of the correct types: a numeric or character vector of arabic or roman numerals. Then it would define the integer equivalent and return the correct structure with class = roman. 

