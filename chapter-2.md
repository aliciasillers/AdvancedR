---
title: "Chapter 2"
author: "Alicia Sillers"
date: "2022-09-02"
output: 
  html_document: 
    keep_md: yes
---



```r
library(lobstr)
```

# Quiz

1. Given the following data frame, how do I create a new column called “3” that contains the sum of 1 and 2? You may only use $, not [[. What makes 1, 2, and 3 challenging as variable names?

```r
df <- data.frame(runif(3), runif(3))
names(df) <- c(1, 2)
```

```r
df$'3' <- c(df$'1' + df$'2')
df
```

```
##            1         2         3
## 1 0.07307315 0.7889253 0.8619984
## 2 0.57356074 0.5526828 1.1262435
## 3 0.53419376 0.5024706 1.0366643
```
My answer: 1, 2, and 3 are challenging variable names because R reads them as numerical values rather than names. To address this, they need to be in quotation marks.
Answer from book: you must quote non-syntactic names with backticks

2. In the following code, how much memory does y occupy?

```r
x <- runif(1e6)
y <- list(x, x, x)
```
My answer: I'm not sure how to answer this question besides the fact that 'y' contains three values. 
Answer from book: 'y' occupies 8 Mb of memory, which is discovered using the obj_size() command.

3. On which line does a get copied in the following example?

```r
a <- c(1, 5, 3, 2)
b <- a
b[[1]] <- 10
```
My answer: 'a' is copied on the second line of the above code. 
Answer from book: 'a' is copied when 'b' is modified, which is the third line of the code. 

# 2.2.2 Exercises

1. Explain the relationship between a, b, c and d in the following code:

```r
a <- 1:10
b <- a
c <- b
d <- 1:10
```
Answer: a, b, and c are three different names tied to the same object. d is a name tied to a different object that is identical to the first object.

2. The following code accesses the mean function in multiple ways. Do they all point to the same underlying function object? Verify this with lobstr::obj_addr().

```r
mean
```

```
## function (x, ...) 
## UseMethod("mean")
## <bytecode: 0x000002161c5a2e30>
## <environment: namespace:base>
```

```r
base::mean
```

```
## function (x, ...) 
## UseMethod("mean")
## <bytecode: 0x000002161c5a2e30>
## <environment: namespace:base>
```

```r
get("mean")
```

```
## function (x, ...) 
## UseMethod("mean")
## <bytecode: 0x000002161c5a2e30>
## <environment: namespace:base>
```

```r
evalq(mean)
```

```
## function (x, ...) 
## UseMethod("mean")
## <bytecode: 0x000002161c5a2e30>
## <environment: namespace:base>
```

```r
match.fun("mean")
```

```
## function (x, ...) 
## UseMethod("mean")
## <bytecode: 0x000002161c5a2e30>
## <environment: namespace:base>
```
Answer: Yes, they all point to the same underlying function object.

```r
obj_addr(mean)
```

```
## [1] "0x2161c5a2ed8"
```

```r
obj_addr(base::mean)
```

```
## [1] "0x2161c5a2ed8"
```

```r
obj_addr(get("mean"))
```

```
## [1] "0x2161c5a2ed8"
```

```r
obj_addr(evalq(mean))
```

```
## [1] "0x2161c5a2ed8"
```

```r
obj_addr(match.fun("mean"))
```

```
## [1] "0x2161c5a2ed8"
```
Answer pt 2: obj_addr() verified that they all point to the same underlying function object, because the same identifier was returned each time. 

3. By default, base R data import functions, like read.csv(), will automatically convert non-syntactic names to syntactic ones. Why might this be problematic? What option allows you to suppress this behaviour?
Answer: One way this might be problematic is if you need to move code back and forth between multiple programs. If the names are changed in R but point to objects that were not created in the code, different programs might not be able to find the objects. This can be suppressed with check.names = FALSE.

4. What rules does make.names() use to convert non-syntactic names into syntactic ones?
Answer: The character "X" is prepended if necessary. All invalid characters are translated to ".". A missing value is translated to "NA". Names which match R keywords have a dot appended to them.

5. Why is .123e1 not a syntactic name? Read ?make.names for the full details.
Answer: A syntactic name is not allowed to start with a "." followed by a number.

# 2.3.6 Exercises

1. Why is tracemem(1:10) not useful?
Answer: It is not useful because 1:10 is set of numbers and not a particular object, so it cannot be copied and there is no need to trace its copy history. 

2. Explain why tracemem() shows two copies when you run this code. Hint: carefully look at the difference between this code and the code shown earlier in the section.

```r
x <- c(1L, 2L, 3L)
tracemem(x)
```

```
## [1] "<000002161C81A958>"
```

```r
x[[3]] <- 4
```

```
## tracemem[0x000002161c81a958 -> 0x000002161c3f06a8]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> 
## tracemem[0x000002161c3f06a8 -> 0x000002161c110648]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous>
```
Answer: I think tracemem(x) shows that x was copied twice because two different data types are used (integer and numeric), so there needed to be an intermediate that converted the data type before changing one of the values. 

3. Sketch out the relationship between the following objects:

```r
a <- 1:10
b <- list(a, a)
c <- list(b, a, 1:10)
```

```r
knitr::include_graphics("2.3.6.3.png")
```

![](2.3.6.3.png)<!-- -->

4. What happens when you run this code? Draw a picture.

```r
x <- list(1:10)
x[[2]] <- x
```
Answer: The code first creates a list in which each part refers to a number 1-10 consecutively. The name x points to this list. Then the list is modified to make the second part of the list refer to the whole original list.

```r
knitr::include_graphics("2.3.6.4.png")
```

![](2.3.6.4.png)<!-- -->

# 2.4.1 Exercises

1. In the following example, why are object.size(y) and obj_size(y) so radically different? Consult the documentation of object.size().

```r
y <- rep(list(runif(1e4)), 100)

object.size(y)
```

```
## 8005648 bytes
```

```r
#> 8005648 bytes
obj_size(y)
```

```
## 80.90 kB
```

```r
#> 80,896 B
```
Answer: object.size() does not detect shared elements in a list, while obj_size() does. 

2. Take the following list. Why is its size somewhat misleading?

```r
funs <- list(mean, sd, var)
obj_size(funs)
```

```
## 17.55 kB
```

```r
#> 17,608 B
```
Answer: I'm not entirely sure, but I would guess that maybe since it is a list of calculations, each calculation is referencing the same set of numbers, which only has to be stored once, making the size smaller than expected. 

3. Predict the output of the following code:

```r
a <- runif(1e6)
obj_size(a)
```

```
## 8.00 MB
```

```r
b <- list(a, a)
obj_size(b)
```

```
## 8.00 MB
```

```r
obj_size(a, b)
```

```
## 8.00 MB
```

```r
b[[1]][[1]] <- 10
obj_size(b)
```

```
## 16.00 MB
```

```r
obj_size(a, b)
```

```
## 16.00 MB
```

```r
b[[2]][[1]] <- 10
obj_size(b)
```

```
## 16.00 MB
```

```r
obj_size(a, b)
```

```
## 24.00 MB
```
Answer: obj_size(a) will produce a number that I don't know how to predict without running the code. obj_size(b) and obj_size(a, b) will both be the size of a + the size of a list. The first modification will create a modified copy of a, which will increase obj_size(b). I think the second modification will alter the copy, which will not change the size of obj_size(b). These modifications will also cause obj_size(a, b) to become greater than obj_size(a) or obj_size(b).

# 2.5.3 Exercises

1. Explain why the following code doesn’t create a circular list.

```r
x <- list()
x[[1]] <- x
```
Answer: I think it is because the modification creates a new copy, so x will be a list containing an empty list rather than a list containing infinite nested lists. 

2. Wrap the two methods for subtracting medians into two functions, then use the ‘bench’ package to carefully compare their speeds. How does performance change as the number of columns increase?

```r
library(bench)
```

```r
x <- data.frame(matrix(runif(5 * 1e4), ncol = 5))
medians <- vapply(x, median, numeric(1))

y <- as.list(x)

fun1 <- function(a){
for (i in seq_along(medians)) {
  a[[i]] <- a[[i]] - medians[[i]]
  }
}

fun2 <- function(b){
  for (i in 1:5) {
  b[[i]] <- b[[i]] - medians[[i]]
  }
}
bench::system_time(fun1(x))
```

```
## process    real 
##  15.6ms  12.8ms
```

```r
bench::system_time(fun2(y))
```

```
## process    real 
## 15.62ms  6.51ms
```
Answer: I might have done it wrong, but for me, the function for subtracting medians from the data frame and the function for subtracting medians from the list have similar speeds, with which is faster changing each time I run the code. 

3. What happens if you attempt to use tracemem() on an environment?
Answer: Using tracemem() on an environment results in an error.
