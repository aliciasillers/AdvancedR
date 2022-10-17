---
title: "chapter 6 pt 2"
author: "Alicia Sillers"
date: "2022-10-10"
output: 
  html_document:
      keep_md: yes 
---



# 6.6.1 Exercises

1. Explain the following results:

```r
sum(1, 2, 3)
```

```
## [1] 6
```

```r
#> [1] 6
mean(1, 2, 3)
```

```
## [1] 1
```

```r
#> [1] 1

sum(1, 2, 3, na.omit = TRUE)
```

```
## [1] 7
```

```r
#> [1] 7
mean(1, 2, 3, na.omit = TRUE)
```

```
## [1] 1
```

```r
#> [1] 1
```
Answer: sum() returns the sum of all values present in its arguments. In the first line, these values are 1, 2, and 3, which it adds to give a sum of 6. The second sum also has an argument "na.omit = TRUE", which sum interprets as 1, making the sum 7. mean() works differently. mean() has three defined arguments: x, trim, and na.rm. The argument of which it actually computes the mean is x, so in the example, it is returning a mean of 1 both times for the x argument 1. 

2. Explain how to find the documentation for the named arguments in the following function call:

```r
plot(1:10, col = "red", pch = 20, xlab = "x", col.lab = "blue")
```

![](chapter-6-pt-2_files/figure-html/unnamed-chunk-2-1.png)<!-- -->
Answer: You could use help(plot.default) or ??plot.default and read the section of the plot documentation about the arguments. You could also try using ?? for the arguments themselves, but some might not have their own documentation.

3. Why does plot(1:10, col = "red") only colour the points, not the axes or labels? Read the source code of plot.default() to find out.     
Answer: The col argument only colors points and lines. Label colors have their own argument, col.lab. 

# 6.7.5 Exercises

1. What does load() return? Why don’t you normally see these values?    
Answer: load() returns a character vector of the names of objects created but does so invisibly, so you don't normally see these values. It would return something visibly if there was a problem with the input, in which case it would return a warning or error. 

2. What does write.table() return? What would be more useful?   
Answer: write.table() prints the x argument to a file and makes sure it is in the format of a data frame or matrix. It might be more useful in some cases to return the x argument as a data frame immediately. 

3. How does the chdir parameter of source() compare to with_dir()? Why might you prefer one to the other?    
Answer: They both temporarily change the working directory for evaluation of some code. chdir is a parameter of source() while with_dir() is a command from the withr package. I don't see any major difference between how they work except that it seems like with_dir() might be a bit more flexible as to what code can be evaluated. 

4. Write a function that opens a graphics device, runs the supplied code, and closes the graphics device (always, regardless of whether or not the plotting code works).    
Answer:

```r
runcode <- function() {
  on.exit(dev.off(), add = TRUE)
  pdf()
}
```

5. We can use on.exit() to implement a simple version of capture.output().

```r
capture.output2 <- function(code) {
  temp <- tempfile()
  on.exit(file.remove(temp), add = TRUE, after = TRUE)

  sink(temp)
  on.exit(sink(), add = TRUE, after = TRUE)

  force(code)
  readLines(temp)
}
capture.output2(cat("a", "b", "c", sep = "\n"))
#> [1] "a" "b" "c"
```
Compare capture.output() to capture.output2(). How do the functions differ? What features have I removed to make the key ideas easier to see? How have I rewritten the key ideas so they’re easier to understand?    
Answer: capture.output2() creates a temporary file, writes input into the file, prints the file, and then removes the file, whereas the original would create a file to write the input into or print the info by specifying the file as null. The second one removes the option of having a permanent file, which also removes the need for ammend, which is used in the original to avoid accidentally overwriting an existing file. 

# 6.8.6 Exercises

1. Rewrite the following code snippets into prefix form:

```r
1 + 2 + 3

1 + (2 + 3)

if (length(x) <= 5) x[[5]] else x[[n]]
```
Answer:

