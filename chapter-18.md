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

```r
library(codetools)
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
Answer: read.csv is removed   

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
Answer: Median is only interpreted as a function when it is not in expr(). The value of x is evaluated when x is not in expr().   

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
Answer: mean has the ... argument, so  call_standardise does not know all of the possible arguments.    

5. Why does this code not make sense?

```r
x <- expr(foo(x = 1))
names(x) <- c("x", "y")
x
```

```
## foo(y = 1)
```
Answer: A symbol always has length 1. c("x", "y") has a length greater than 1, plus the expression already has the name x.   

6. Construct the expression if(x > 1) "a" else "b" using multiple calls to call2(). How does the code structure reflect the structure of the AST?

```r
call2("if", (x > 1), "a")
```

```
## if (TRUE) "a"
```

```r
call2("else", "b")
```

```
## `else`("b")
```

#18.4 Notes

Precedence of infix operators is described in ?Syntax. Most operators are left-associative, but exponentiation and assignment are not.    
Parsing and deparsing are not perfectly symmetric because parsing generates an abstract syntax tree. This means we lose backticks around ordinary names, comments, and whitespace

#18.4 Exercises

1. R uses parentheses in two slightly different ways as illustrated by these two calls:

```r
ast(f((1)))
```

```
## █─f 
## └─█─`(` 
##   └─1
```

```r
ast((1 + 1))
```

```
## █─`(` 
## └─█─`+` 
##   ├─1 
##   └─1
```
Compare and contrast the two uses by referencing the AST.   
Answer: In the first tree, it has precedence over the the other object, f, whereas in the second tree, it does not take precedence over the other object, +. 

2. = can also be used in two ways. Construct a simple example that shows both uses.

```r
ast(1 == 1)
```

```
## █─`==` 
## ├─1 
## └─1
```

```r
ast(f(x=1))
```

```
## █─f 
## └─x = 1
```


3. Does -2^2 yield 4 or -4? Why?

```r
-2^2
```

```
## [1] -4
```

```r
ast(-2^2)
```

```
## █─`-` 
## └─█─`^` 
##   ├─2 
##   └─2
```
Answer: -4 because ^ takes precedent over -, so 2 is raised to the power of 2 and then the negative is applied

4. What does !1 + !1 return? Why?

```r
!1 + !1
```

```
## [1] FALSE
```

```r
ast(!1 + !1)
```

```
## █─`!` 
## └─█─`+` 
##   ├─1 
##   └─█─`!` 
##     └─1
```
Answer: It returns false because the precedence is left-associative, so it first applies the not operator, then adds one, and then applies the not operator again to return false    

5. Why does x1 <- x2 <- x3 <- 0 work? Describe the two reasons.

```r
x1 <- x2 <- x3 <- 0
ast(x1 <- x2 <- x3 <- 0)
```

```
## █─`<-` 
## ├─x1 
## └─█─`<-` 
##   ├─x2 
##   └─█─`<-` 
##     ├─x3 
##     └─0
```
Answer: Assignment is right-associative, so it applies 0 to x3 first and then works its way left

6. Compare the ASTs of x + y %+% z and x ^ y %+% z. What have you learned about the precedence of custom infix functions?

```r
ast(x + y %+% z)
```

```
## █─`+` 
## ├─x 
## └─█─`%+%` 
##   ├─y 
##   └─z
```

```r
ast(x ^ y %+% z)
```

```
## █─`%+%` 
## ├─█─`^` 
## │ ├─x 
## │ └─y 
## └─z
```
Answer: Custom infix functions take precedence over + but not over ^

7. What happens if you call parse_expr() with a string that generates multiple expressions? e.g. parse_expr("x + 1; y + 1")

```r
#parse_expr("x + 1; y + 1")
```
Answer: It returns an error with a message stating that the argument must contain one expression

8. What happens if you attempt to parse an invalid expression? e.g. "a +" or "f())".

```r
#parse_expr("a +")
```
Answer: It returns an error

9. deparse() produces vectors when the input is long. For example, the following call produces a vector of length two:

```r
expr <- expr(g(a + b + c + d + e + f + g + h + i + j + k + l + 
  m + n + o + p + q + r + s + t + u + v + w + x + y + z))

