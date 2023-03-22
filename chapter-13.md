---
title: "chapter-13"
output: 
  html_document:
    keep_md: TRUE
date: "2023-02-27"
---



```r
library(sloop)
library(tidyverse)
```

```
## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.2 ──
## ✔ ggplot2 3.4.1      ✔ purrr   1.0.1 
## ✔ tibble  3.1.8      ✔ dplyr   1.0.10
## ✔ tidyr   1.3.0      ✔ stringr 1.5.0 
## ✔ readr   2.1.4      ✔ forcats 1.0.0 
## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::filter() masks stats::filter()
## ✖ dplyr::lag()    masks stats::lag()
```

```r
library(purrr)
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
## <bytecode: 0x000001e7ee551810>
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


```r
new_factor <- function(x = integer(), levels = character()) {
  stopifnot(is.integer(x))
  stopifnot(is.character(levels))

  structure(
    x,
    levels = levels,
    class = "factor"
  )
}
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
factor <- function(x = character(), levels = unique(x)) {
  ind <- match(x, levels)
  #stopifnot(ind)
  validate_factor(new_factor(ind, levels))
}

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
## function(x = character(), levels = unique(x)) {
##   ind <- match(x, levels)
##   #stopifnot(ind)
##   validate_factor(new_factor(ind, levels))
## }
```
It coerces data types whereas my_factor only checks data types    

4. Factors have an optional “contrasts” attribute. Read the help for C(), and briefly describe the purpose of the attribute. What type should it have? Rewrite the new_factor() constructor to include this attribute.   
Answer: the "contrasts" attribute can be a matrix or function. It determines the type constrasts applied

5. Read the documentation for utils::as.roman(). How would you write a constructor for this class? Does it need a validator? What might a helper do?      
Answer: the constructor would check that the input is one of the correct types: a numeric or character vector of arabic or roman numerals. Then it would define the integer equivalent and return the correct structure with class = roman. 

#13.4 Notes

##generics and methods
method dispatch is finding the specific implementation for a class, and it is performed by UseMethod(), which takes the name of a generic function and, optionally, the argument to use for method dispatch.    

##method dispatch
UseMethod() creates a vector of method names and looks for each potential method in turn. s3_dispatch() lists all possible methods. 


```r
x <- matrix(1:10, nrow = 2)
s3_dispatch(mean(x))
```

```
##    mean.matrix
##    mean.integer
##    mean.numeric
## => mean.default
```

```r
#>    mean.matrix
#>    mean.integer
#>    mean.numeric
#> => mean.default

s3_dispatch(sum(Sys.time()))
```

```
##    sum.POSIXct
##    sum.POSIXt
##    sum.default
## => Summary.POSIXct
##    Summary.POSIXt
##    Summary.default
## -> sum (internal)
```

```r
#>    sum.POSIXct
#>    sum.POSIXt
#>    sum.default
#> => Summary.POSIXct
#>    Summary.POSIXt
#>    Summary.default
#> -> sum (internal)
```

##finding methods
s3_methods_generic() and s3_methods_class() give you all the methods defined for a generic or associated with a class.


```r
s3_methods_generic("mean")
```

```
## # A tibble: 7 × 4
##   generic class      visible source             
##   <chr>   <chr>      <lgl>   <chr>              
## 1 mean    Date       TRUE    base               
## 2 mean    default    TRUE    base               
## 3 mean    difftime   TRUE    base               
## 4 mean    POSIXct    TRUE    base               
## 5 mean    POSIXlt    TRUE    base               
## 6 mean    quosure    FALSE   registered S3method
## 7 mean    vctrs_vctr FALSE   registered S3method
```

```r
#> # A tibble: 7 x 4
#>   generic class      visible source             
#>   <chr>   <chr>      <lgl>   <chr>              
#> 1 mean    Date       TRUE    base               
#> 2 mean    default    TRUE    base               
#> 3 mean    difftime   TRUE    base               
#> 4 mean    POSIXct    TRUE    base               
#> 5 mean    POSIXlt    TRUE    base               
#> 6 mean    quosure    FALSE   registered S3method
#> 7 mean    vctrs_vctr FALSE   registered S3method