```r
`+`(`+`(1, 2), 3)
`+`(`+`(2, 3), 1)
`if`(length(x) <= 5, `[[`(x, 5), `[[`(x, n))
```

2. Clarify the following list of odd function calls:

```r
x <- sample(replace = TRUE, 20, x = c(1:10, NA))
y <- runif(min = 0, max = 1, 20)
cor(m = "k", y = y, u = "p", x = x)
```

```
## [1] 0.3028081
```
Answer: The first one creates a list of 20 random numbers of any value from 1 to 10, the second one does the same thing but of any value from o to 1 and they must be a normal distribution, and the last one computes the covariance or correlation of x and y, but i am unsure what the m and u arguments are for. 

3. Explain why the following code fails:

```r
#modify(get("x"), 1) <- 10
#> Error: target of assignment expands to non-language object
```
Answer: I think the code fails because x is in quotation makes, so it is being read as a string rather than an object.

4. Create a replacement function that modifies a random location in a vector.   
Answer:

```r
`locmod<-` <- function(x, value){
  i <- sample(x, length(x), replace=TRUE) [1]
  x[i] <- value
  x
}

x <- 1:10
locmod(x) <- 100
x
```

```
##  [1]   1   2   3   4   5   6   7   8   9 100
```

5. Write your own version of + that pastes its inputs together if they are character vectors but behaves as usual otherwise. In other words, make this code work:

```r
1 + 2
```

```
## [1] 3
```

```r
#> [1] 3

#"a" + "b"
#> [1] "ab"
```
Answer:

```r
`+` <- function(a, b) {
    if (is.character(a) | is.character(b)) {
        return(c(a, b))
    }
    else {
        return(sum(a, b))
    }
}

"a" + "b"
```

```
## [1] "a" "b"
```

```r
1 + 2
```

```
## [1] 3
```

6. Create a list of all the replacement functions found in the base package. Which ones are primitive functions? (Hint: use apropos().)    
Answer:

```r
repfuns <- list(apropos("<-$", mode = "function"))
primreps <- sapply(repfuns, is.primitive)
repfuns
```

```
## [[1]]
##  [1] "$<-"              ".rowNamesDF<-"    "@<-"              "[[<-"            
##  [5] "[<-"              "<-"               "<<-"              "as<-"            
##  [9] "attr<-"           "attributes<-"     "body<-"           "body<-"          
## [13] "class<-"          "coerce<-"         "colnames<-"       "comment<-"       
## [17] "contrasts<-"      "diag<-"           "dim<-"            "dimnames<-"      
## [21] "el<-"             "elNamed<-"        "Encoding<-"       "environment<-"   
## [25] "formals<-"        "functionBody<-"   "is.na<-"          "languageEl<-"    
## [29] "length<-"         "levels<-"         "locmod<-"         "mode<-"          
## [33] "mostattributes<-" "names<-"          "oldClass<-"       "packageSlot<-"   
## [37] "parent.env<-"     "regmatches<-"     "row.names<-"      "rownames<-"      
## [41] "S3Class<-"        "S3Part<-"         "slot<-"           "split<-"         
## [45] "storage.mode<-"   "substr<-"         "substring<-"      "tsp<-"           
## [49] "units<-"          "window<-"
```

```r
primreps
```

```
## [1] FALSE
```


7. What are valid names for user-created infix functions?   
Answer: Any name that starts and ends with %

8. Create an infix xor() operator.    
Answer:

```r
`%xor%` <- function(x, y){
    (x | y) & !(x & y)
}
```


9. Create infix versions of the set functions intersect(), union(), and setdiff(). You might call them %n%, %u%, and %/% to match conventions from mathematics.    
Answer:

```r
`%n%` <- function(x, y){instersect(x, y)}
  
`%u%` <- function(x, y){union(x, y)}
  
`%/%` <- function(x, y){setdiff(x, y)}
```

