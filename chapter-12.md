---
title: "chapter-12"
output: 
  html_document:
    keep_md: TRUE
date: "2023-02-27"
---



```r
library(sloop)
```

#12.2 Base vs Object-Oriented Object


```r
# A base object:
is.object(1:10)
```

```
## [1] FALSE
```

```r
# An OO object
is.object(mtcars)
```

```
## [1] TRUE
```

OO objects have a class attribute

```r
attr(1:10, "class")
```

```
## NULL
```

```r
attr(mtcars, "class")
```

```
## [1] "data.frame"
```

#12.3 Base Types

Every object has a base type, whether base or object-oriented

```r
typeof(1:10)
```

```
## [1] "integer"
```

```r
typeof(mtcars)
```

```
## [1] "list"
```

There are 25 base types:    
Vectors include types NULL (NILSXP), logical (LGLSXP), integer (INTSXP), double (REALSXP), complex (CPLXSXP), character (STRSXP), list (VECSXP), and raw (RAWSXP).    
Functions include types closure (regular R functions, CLOSXP), special (internal functions, SPECIALSXP), and builtin (primitive functions, BUILTINSXP).    
Environments have type environment (ENVSXP).    
The S4 type (S4SXP), Chapter 15, is used for S4 classes that don’t inherit from an existing base type.   
Language components include symbol (aka name, SYMSXP), language (usually called calls, LANGSXP), and pairlist (used for function arguments, LISTSXP) types.   
The remaining types are esoteric and rarely seen in R. They are important primarily for C code: externalptr (EXTPTRSXP), weakref (WEAKREFSXP), bytecode (BCODESXP), promise (PROMSXP), ... (DOTSXP), and any (ANYSXP).   

##12.3.1 Numeric Type

as.numeric() coerces to type double, is.numeric() tests for objects that behave as numbers (excludes factors), numeric in S3 and S4 system can be either integer or double
