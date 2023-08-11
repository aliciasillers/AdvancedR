---
title: "chapter-20"
author: "Alicia Sillers"
date: "2023-07-28"
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


#20.1 Notes

Quosure: a data structure that captures an expression along with its associated environment, as found in function arguments   

Data mask: makes it easier to evaluate an expression in the context of a data frame

#20.2 Notes

for eval(), expr is the object to evaluate

```r
x <- 10
eval(expr(x))
```

```
## [1] 10
```

```r
#> [1] 10

y <- 2
eval(expr(x + y))
```

```
## [1] 12
```

```r
#> [1] 12
```
env is the environment in which the expression should be evaluated, determining where to look for values

```r
eval(expr(x + y), env(x = 1000))
```

```
## [1] 1002
```

```r
#> [1] 1002
```


```r
# Clean up variables created earlier
rm(x, y)

foo <- local({
  x <- 10
  y <- 200
  x + y
})

foo
```

```
## [1] 210
```

```r
#x
#> Error in eval(expr, envir, enclos): object 'x' not found
#y
#> Error in eval(expr, envir, enclos): object 'y' not found
```

read in the file from disk, use parse_expr() to parse the string into a list of expressions, and then use eval() to evaluate each element in turn

```r
source2 <- function(path, env = caller_env()) {
  file <- paste(readLines(path, warn = FALSE), collapse = "\n")
  exprs <- parse_exprs(file)

  res <- NULL
  for (i in seq_along(exprs)) {
    res <- eval(exprs[[i]], env)
  }

  invisible(res)
}
```

base::eval() has special behaviour for expression vectors, evaluating each component in turn.

```r
source3 <- function(file, env = parent.frame()) {
  lines <- parse(file)
  res <- eval(lines, envir = env)
  invisible(res)
}
```

functions print their srcref attribute (Section 6.2.1), and because srcref is a base R feature it’s unaware of quasiquotation

```r
x <- 10
y <- 20
f <- eval(expr(function(x, y) !!x + !!y))
f
```

```
## function(x, y) !!x + !!y
```

```r
f()
```

```
## [1] 30
```

#20.2 Exercises

1. Carefully read the documentation for source(). What environment does it use by default? What if you supply local = TRUE? How do you provide a custom environment?    
Answer: The default environment is the global environment. When local = TRUE, the environment is the environment from which source is called. You can provide a custom environment with local = env, where env is the custom environment. 

2. Predict the results of the following lines of code:

```r
eval(expr(eval(expr(eval(expr(2 + 2))))))
```

```
## [1] 4
```

```r
eval(eval(expr(eval(expr(eval(expr(2 + 2)))))))
```

```
## [1] 4
```

```r
expr(eval(expr(eval(expr(eval(expr(2 + 2)))))))
```

```
## eval(expr(eval(expr(eval(expr(2 + 2))))))
```
Answer: I predict that the first two will both output 4 while the last one will output eval(expr(eval(expr(eval(expr(2 + 2))))))

3. Fill in the function bodies below to re-implement get() using sym() and eval(), and assign() using sym(), expr(), and eval(). Don’t worry about the multiple ways of choosing an environment that get() and assign() support; assume that the user supplies it explicitly.

```r
# name is a string
get2 <- function(name, env) {
  name2 <- sym(name)
  eval(name2, env)
}
assign2 <- function(name, value, env) {
  name2 <- sym(name)
  value2 <- expr(!!name2 <- !!value)
  eval(value2, env)
}
```

4. Modify source2() so it returns the result of every expression, not just the last one. Can you eliminate the for loop?

```r
source2 <- function(path, env = caller_env()) {
  file <- paste(readLines(path, warn = FALSE), collapse = "\n")
  exprs <- parse_exprs(file)
  
  results <- map2(exprs, env, eval)

  results
}
```

5. We can make base::local() slightly easier to understand by spreading out over multiple lines:

