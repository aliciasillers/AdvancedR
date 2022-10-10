---
title: "chapter 6 pt 1"
author: "Alicia Sillers"
date: "2022-10-03"
output:
  html_document:
    keep_md: yes
---



# Quiz

1. What are the three components of a function?
My Answer: Function name, placeholder variable which represents what you read into the function, and the body of the function which codes for the actions you want it to carry out.     
Textbook Answer: The three components of a function are its body, arguments, and environment

2. What does the following code return?

```r
x <- 10
f1 <- function(x) {
  function() {
    x + 10
  }
}
f1(1)()
```

```
## [1] 11
```
My Answer: 11, because x is overwritten as 1.     
Textbook Answer: f1(1)() returns 11.

3. How would you usually write this code?

```r
`+`(1, `*`(2, 3))
```

```
## [1] 7
```
My Answer: 2 * 3 + 1    
Textbook Answer: You’d normally write it in infix style: 1 + (2 * 3) 

4. How could you make this call easier to read?

```r
mean(, TRUE, x = c(1:10, NA))
```

```
## [1] 5.5
```
My Answer: mean(c(1:10, NA))    
Textbook Answer: Rewriting the call to mean(c(1:10, NA), na.rm = TRUE) is easier to understand 

5. Does the following code throw an error when executed? Why or why not?

```r
f2 <- function(a, b) {
  a * 10
}
f2(10, stop("This is an error!"))
```

```
## [1] 100
```
My Answer: No, only a is part of the body of the function, so the error message of the b input is not printed.    
Textbook Answer: No, it does not throw an error because the second argument is never used so it’s never evaluated. 

6. What is an infix function? How do you write it? What’s a replacement function? How do you write it?
My Answer: I don't know the different types of functions.     
Textbook Answer: See Sections 6.8.3 and 6.8.4.

7. How do you ensure that cleanup action occurs regardless of how a function exits?
My Answer: I don't know     
Textbook Answer: You use on.exit() 

# 6.2.5 Exercises

1. Given a name, like "mean", match.fun() lets you find a function. Given a function, can you find its name? Why doesn’t that make sense in R?
Answer: No because a function is an object, so it can have 0, 1, or multiple names that point to it, but these names are not a part of what the function is, they are just a way to call it. 

2. It’s possible (although typically not useful) to call an anonymous function. Which of the two approaches below is correct? Why?

```r
function(x) 3()
```

```
## function(x) 3()
```

```r
#> function(x) 3()
(function(x) 3)()
```

```
## [1] 3
```

```r
#> [1] 3
```
Answer: I think the second one is correct because I think with an anonymous function, the body of the function has to be in parentheses with the call.

3. A good rule of thumb is that an anonymous function should fit on one line and shouldn’t need to use {}. Review your code. Where could you have used an anonymous function instead of a named function? Where should you have used a named function instead of an anonymous function?
Answer: Assuming this is referring to code in this chapter of the book, some named functions that could be anonymous include f02, the first function in 6.2.1;and f01, the first function in 6.2.3. All of the anonymous functions in the chapter work as anonymous functions, but the second two might be easier to follow if they were named and didn't have to be on only one line. 

4. What function allows you to tell if an object is a function? What function allows you to tell if a function is a primitive function?
Answer: is.function() allows you to tell if an object is a function. is.primitive allows you to tell if an object is a primitive function.

5. This code makes a list of all functions in the base package.

```r
objs <- mget(ls("package:base", all = TRUE), inherits = TRUE)
funs <- Filter(is.function, objs)
```

```r
fun.args <- sapply(funs, function(x) length(formals(x))) 

names(funs)[which.max(fun.args)] 
```

```
## [1] "scan"
```

```r
length(names(funs)[fun.args==0])
```

```
## [1] 253
```

Use it to answer the following questions:   

a) Which base function has the most arguments?
Answer: scan

b) How many base functions have no arguments? What’s special about those functions?
Answer: 253

c) How could you adapt the code to find all primitive functions?
Answer: funs <- Filter(is.primitive, objs)

6. What are the three important components of a function?
Answer: arguments, body, and environment. 

7. When does printing a function not show the environment it was created in?
Answer: It does not show the environment when the function is defined in the global environment. 

# 6.4.5 Exercises

1. What does the following code return? Why? Describe how each of the three c’s is interpreted.