s3_methods_class("ordered")
```

```
## # A tibble: 6 × 4
##   generic       class   visible source             
##   <chr>         <chr>   <lgl>   <chr>              
## 1 as.data.frame ordered TRUE    base               
## 2 Ops           ordered TRUE    base               
## 3 relevel       ordered FALSE   registered S3method
## 4 scale_type    ordered FALSE   registered S3method
## 5 Summary       ordered TRUE    base               
## 6 type_sum      ordered FALSE   registered S3method
```

```r
#> # A tibble: 4 x 4
#>   generic       class   visible source             
#>   <chr>         <chr>   <lgl>   <chr>              
#> 1 as.data.frame ordered TRUE    base               
#> 2 Ops           ordered TRUE    base               
#> 3 relevel       ordered FALSE   registered S3method
#> 4 Summary       ordered TRUE    base
```

##creating methods
rules for creating methods: only write a method if you own the generic or class; and a method must have the same arguments as its generic, unless the generic has ..., in which case the method can take arbitrary additional arguments.

#13.4 Exercises

1. Read the source code for t() and t.test() and confirm that t.test() is an S3 generic and not an S3 method. What happens if you create an object with class test and call t() with it? Why?   

```r
getAnywhere(t)
```

```
## A single object matching 't' was found
## It was found in the following places
##   package:base
##   namespace:base
## with value
## 
## function (x) 
## UseMethod("t")
## <bytecode: 0x000001e7ee386ef0>
## <environment: namespace:base>
```

```r
getAnywhere(t.test)
```

```
## A single object matching 't.test' was found
## It was found in the following places
##   package:stats
##   registered S3 method for t from namespace stats
##   namespace:stats
## with value
## 
## function (x, ...) 
## UseMethod("t.test")
## <bytecode: 0x000001e7e90b94e0>
## <environment: namespace:stats>
```

```r
s3_methods_generic("t.test")
```

```
## # A tibble: 2 × 4
##   generic class   visible source             
##   <chr>   <chr>   <lgl>   <chr>              
## 1 t.test  default FALSE   registered S3method
## 2 t.test  formula FALSE   registered S3method
```

```r
s3_methods_class("t.test")
```

```
## # A tibble: 0 × 4
## # … with 4 variables: generic <chr>, class <chr>, visible <lgl>, source <chr>
```

```r
x <- structure(x, class = "test")
t(x)
```

```
##      [,1] [,2]
## [1,]    1    2
## [2,]    3    4
## [3,]    5    6
## [4,]    7    8
## [5,]    9   10
## attr(,"class")
## [1] "test"
```

```r
s3_dispatch(t(x))
```

```
## => t.test
##  * t.default
```
Answer: Calling t(x) printed the types of attributes belonging to x and what the class attribute is. I'm not sure why, but maybe it has something to do with not having a method for the class test    

2. What generics does the table class have methods for?

```r
s3_methods_class("table")
```

```
## # A tibble: 11 × 4
##    generic       class visible source             
##    <chr>         <chr> <lgl>   <chr>              
##  1 [             table TRUE    base               
##  2 aperm         table TRUE    base               
##  3 as.data.frame table TRUE    base               
##  4 as_tibble     table FALSE   registered S3method
##  5 Axis          table FALSE   registered S3method
##  6 lines         table FALSE   registered S3method
##  7 plot          table FALSE   registered S3method
##  8 points        table FALSE   registered S3method
##  9 print         table TRUE    base               
## 10 summary       table TRUE    base               
## 11 tail          table FALSE   registered S3method
```

3. What generics does the ecdf class have methods for?

```r
s3_methods_class("ecdf")
```

```
## # A tibble: 4 × 4
##   generic  class visible source             
##   <chr>    <chr> <lgl>   <chr>              
## 1 plot     ecdf  TRUE    stats              
## 2 print    ecdf  FALSE   registered S3method
## 3 quantile ecdf  FALSE   registered S3method
## 4 summary  ecdf  FALSE   registered S3method
```

4. Which base generic has the greatest number of defined methods?

```r
#make list of generics
#get lengths of methods
#max length of methods