```r
local3 <- function(expr, envir = new.env()) {
  call <- substitute(eval(quote(expr), envir))
  eval(call, envir = parent.frame())
}
```
Explain how local() works in words. (Hint: you might want to print(call) to help understand what substitute() is doing, and read the documentation to remind yourself what environment new.env() will inherit from.)   
Answer: local captures the input expression and creates a new environment in which to evaluate it. 

#20.3 Notes

Use enquo() and enquos() to capture user-supplied expressions. The vast majority of quosures should be created this way. 
quo() and quos() exist to match to expr() and exprs(), but they are included only for the sake of completeness and are needed very rarely. 

```r
foo <- function(x) enquo(x)
foo(a + b)
```

```
## <quosure>
## expr: ^a + b
## env:  global
```

```r
#> <quosure>
#> expr: ^a + b
#> env:  global
```

new_quosure() creates a quosure from its components: an expression and an environment.

```r
new_quosure(expr(x + y), env(x = 1, y = 10))
```

```
## <quosure>
## expr: ^x + y
## env:  0x000001c2f98e8ea0
```

```r
#> <quosure>
#> expr: ^x + y
#> env:  0x7fac62d44870
```

Quosures are paired with a new evaluation function eval_tidy() that takes a single quosure instead of an expression-environment pair.

```r
q1 <- new_quosure(expr(x + y), env(x = 1, y = 10))
eval_tidy(q1)
```

```
## [1] 11
```

```r
#> [1] 11
```

it’s possible for each argument passed to ... to be associated with a different environment

```r
f <- function(...) {
  x <- 1
  g(..., f = x)
}
g <- function(...) {
  enquos(...)
}

x <- 0
qs <- f(global = x)
qs
```

```
## <list_of<quosure>>
## 
## $global
## <quosure>
## expr: ^x
## env:  global
## 
## $f
## <quosure>
## expr: ^x
## env:  0x000001c2faa0c908
```

```r
#> <list_of<quosure>>
#> 
#> $global
#> <quosure>
#> expr: ^x
#> env:  global
#> 
#> $f
#> <quosure>
#> expr: ^x
#> env:  0x7fac60661d88
```

It’s possible to use quasiquotation to embed a quosure in an expression.

```r
q2 <- new_quosure(expr(x), env(x = 1))
q3 <- new_quosure(expr(x), env(x = 10))

x <- expr(!!q2 + !!q3)

eval_tidy(x)
```

```
## [1] 11
```

```r
#> [1] 11
 
x
```

```
## (~x) + ~x
```

```r
#> (~x) + ~x

expr_print(x)
```

```
## (^x) + (^x)
```

```r
#> (^x) + (^x)
```

#20.3 Exercises

1. Predict what each of the following quosures will return if evaluated.

```r
q1 <- new_quosure(expr(x), env(x = 1))
q1
```

```
## <quosure>
## expr: ^x
## env:  0x000001c2ff08f880
```

```r
#> <quosure>
#> expr: ^x
#> env:  0x7fac62d19130

q2 <- new_quosure(expr(x + !!q1), env(x = 10))
q2
```

```
## <quosure>
## expr: ^x + (^x)
## env:  0x000001c2fd63d5d0
```

```r
#> <quosure>
#> expr: ^x + (^x)
#> env:  0x7fac62e35a98

q3 <- new_quosure(expr(x + !!q2), env(x = 100))
q3
```

```
## <quosure>
## expr: ^x + (^x + (^x))
## env:  0x000001c2fdab80d8
```

```r
#> <quosure>
#> expr: ^x + (^x + (^x))
#> env:  0x7fac6302feb0
```
Answer: I predict that eval_tidy(q1) will return 1, eval_tidy(q2) will return 11, and eval_tidy(q3) will return 111.    

2. Write an enenv() function that captures the environment associated with an argument. (Hint: this should only require two function calls.)

```r
enenv <- function(x){
  get_env(x)
}
```


#20.4 Exercises

1. Why did I use a for loop in transform2() instead of map()? Consider transform2(df, x = x * 2, x = x * 2).   
Answer: so that it can be step-wise

2. Here’s an alternative implementation of subset2():