```r
c <- 10
c(c = c)
```

```
##  c 
## 10
```
Answer: The code returns c 10. I think it prints the name and the value, rather than c and 10 being two different values of c, but I'm not entirely sure. 

2. What are the four principles that govern how R looks for values?
Answer: The four primary rules of lexical scoping in R are name masking, which means that R searches for the name inside of the funciton before looking outside; functions vs variables, which means that when you use a name in a function call, R ignores non-function objects when searching; fresh start, which means running a function multiple times will not change the outcome of later runs based on early runs as in a loop; and dynamic lookup, which means that R looks for values when the function is run, not when it is created. 

3. What does the following function return? Make a prediction before running the code yourself.

```r
f <- function(x) {
  f <- function(x) {
    f <- function() {
      x ^ 2
    }
    f() + 1
  }
  f(x) * 2
}
f(10)
```

```
## [1] 202
```
Answer: My prediction is that it will return the value 202. This prediction was correct. 

# 6.5.4

1. What important property of && makes x_ok() work?

```r
x_ok <- function(x) {
  !is.null(x) && length(x) == 1 && x > 0
}

x_ok(NULL)
```

```
## [1] FALSE
```

```r
#> [1] FALSE
x_ok(1)
```

```
## [1] TRUE
```

```r
#> [1] TRUE
x_ok(1:3)
```

```
## [1] FALSE
```

```r
#> [1] FALSE
```
Answer: && makes it so that all of the conditions are examined as one statement, so everything has to be true for the function to return true.      

What is different with this code? Why is this behaviour undesirable here?

```r
x_ok <- function(x) {
  !is.null(x) & length(x) == 1 & x > 0
}

x_ok(NULL)
```

```
## logical(0)
```

```r
#> logical(0)
x_ok(1)
```

```
## [1] TRUE
```

```r
#> [1] TRUE
x_ok(1:3)
```

```
## [1] FALSE FALSE FALSE
```

```r
#> [1] FALSE FALSE FALSE
```
Answer: This code uses & instead of &&, which behaves a bit differently. For example, the output matches the length of the input. This makes evaluation of whether input is null complicated because an input of null results in zero-length output.

2. What does this function return? Why? Which principle does it illustrate?

```r
f2 <- function(x = z) {
  z <- 100
  x
}
f2()
```

```
## [1] 100
```
Answer: It returns 100 because x is not specified when the function is run so it takes on the internally specified value of being set equal to z. This illustrates the principle of default arguments.

3. What does this function return? Why? Which principle does it illustrate?

```r
y <- 10
f1 <- function(x = {y <- 1; 2}, y = 0) {
  c(x, y)
}
f1()
```

```
## [1] 2 1
```

```r
y
```

```
## [1] 10
```
Answer: It returns 2 1 on the first line and 10 on the second line. I don't totally understand why x and y have the values they do in f1(), but f1() is printing these values based on the default arguments, while calling y afterwards still returns the value of y before the function was created and run. This illustrates the principle of promises that values given in function do not change the value of the object in the outside environment. It seems like this also relates to the fresh start principle from the last section. 

4. In hist(), the default value of xlim is range(breaks), the default value for breaks is "Sturges", and

```r
range("Sturges")
```

```
## [1] "Sturges" "Sturges"
```

```r
#> [1] "Sturges" "Sturges"
```
Explain how hist() works to get a correct xlim value.   
Answer: xlim is the range of x values with sensible defaults. breaks determines where there are breakpoints between histogram cells. The sturges formula bases bin sizes on the range of the data. By nesting these arguments, everything is based on the same formula to know where the data is broken up, then how much data is in each cell and the range of values, then what xlim is needed to contain that range. 

5. Explain why this function works. Why is it confusing?

```r
show_time <- function(x = stop("Error!")) {
  stop <- function(...) Sys.time()
  print(x)
}
show_time()
```

```
## [1] "2022-10-09 22:45:46 MST"
```

```r
#> [1] "2021-02-21 19:22:36 UTC"
```
Answer: Within the function, stop is defined as giving the system time and then x is printed. The argument sets x equal to stop. I don't understand the error message part though. I think it is also confusing because x is set equal to stop in the argument but stop is not defined until the body of the function. 

6. How many arguments are required when calling library()?
Answer: 9 arguments are required. 