fxns <- tibble(fn_name={ls("package:base")}) %>%
  filter(map_lgl(fn_name, ~ {get(.) %>% is_function()})) %>%
  filter(map_lgl(fn_name, is_s3_generic))

getnum <- function(x){
  length(methods(x))
}

methodnum <- map_dbl(fxns$fn_name, getnum)
```

```r
fxns <- cbind(fxns, methodnum)
```

```r
maxmethods <- fxns %>% filter(fxns$methodnum == max(methodnum))
maxmethods[,1]
```

```
## [1] "print"
```
Answer: print has the greatest number of defined methods    

5. Carefully read the documentation for UseMethod() and explain why the following code returns the results that it does. What two usual rules of function evaluation does UseMethod() violate?

```r
g <- function(x) {
  x <- 10
  y <- 10
  UseMethod("g")
}
g.default <- function(x) c(x = x, y = y)

x <- 1
y <- 1
g(x)
```

```
##  x  y 
##  1 10
```

```r
#>  x  y 
#>  1 10
```
Answer: It seems like x is 1 because it is an argument is g.default and gets the value from the global environment, whereas y is 10 because this value is defined in the function body. I think getting the value of x from the parent environment over the function body violates the rule of name masking. It has this behavior because of the method used, which sets x back to the argument value of x.    

6. What are the arguments to [? Why is this a hard question to answer?    
Answer: Arguments for extract/replace operators are x, object, i, j, name, drop, exact, value. If I am answering the question correctly, I'm not sure why it is a hard question to answer, except maybe that you have to put the [ in quotation marks when using the command ?']' or else R thinks there is a syntax error.

#13.5 Notes

Record style objects use a list of equal-length vectors to represent individual components of an object. Record style classes override length() and subsetting methods to conceal this implementation detail.   


```r
x <- as.POSIXlt(ISOdatetime(2020, 1, 1, 0, 0, 1:3))
x
```

```
## [1] "2020-01-01 00:00:01 PST" "2020-01-01 00:00:02 PST"
## [3] "2020-01-01 00:00:03 PST"
```

```r
#> [1] "2020-01-01 00:00:01 UTC" "2020-01-01 00:00:02 UTC"
#> [3] "2020-01-01 00:00:03 UTC"

length(x)
```

```
## [1] 3
```

```r
#> [1] 3
length(unclass(x))
```

```
## [1] 11
```

```r
#> [1] 9

x[[1]] # the first date time
```

```
## [1] "2020-01-01 00:00:01 PST"
```

```r
#> [1] "2020-01-01 00:00:01 UTC"
unclass(x)[[1]] # the first component, the number of seconds
```

```
## [1] 1 2 3
```

```r
#> [1] 1 2 3
```

Data frames use lists of equal length vectors but are conceptually two dimensional, and the individual components are readily exposed to the user. The number of observations is the number of rows, not the length.


```r
x <- data.frame(x = 1:100, y = 1:100)
length(x)
```

```
## [1] 2
```

```r
#> [1] 2
nrow(x)
```

```
## [1] 100
```

```r
#> [1] 100
```

Scalar objects typically use a list to represent a single thing. They can also be built on top of functions, calls, and environments.


```r
mod <- lm(mpg ~ wt, data = mtcars)
length(mod)
```

```
## [1] 12
```

```r
#> [1] 12
```

#13.5 Exercises

1. Categorise the objects returned by lm(), factor(), table(), as.Date(), as.POSIXct() ecdf(), ordered(), I() into the styles described above.   
Answer:   
lm() -> scalar    
factor() -> vector    
table() -> data frame   
as.Date() -> vector   
as.POSIXct() -> record    
ecdf() -> scalar    
ordered() -> vector   
I() -> vector   

2. What would a constructor function for lm objects, new_lm(), look like? Use ?lm and experimentation to figure out the required fields and their types.

```r
new_lm <- function(f, method = "qr", x = FALSE, y = FALSE, qr = TRUE, singular.ok = TRUE, model = TRUE){
  stopifnot(class(f) != "formula")
  stopifnot(method = "qr")
  stopifnot(is.logical(model))
  stopifnot(is.logical(x))
  stopifnot(is.logical(y))
  stopifnot(is.logical(qr))
  stopifnot(is.logical(singular.ok))
  structure(x, method, model, x, y, qr, singular.ok, class = "formula")
}
```

#13.6 Notes

Classes can be character vectors. R checks each class in order until it finds one for which there is a method. Subclasses are listed before superclasses.

Recommended rules for subclasses and superclasses: They should be the same base type, and the attributes of the subclass should be a superset of the attributes for the superclass


```r
new_secret <- function(x = double()) {
  stopifnot(is.double(x))
  structure(x, class = "secret")
}

