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

#18.3 Exercises

1. 

2. 

3. 

4. 

5. 

6.
