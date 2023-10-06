---
title: "chapter-25"
author: "Alicia Sillers"
date: "2023-10-06"
output: 
  html_document:
    keep_md: TRUE
---




```r
library(Rcpp)
```

```
## Warning: package 'Rcpp' was built under R version 4.2.3
```

# 25.1 Notes

Rcpp package allows rewriting of functions in C++     

Typical bottlenecks that C++ can address include:   

1. Loops that can’t be easily vectorised because subsequent iterations depend on previous ones.    

2. Recursive functions, or problems which involve calling functions millions of times. The overhead of calling a function in C++ is much lower than in R.    

3. Problems that require advanced data structures and algorithms that R doesn’t provide. Through the standard template library (STL), C++ has efficient implementations of many important data structures, from ordered maps to double-ended queues.   

# 25.2 Notes

cppFunction() allows you to write C++ functions in R:

```r
cppFunction('int add(int x, int y, int z) {
  int sum = x + y + z;
  return sum;
}')
# add works like a regular R function
add
#> function (x, y, z) 
#> .Call(<pointer: 0x107536a00>, x, y, z)
add(1, 2, 3)
#> [1] 6
```

Example with no input, scalar output: return integer one

```r
#R version

one <- function() 1L

#C++ equivalent

cppFunction('int one() {
  return 1;
}')
```

This small function illustrates a number of important differences between R and C++:

1. The syntax to create a function looks like the syntax to call a function; you don’t use assignment to create functions as you do in R.

2. You must declare the type of output the function returns. This function returns an int (a scalar integer). The classes for the most common types of R vectors are: NumericVector, IntegerVector, CharacterVector, and LogicalVector.

3. Scalars and vectors are different. The scalar equivalents of numeric, integer, character, and logical vectors are: double, int, String, and bool.

4. You must use an explicit return statement to return a value from a function.

5. Every statement is terminated by a ;   

Example with scalar input and scalar output: returns 1 or -1 depending on whether input is positive or negative

```r
signR <- function(x) {
  if (x > 0) {
    1
  } else if (x == 0) {
    0
  } else {
    -1
  }
}

cppFunction('int signC(int x) {
  if (x > 0) {
    return 1;
  } else if (x == 0) {
    return 0;
  } else {
    return -1;
  }
}')
```

In the C++ version:

1. We declare the type of each input in the same way we declare the type of the output. While this makes the code a little more verbose, it also makes clear the type of input the function needs.

2. The if syntax is identical — while there are some big differences between R and C++, there are also lots of similarities! C++ also has a while statement that works the same way as R’s. As in R you can use break to exit the loop, but to skip one iteration you need to use continue instead of next.   

Example with vector input, scalar output: sum for loop

```r
sumR <- function(x) {
  total <- 0
  for (i in seq_along(x)) {
    total <- total + x[i]
  }
  total
}

cppFunction('double sumC(NumericVector x) {
  int n = x.size();
  double total = 0;
  for(int i = 0; i < n; ++i) {
    total += x[i];
  }
  return total;
}')
```

The C++ version is similar, but:

1. To find the length of the vector, we use the .size() method, which returns an integer. C++ methods are called with . (i.e., a full stop).

2. The for statement has a different syntax: for(init; check; increment). This loop is initialised by creating a new variable called i with value 0. Before each iteration we check that i < n, and terminate the loop if it’s not. After each iteration, we increment the value of i by one, using the special prefix operator ++ which increases the value of i by 1.

3. In C++, vector indices start at 0, which means that the last element is at position n - 1. I’ll say this again because it’s so important: IN C++, VECTOR INDICES START AT 0! This is a very common source of bugs when converting R functions to C++.

4. Use = for assignment, not <-.

5. C++ provides operators that modify in-place: total += x[i] is equivalent to total = total + x[i]. Similar in-place operators are -=, *=, and /=.    

Example with vector input, vector output: comput distance between values

```r
pdistR <- function(x, ys) {
  sqrt((x - ys) ^ 2)
}

cppFunction('NumericVector pdistC(double x, NumericVector ys) {
  int n = ys.size();
  NumericVector out(n);

  for(int i = 0; i < n; ++i) {
    out[i] = sqrt(pow(ys[i] - x, 2.0));
  }
  return out;
}')
```

This function introduces only a few new concepts:

1. We create a new numeric vector of length n with a constructor: NumericVector out(n). Another useful way of making a vector is to copy an existing one: NumericVector zs = clone(ys).