deparse(expr)
```

```
## [1] "g(a + b + c + d + e + f + g + h + i + j + k + l + m + n + o + "
## [2] "    p + q + r + s + t + u + v + w + x + y + z)"
```
What does expr_text() do instead?   

```r
expr_text(expr)
```

```
## [1] "g(a + b + c + d + e + f + g + h + i + j + k + l + m + n + o + \n    p + q + r + s + t + u + v + w + x + y + z)"
```
Answer: expr_text(expr) produces a single string but adds a \n to indicate that it goes to a new line

10. pairwise.t.test() assumes that deparse() always returns a length one character vector. Can you construct an input that violates this expectation? What happens?

```r
#pairwise.t.test(1, deparse(expr))
```
Answer: It returns an error stating that "g" in factor(g) is missing

#18.5 Notes

findGlobals() locates all global variables used by a function. This can be useful if you want to check that your function doesn’t inadvertently rely on variables defined in their parent environment.    

checkUsage() checks for a range of common problems including unused local variables, unused parameters, and the use of partial argument matching. The recursive case handles the nodes in the tree. The base case handles the leaves of the tree


```r
expr_type <- function(x) {
  if (rlang::is_syntactic_literal(x)) {
    "constant"
  } else if (is.symbol(x)) {
    "symbol"
  } else if (is.call(x)) {
    "call"
  } else if (is.pairlist(x)) {
    "pairlist"
  } else {
    typeof(x)
  }
}

expr_type(expr("a"))
```

```
## [1] "constant"
```

```r
#> [1] "constant"
expr_type(expr(x))
```

```
## [1] "symbol"
```

```r
#> [1] "symbol"
expr_type(expr(f(1, 2)))
```

```
## [1] "call"
```

```r
#> [1] "call"
```


```r
switch_expr <- function(x, ...) {
  switch(expr_type(x),
    ...,
    stop("Don't know how to handle type ", typeof(x), call. = FALSE)
  )
}
```


```r
recurse_call <- function(x) {
  switch_expr(x,
    # Base cases
    symbol = ,
    constant = ,

    # Recursive cases
    call = ,
    pairlist =
  )
}
```


```r
logical_abbr_rec <- function(x) {
  switch_expr(x,
    # Base cases
    constant = FALSE,
    symbol = as_string(x) %in% c("F", "T"),

    # Recursive cases
    call = ,
    pairlist = purrr::some(x, logical_abbr_rec)
  )
}
```


```r
logical_abbr <- function(x) {
  logical_abbr_rec(enexpr(x))
}

logical_abbr(T)
```

```
## [1] TRUE
```

```r
#> [1] TRUE
logical_abbr(FALSE)
```

```
## [1] FALSE
```

```r
#> [1] FALSE
```


```r
find_assign_rec <- function(x) {
  switch_expr(x,
    constant = ,
    symbol = character()
  )
}
find_assign <- function(x) find_assign_rec(enexpr(x))

find_assign("x")
```

```
## character(0)
```

```r
#> character(0)
find_assign(x)
```

```
## character(0)
```

```r
#> character(0)
```


```r
flat_map_chr <- function(.x, .f, ...) {
  purrr::flatten_chr(purrr::map(.x, .f, ...))
}

flat_map_chr(letters[1:3], ~ rep(., sample(3, 1)))
```

```
## [1] "a" "a" "a" "b" "c" "c"
```

```r
#> [1] "a" "b" "b" "b" "c" "c" "c"
```


```r
find_assign_rec <- function(x) {
  switch_expr(x,
    # Base cases
    constant = ,
    symbol = character(),

    # Recursive cases
    pairlist = flat_map_chr(as.list(x), find_assign_rec),
    call = {
      if (is_call(x, "<-")) {
        as_string(x[[2]])
      } else {
        flat_map_chr(as.list(x), find_assign_rec)
      }
    }
  )
}