```r
subset3 <- function(data, rows) {
  rows <- enquo(rows)
  eval_tidy(expr(data[!!rows, , drop = FALSE]), data = data)
}

df <- data.frame(x = 1:3)
subset3(df, x == 1)
```

```
##   x
## 1 1
```
Compare and contrast subset3() to subset2(). What are its advantages and disadvantages?   
Answer: subset3 is more succinct, with subsetting occurring with the data mask

3. The following function implements the basics of dplyr::arrange(). Annotate each line with a comment explaining what it does. Can you explain why !!.na.last is strictly correct, but omitting the !! is unlikely to cause problems?

```r
arrange2 <- function(.df, ..., .na.last = TRUE) {
  args <- enquos(...) #captures and quotes arguments

  order_call <- expr(order(!!!args, na.last = !!.na.last)) #defines order_call as an expression

  ord <- eval_tidy(order_call, .df) #evaluates order_call
  stopifnot(length(ord) == nrow(.df)) #checks to make sure ord length is the same as number of rows

  .df[ord, , drop = FALSE] #reorder rows
}
```


#20.5 Exercises

1. I’ve included an alternative implementation of threshold_var() below. What makes it different to the approach I used above? What makes it harder?

```r
threshold_var <- function(df, var, val) {
  var <- ensym(var)
  subset2(df, `$`(.data, !!var) >= !!val)
}
```
Answer: ensym(var) is not coerced to a string

#20.6 Notes

There are three pieces that you’ll use whenever wrapping a base NSE function:

You capture the unevaluated arguments using enexpr(), and capture the caller environment using caller_env().

You generate a new expression using expr() and unquoting.

You evaluate that expression in the caller environment. You have to accept that the function will not work correctly if the arguments are not defined in the caller environment. Providing the env argument at least provides a hook that experts can use if the default environment isn’t correct.

#20.6 Exercises

1. Why does this function fail?

```r
lm3a <- function(formula, data) {
  formula <- enexpr(formula)

  lm_call <- expr(lm(!!formula, data = data))
  eval(lm_call, caller_env())
}
#lm3a(mpg ~ disp, mtcars)$call
#> Error in as.data.frame.default(data, optional = TRUE): 
#> cannot coerce class ‘"function"’ to a data.frame
```
Answer: data is already defined in the global environment 

2. When model building, typically the response and data are relatively constant while you rapidly experiment with different predictors. Write a small wrapper that allows you to reduce duplication in the code below.

```r
lm(mpg ~ disp, data = mtcars)
```

```
## 
## Call:
## lm(formula = mpg ~ disp, data = mtcars)
## 
## Coefficients:
## (Intercept)         disp  
##    29.59985     -0.04122
```

```r
lm(mpg ~ I(1 / disp), data = mtcars)
```

```
## 
## Call:
## lm(formula = mpg ~ I(1/disp), data = mtcars)
## 
## Coefficients:
## (Intercept)    I(1/disp)  
##       10.75      1557.67
```

```r
lm(mpg ~ disp * cyl, data = mtcars)
```

```
## 
## Call:
## lm(formula = mpg ~ disp * cyl, data = mtcars)
## 
## Coefficients:
## (Intercept)         disp          cyl     disp:cyl  
##    49.03721     -0.14553     -3.40524      0.01585
```

```r
wrapper <- function(formula, data = mtcars, env = caller_env()){
  formula <- enexp(formula)
  data <- enexpr(data)
  
  lm_call <- expr(lm(formula = !!formula, data = !!data))
  eval(lm_call, envir = env)
}
```

3. Another way to write resample_lm() would be to include the resample expression (data[sample(nrow(data), replace = TRUE), , drop = FALSE]) in the data argument. Implement that approach. What are the advantages? What are the disadvantages?

```r
resample_lm <- function(formula, data,
  resample_data = data[sample(nrow(data), replace = TRUE), ,
                       drop = FALSE],
  env = current_env()) {
  formula <- enexpr(formula)

  lm_env <- env(env, resample_data = resample_data)
  lm_call <- expr(lm(!!formula, data = resample_data))
  expr_print(lm_call)
  eval(lm_call, lm_env)
}
```

