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
#eval(expr(x + y), env(x = 1000))
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
## env:  0x0000021bbcd04fc0
```

```r
#> <quosure>
#> expr: ^x + y
#> env:  0x7fac62d44870
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
## env:  0x0000021bbbb2f3d8
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
## env:  0x0000021bbb2b52a0
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
## env:  0x0000021bbc75e5e0
```

```r
#> <quosure>
#> expr: ^x + (^x + (^x))
#> env:  0x7fac6302feb0
```
Answer:

2. Write an enenv() function that captures the environment associated with an argument. (Hint: this should only require two function calls.)


