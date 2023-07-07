---
title: "chapter-18"
author: "Alicia Sillers"
date: "2023-07-07"
output: 
  html_document:
    keep_md: TRUE
---




```r
library(rlang)
library(lobstr)
```

```
## Warning: package 'lobstr' was built under R version 4.2.3
```

#18.2 Notes

Branches of the tree are called objects and leaves of the tree are symbols or constants.

```r
lobstr::ast(f(x, "y", 1))
```

```
## █─f 
## ├─x 
## ├─"y" 
## └─1
```

#18.2 Exercises

1. Reconstruct the code represented by the trees below:

```r
#> █─f 
#> └─█─g 
#>   └─█─h
#> █─`+` 
#> ├─█─`+` 
#> │ ├─1 
#> │ └─2 
#> └─3
#> █─`*` 
#> ├─█─`(` 
#> │ └─█─`+` 
#> │   ├─x 
#> │   └─y 
#> └─z
```
Answer:

```r
lobstr::ast(f(g(h())))
```

```
## █─f 
## └─█─g 
##   └─█─h
```

```r
lobstr::ast(f(1 + 2 + 3))
```

```
## █─f 
## └─█─`+` 
##   ├─█─`+` 
##   │ ├─1 
##   │ └─2 
##   └─3
```

```r
lobstr::ast(f((x + y)*z))
```

```
## █─f 
## └─█─`*` 
##   ├─█─`(` 
##   │ └─█─`+` 
##   │   ├─x 
##   │   └─y 
##   └─z
```

2. Draw the following trees by hand and then check your answers with lobstr::ast().

```r
ast(f(g(h(i(1, 2, 3)))))
```

```
## █─f 
## └─█─g 
##   └─█─h 
##     └─█─i 
##       ├─1 
##       ├─2 
##       └─3
```

```r
ast(f(1, g(2, h(3, i()))))
```

```
## █─f 
## ├─1 
## └─█─g 
##   ├─2 
##   └─█─h 
##     ├─3 
##     └─█─i
```

```r
ast(f(g(1, 2), h(3, i(4, 5))))
```

```
## █─f 
## ├─█─g 
## │ ├─1 
## │ └─2 
## └─█─h 
##   ├─3 
##   └─█─i 
##     ├─4 
##     └─5
```

3. What’s happening with the ASTs below? (Hint: carefully read ?"^".)

```r
lobstr::ast(`x` + `y`)
```

```
## █─`+` 
## ├─x 
## └─y
```

```r
#> █─`+` 
#> ├─x 
#> └─y
lobstr::ast(x ** y)
```

```
## █─`^` 
## ├─x 
## └─y
```

```r
#> █─`^` 
#> ├─x 
#> └─y
lobstr::ast(1 -> x)
```

```
## █─`<-` 
## ├─x 
## └─1
```

```r
#> █─`<-` 
#> ├─x 
#> └─1
```
Answer: ** is translated to ^.

4. What is special about the AST below? (Hint: re-read Section 6.2.1.)

```r
lobstr::ast(function(x = 1, y = 2) {})
```

```
## █─`function` 
## ├─█─x = 1 
## │ └─y = 2 
## ├─█─`{` 
## └─<inline srcref>
```

```r
#> █─`function` 
#> ├─█─x = 1 
#> │ └─y = 2 
#> ├─█─`{` 
#> └─<inline srcref>
```
Answer: This AST has a function with more parts than the others we have seen so far, in that it includes curly braces for a function body, even though there is nothing inside of them. srcref is short for source reference and is an attribute of functions that is used to print the function body with comments and formatting.   

5. What does the call tree of an if statement with multiple else if conditions look like? Why?

```r
ast(
  if(x < 39) y <- "haplotig"
  else if(39 <= x && x < 66) y <- "diplotig"
  else if(66 <= x && x < 92) y <- "triplotig"
  else if(92 <= x) y <- "tetraplotig"
)
```

```
## █─`if` 
## ├─█─`<` 
## │ ├─x 
## │ └─39 
## ├─█─`<-` 
## │ ├─y 
## │ └─"haplotig" 
## └─█─`if` 
##   ├─█─`&&` 
##   │ ├─█─`<=` 
##   │ │ ├─39 
##   │ │ └─x 
##   │ └─█─`<` 
##   │   ├─x 
##   │   └─66 
##   ├─█─`<-` 
##   │ ├─y 
##   │ └─"diplotig" 
##   └─█─`if` 
##     ├─█─`&&` 
##     │ ├─█─`<=` 
##     │ │ ├─66 
##     │ │ └─x 
##     │ └─█─`<` 
##     │   ├─x 
##     │   └─92 
##     ├─█─`<-` 
##     │ ├─y 
##     │ └─"triplotig" 
##     └─█─`if` 
##       ├─█─`<=` 
##       │ ├─92 
##       │ └─x 
##       └─█─`<-` 
##         ├─y 
##         └─"tetraplotig"
```
Answer: All of the else if statements just show up as 'if' in the tree. There is an indentation with each 'if' statement even though the else if statements are not nested within each other, probably because they each depend on the statement before. 

#18.3 Notes

An expression is any member of the set of base types created by parsing code: constant scalars, symbols, call objects, and pairlists.   

A constant is either NULL or a length-1 atomic vector or scalar. Constants are self-quoting in the sense that the expression used to represent a constant is the same constant:

```r
identical(expr(TRUE), TRUE)
```

```
## [1] TRUE
```

```r
#> [1] TRUE
identical(expr(1), 1)
```

```
## [1] TRUE
```

```r
#> [1] TRUE
identical(expr(2L), 2L)
```

```
## [1] TRUE
```

```r
#> [1] TRUE
identical(expr("x"), "x")
```

```
## [1] TRUE
```

```r
#> [1] TRUE
```

A symbol represents the name of an object. You can create a symbol in two ways: by capturing code that references an object with expr(), or turning a string into a symbol with rlang::sym()   

A call object represents a captured function call. Call objects are a special type of list90 where the first component specifies the function to call (usually a symbol), and the remaining elements are the arguments for that call.

#18.3 Exercises

1. Which two of the six types of atomic vector can’t appear in an expression? Why? Similarly, why can’t you create an expression that contains an atomic vector of length greater than one?   
Answer: Complex and raw atomic vectors can't appear in an expression. An atomic vector of length greater than one is not a constant.    

2. What happens when you subset a call object to remove the first element? e.g. expr(read.csv("foo.csv", header = TRUE))[-1]. Why?   

```r
expr(read.csv("foo.csv", header = TRUE))[-1]
```

```
## "foo.csv"(header = TRUE)
```


3. Describe the differences between the following call objects.

```r
x <- 1:10

call2(median, x, na.rm = TRUE)
```

```
## (function (x, na.rm = FALSE, ...) 
## UseMethod("median"))(1:10, na.rm = TRUE)
```

```r
call2(expr(median), x, na.rm = TRUE)
```

```
## median(1:10, na.rm = TRUE)
```

```r
call2(median, expr(x), na.rm = TRUE)
```

```
## (function (x, na.rm = FALSE, ...) 
## UseMethod("median"))(x, na.rm = TRUE)
```

```r
call2(expr(median), expr(x), na.rm = TRUE)
```

```
## median(x, na.rm = TRUE)
```


4. rlang::call_standardise() doesn’t work so well for the following calls. Why? What makes mean() special?

```r
call_standardise(quote(mean(1:10, na.rm = TRUE)))
```

```
## mean(x = 1:10, na.rm = TRUE)
```

```r
#> mean(x = 1:10, na.rm = TRUE)
call_standardise(quote(mean(n = T, 1:10)))
```

```
## mean(x = 1:10, n = T)
```

```r
#> mean(x = 1:10, n = T)
call_standardise(quote(mean(x = 1:10, , TRUE)))
```

```
## mean(x = 1:10, , TRUE)
```

```r
#> mean(x = 1:10, , TRUE)
```


5. Why does this code not make sense?

```r
x <- expr(foo(x = 1))
names(x) <- c("x", "y")
```


6. Construct the expression if(x > 1) "a" else "b" using multiple calls to call2(). How does the code structure reflect the structure of the AST?