print.secret <- function(x, ...) {
  print(strrep("x", nchar(x)))
  invisible(x)
}

x <- new_secret(c(15, 1, 456))
x
```

```
## [1] "xx"  "x"   "xxx"
```

```r
#> [1] "xx"  "x"   "xxx"
```


```r
`[.secret` <- function(x, i) {
  new_secret(NextMethod())
}
x[1]
```

```
## [1] "xx"
```

```r
#> [1] "xx"
```


```r
s3_dispatch(x[1])
```

```
## => [.secret
##    [.default
## -> [ (internal)
```

```r
#> => [.secret
#>    [.default
#> -> [ (internal)
```
The => indicates that [.secret is called, but that NextMethod() delegates work to the underlying internal [ method, as shown by the ->.   

To allow subclasses, the parent constructor needs to have ... and class arguments:

```r
new_secret <- function(x, ..., class = character()) {
  stopifnot(is.double(x))

  structure(
    x,
    ...,
    class = c(class, "secret")
  )
}
```

vctrs::vec_restore() generic takes two inputs: an object which has lost subclass information, and a template object to use for restoration. Typically vec_restore() methods are quite simple: you just call the constructor with appropriate arguments:

```r
vec_restore.secret <- function(x, to, ...) new_secret(x)
vec_restore.supersecret <- function(x, to, ...) new_supersecret(x)
```


```r
`[.secret` <- function(x, ...) {
  vctrs::vec_restore(NextMethod(), x)
}
x[1:3]
```

```
## [1] "xx"  "x"   "xxx"
```

If you build your class using the tools provided by the vctrs package, [ will gain this behaviour automatically. You will only need to provide your own [ method if you use attributes that depend on the data or want non-standard subsetting behaviour. 

#13.6 Exercises

1. How does [.Date support subclasses? How does it fail to support subclasses?

```r
s3_methods_class("Date")
```

```
## # A tibble: 46 × 4
##    generic class visible source             
##    <chr>   <chr> <lgl>   <chr>              
##  1 -       Date  TRUE    base               
##  2 !=      Date  FALSE   registered S3method
##  3 [       Date  TRUE    base               
##  4 [[      Date  TRUE    base               
##  5 [<-     Date  TRUE    base               
##  6 +       Date  TRUE    base               
##  7 <       Date  FALSE   registered S3method
##  8 <=      Date  FALSE   registered S3method
##  9 ==      Date  FALSE   registered S3method
## 10 >       Date  FALSE   registered S3method
## # … with 36 more rows
```

```r
attributes(date)
```

```
## NULL
```


2. R has two classes for representing date time data, POSIXct and POSIXlt, which both inherit from POSIXt. Which generics have different behaviours for the two classes? Which generics share the same behaviour?

```r
s3_methods_class("POSIXct")
```

```
## # A tibble: 19 × 4
##    generic       class   visible source             
##    <chr>         <chr>   <lgl>   <chr>              
##  1 [             POSIXct TRUE    base               
##  2 [[            POSIXct TRUE    base               
##  3 [<-           POSIXct TRUE    base               
##  4 as.data.frame POSIXct TRUE    base               
##  5 as.Date       POSIXct TRUE    base               
##  6 as.list       POSIXct TRUE    base               
##  7 as.POSIXlt    POSIXct TRUE    base               
##  8 c             POSIXct TRUE    base               
##  9 format        POSIXct TRUE    base               
## 10 full_seq      POSIXct FALSE   registered S3method
## 11 length<-      POSIXct TRUE    base               
## 12 mean          POSIXct TRUE    base               
## 13 print         POSIXct TRUE    base               
## 14 rep           POSIXct TRUE    base               
## 15 split         POSIXct TRUE    base               
## 16 summary       POSIXct TRUE    base               
## 17 Summary       POSIXct TRUE    base               
## 18 weighted.mean POSIXct FALSE   registered S3method
## 19 xtfrm         POSIXct TRUE    base
```

```r
s3_methods_class("POSIXlt")
```

```
## # A tibble: 29 × 4
##    generic       class   visible source
##    <chr>         <chr>   <lgl>   <chr> 
##  1 [             POSIXlt TRUE    base  
##  2 [[            POSIXlt TRUE    base  
##  3 [[<-          POSIXlt TRUE    base  
##  4 [<-           POSIXlt TRUE    base  
##  5 anyNA         POSIXlt TRUE    base  
##  6 as.data.frame POSIXlt TRUE    base  
##  7 as.Date       POSIXlt TRUE    base  
##  8 as.double     POSIXlt TRUE    base  
##  9 as.list       POSIXlt TRUE    base  
## 10 as.matrix     POSIXlt TRUE    base  
## # … with 19 more rows
```


3. What do you expect this code to return? What does it actually return? Why?

```r
generic2 <- function(x) UseMethod("generic2")
generic2.a1 <- function(x) "a1"
generic2.a2 <- function(x) "a2"
generic2.b <- function(x) {
  class(x) <- "a1"
  NextMethod()
}