find_assign(a <- 1)
```

```
## [1] "a"
```

```r
#> [1] "a"
find_assign({
  a <- 1
  {
    b <- 2
  }
})
```

```
## [1] "a" "b"
```

```r
#> [1] "a" "b"
```


```r
find_assign_call <- function(x) {
  if (is_call(x, "<-") && is_symbol(x[[2]])) {
    lhs <- as_string(x[[2]])
    children <- as.list(x)[-1]
  } else {
    lhs <- character()
    children <- as.list(x)
  }

  c(lhs, flat_map_chr(children, find_assign_rec))
}

find_assign_rec <- function(x) {
  switch_expr(x,
    # Base cases
    constant = ,
    symbol = character(),

    # Recursive cases
    pairlist = flat_map_chr(x, find_assign_rec),
    call = find_assign_call(x)
  )
}

find_assign(a <- b <- c <- 1)
```

```
## [1] "a" "b" "c"
```

```r
#> [1] "a" "b" "c"
find_assign(system.time(x <- print(y <- 5)))
```

```
## [1] "x" "y"
```

```r
#> [1] "x" "y"
```

#18.5 Exercises

1. logical_abbr() returns TRUE for T(1, 2, 3). How could you modify logical_abbr_rec() so that it ignores function calls that use T or F?

```r
T_call <- function(x) {
  if (is_call(x, "T") | is_call(x, "F")) { # check if T or F are used as function calls
    x <- as.list(x)[-1]
    purrr::some(x, logical_abbr_rec)
  } else { # treat same as pairlist
    purrr::some(x, logical_abbr_rec)
  }
}

logical_abbr_rec <- function(x) {
  switch_expr(x,
    # Base cases
    constant = FALSE,
    symbol = as_string(x) %in% c("F", "T"),

    # Recursive cases
    call = T_call(x),
    pairlist = purrr::some(x, logical_abbr_rec)
  )
}

logical_abbr(T(1, 2, 3))
```

```
## [1] FALSE
```

```r
logical_abbr(T)
```

```
## [1] TRUE
```

```r
logical_abbr(T + 2)
```

```
## [1] TRUE
```
Answer: If it is a call, remove pairlist to recurse over function body

2. logical_abbr() works with expressions. It currently fails when you give it a function. Why? How could you modify logical_abbr() to make it work? What components of a function will you need to recurse over?

```r
logical_abbr(function(x = TRUE) {
  g(x + T)
})
```

```
## [1] TRUE
```

```r
logical_abbr <- function(x) {
  logical_abbr_rec(enexpr(x))
}
```
Answer: If it is a call, remove pairlist to recurse over function body


```r
f <- quote(function(x=3, y=5) z=x+y)

lobstr::ast(!!f)
```

```
## █─`function` 
## ├─█─x = 3 
## │ └─y = 5 
## ├─█─`=` 
## │ ├─z 
## │ └─█─`+` 
## │   ├─x 
## │   └─y 
## └─<inline srcref>
```

```r
lobstr::ast(function(x=3, y=5) z=x+y)
```

```
## █─`function` 
## ├─█─x = 3 
## │ └─y = 5 
## ├─█─`=` 
## │ ├─z 
## │ └─█─`+` 
## │   ├─x 
## │   └─y 
## └─<inline srcref>
```


3. Modify find_assign to also detect assignment using replacement functions, i.e. names(x) <- y.

```r
find_assign_rec <- function(x) {
  switch_expr(x,
    # Base cases
    constant = ,
    symbol = character(),

    # Recursive cases
    pairlist = flat_map_chr(x, find_assign_rec),
    call = {
      if (is_call(x, "<-")) {
        if (is_call(x[[2]])) {
            as_string(x[[3]])
          } else {
            as_string(x[[2]])
          }
      } else {
        flat_map_chr(as.list(x), find_assign_rec)
      }
    }  
  )
}
```


4. Write a function that extracts all calls to a specified function.

```r
find_assign_call <- function(x) {
  if (is_call(x, "<-") && is_symbol(x[[2]])) {
    lhs <- as_string(x[[2]])
    children <- as.list(x)[-1]
  } else {
    lhs <- character()
    children <- as.list(x)
  }

  c(lhs, flat_map_chr(children, find_assign_rec))
}
```