2. C++ uses pow(), not ^, for exponentiation.   

sourceCpp() allows use of C++ files   

Your stand-alone C++ file should have extension .cpp, and needs to start with:    
#include <Rcpp.h>   
using namespace Rcpp;   
And for each function that you want available within R, you need to prefix it with:   
// [[Rcpp::export]]   
To compile the C++ code, use sourceCpp("path/to/file.cpp")

# 25.2 Exercises

1. With the basics of C++ in hand, it’s now a great time to practice by reading and writing some simple C++ functions. For each of the following functions, read the code and figure out what the corresponding base R function is. You might not understand every part of the code yet, but you should be able to figure out the basics of what the function does.

```r
double f1(NumericVector x) {
  int n = x.size();
  double y = 0;

  for(int i = 0; i < n; ++i) {
    y += x[i] / n;
  }
  return y;
} #answer: mean. the function is adding everything in the vector and dividing by the length of the vector

NumericVector f2(NumericVector x) {
  int n = x.size();
  NumericVector out(n);

  out[0] = x[0];
  for(int i = 1; i < n; ++i) {
    out[i] = out[i - 1] + x[i];
  }
  return out;
} #answer: cumsum. creates a new function in which each value is the sum of the value in the input vector and all the values before it in the input vector

bool f3(LogicalVector x) {
  int n = x.size();

  for(int i = 0; i < n; ++i) {
    if (x[i]) return true;
  }
  return false;
} #answer: any. the if statement returns true if a value in the logical vector is true

int f4(Function pred, List x) { #function and list input
  int n = x.size();

  for(int i = 0; i < n; ++i) {
    LogicalVector res = pred(x[i]); #applies function to each value in list
    if (res[0]) return i + 1; #if result of function is 0, return position in list + 1
  }
  return 0;
} #answer: not sure what the function is, but i wrote what i think is happening

NumericVector f5(NumericVector x, NumericVector y) {
  int n = std::max(x.size(), y.size());
  NumericVector x1 = rep_len(x, n);
  NumericVector y1 = rep_len(y, n);

  NumericVector out(n);

  for (int i = 0; i < n; ++i) {
    out[i] = std::min(x1[i], y1[i]);
  }

  return out;
} #answer: pmin
```
Answers in comments

2. To practice your function writing skills, convert the following functions into C++. For now, assume the inputs have no missing values.    

all().

```r
cppFunction('bool all2(LogicalVector x) {
  int n = x.size();
  for (int i = 0; i < n; ++i) {
    if (!x[i]) return false;
  }
  return true;
}')
```

cumprod(), cummin(), cummax().

```r
cppFunction('NumericVector cumprod2(NumericVector x) {
  int n = x.size();
  NumericVector out(n);

  out[0] = x[0];
  for(int i = 1; i < n; ++i) {
    out[i] = out[i - 1] * x[i];
  }
  return out;
}')

cppFunction('NumericVector cummin2(NumericVector x) {
  int n = x.size();
  NumericVector out(n);

  out[0] = x[0];
  for(int i = 1; i < n; ++i) {
    out[i]  = std::min(out[i - 1], x[i]);
  }
  return out;
}')
```

diff(). Start by assuming lag 1, and then generalise for lag n.

```r
cppFunction('NumericVector diff2(NumericVector x) {
  int n = x.size();
  NumericVector out(n - 1);

  for (int i = 1; i < n; i++) {
    out[i - 1] = x[i] - x[i - 1];
  }
  return out ;
}')
```

range().

```r
cppFunction('NumericVector range2(NumericVector x) {
  double omin = x[0], omax = x[0];
  int n = x.size();

  if (n == 0) stop("`length(x)` must be greater than 0.");

  for (int i = 1; i < n; i++) {
    omin = std::min(x[i], omin);
    omax = std::max(x[i], omax);
  }

  NumericVector out(2);
  out[0] = omin;
  out[1] = omax;
  return out;
}')
```

var(). Read about the approaches you can take on Wikipedia. Whenever implementing a numerical algorithm, it’s always good to check what is already known about the problem.

```r
cppFunction('double var2(NumericVector x) {
  int n = x.size();

  if (n < 2) {
    return NA_REAL;
  }

  double mx = 0;
  for (int i = 0; i < n; ++i) {
    mx += x[i] / n;
  }

  double out = 0;
  for (int i = 0; i < n; ++i) {
    out += pow(x[i] - mx, 2);
  }

  return out / (n - 1);
}')
```