generic2(structure(list(), class = c("b", "a2")))
```

```
## [1] "a2"
```


#13.7 Notes

Dispatch occurs on the implicit class, which has three components: the string "array" or "matrix", the result of typeof(), and the string "numeric" if object is int or dbl.


```r
s3_class(matrix(1:5))
```

```
## [1] "matrix"  "integer" "numeric"
```

```r
#> [1] "matrix"  "integer" "numeric"
```
s3_class() will return the implicit class     

internal generics do not call UseMethod() and instead call the C functions DispatchGroup() or DispatchOrEval(). s3_dispatch() shows internal generics by including the name of the generic followed by (internal):

```r
s3_dispatch(Sys.time()[1])
```

```
## => [.POSIXct
##    [.POSIXt
##    [.default
## -> [ (internal)
```

```r
#> => [.POSIXct
#>    [.POSIXt
#>    [.default
#> -> [ (internal)
```


#13.7 Exercises

1. Explain the differences in dispatch below:

```r
length.integer <- function(x) 10

x1 <- 1:5
class(x1)
```

```
## [1] "integer"
```

```r
#> [1] "integer"
s3_dispatch(length(x1))
```

```
##  * length.integer
##    length.numeric
##    length.default
## => length (internal)
```

```r
#>  * length.integer
#>    length.numeric
#>    length.default
#> => length (internal)

x2 <- structure(x1, class = "integer")
class(x2)
```

```
## [1] "integer"
```

```r
#> [1] "integer"
s3_dispatch(length(x2))
```

```
## => length.integer
##    length.default
##  * length (internal)
```

```r
#> => length.integer
#>    length.default
#>  * length (internal)
```
Answer: Different methods are being called. In the first, the internal length method is being called, while in the second example, in which the class is defined as "integer", length.integer method is being called. The first example also has a length.numeric method while the second does not. 

2. What classes have a method for the Math group generic in base R? Read the source code. How do the methods work?

3. Math.difftime() is more complicated than I described. Why?
