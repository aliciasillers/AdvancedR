---
title: "chapter 3"
author: "Alicia Sillers"
date: "2022-09-09"
output: 
  html_document: 
    keep_md: yes 
---



# Quiz

1. What are the four common types of atomic vectors? What are the two rare types?
My Answer: I'm not sure what this means. 
Textbook Answer: The four common types of atomic vectors are logical, integer, double and character. The two rarer types are complex and raw.

2. What are attributes? How do you get them and set them?
My Answer: Attributes are additional information associated with objects. 
Textbook Answer: Attributes allow you to associate arbitrary additional metadata to any object. You can get and set individual attributes with attr(x, "y") and attr(x, "y") <- value; or you can get and set all attributes at once with attributes().

3. How is a list different from an atomic vector? How is a matrix different from a data frame?
My Answer: Lists and data frames reference data and can have multiple data types. Atomic vectors and matrices must have only one data type, and I think they might hold data rather than referencing data. 
Textbook Answer: The elements of a list can be any type (even a list); the elements of an atomic vector are all of the same type. Similarly, every element of a matrix must be the same type; in a data frame, different columns can have different types.

4. Can you have a list that is a matrix? Can a data frame have a column that is a matrix?
My Answer: I think yes for both.
Textbook Answer: You can make a list-array by assigning dimensions to a list. You can make a matrix a column of a data frame with df$x <- matrix(), or by using I() when creating a new data frame data.frame(x = I(matrix())).

5. How do tibbles behave differently from data frames?
My Answer: I don't know what a tibble is.
Textbook Answer: Tibbles have an enhanced print method, never coerce strings to factors, and provide stricter subsetting methods.

# 3.2.5 Exercises

1. How do you create raw and complex scalars?
Answer: raw() creates a raw vector of a specified length. Complex vectors can be created with complex(). The complex vector can be specified either by giving its length, its real and imaginary parts, or modulus and argument.

2. Test your knowledge of the vector coercion rules by predicting the output of the following uses of c():

```r
c(1, FALSE)
```

```
## [1] 1 0
```

```r
c("a", 1)
```

```
## [1] "a" "1"
```

```r
c(TRUE, 1L)
```

```
## [1] 1 1
```
Answer: My prediction is that the first line's output will be (1, 0), the second line's output will be ("a", "1"), and the third line's output will be (1L, 1L).

3. Why is 1 == "1" true? Why is -1 < FALSE true? Why is "one" < 2 false?
Answer: The first one is true because 1 will be coerced to a character, so both will be the same string. The second is true because FALSE will be coerced to a double, and -1 is less than 0. The third one is false because 2 will be coerced to a character and the strings will not be the same.

4. Why is the default missing value, NA, a logical vector? What’s special about logical vectors? (Hint: think about c(FALSE, NA_character_).)
Answer: I don't know how to explain this that well, but NA is a value that is non-numeric and has a specific meaning in R rather than just being any random word that would have meaning in a data set but not in R in general.

5. Precisely what do is.atomic(), is.numeric(), and is.vector() test for?
Answer: is.atomic() tests whether something is one of the six atomic types or NULL. is.numeric() tests whether an object is interpretable as numbers. is.vector() tests whether something is a vector of the specified mode having no attributes other than names.

# 3.3.4 Exercises

1. How is setNames() implemented? How is unname() implemented? Read the source code.
Answer: setNames() sets the names on an object and returns the object: setNames(object = nm, nm). unname() removes names or dimnames attribute of an R object: unname(obj, force = FALSE).

2. What does dim() return when applied to a 1-dimensional vector? When might you use NROW() or NCOL()?
Answer: If a vector has no dimensions, dim() would return NULL, but if it is set to have one dimension, dim() would return the length of the vector. NROW() and NCOL() would be used to set or determine how many rows and columns are in a matrix. 

3. How would you describe the following three objects? What makes them different from 1:5?

```r
x1 <- array(1:5, c(1, 1, 5))
x2 <- array(1:5, c(1, 5, 1))
x3 <- array(1:5, c(5, 1, 1))
```
Answer: On it's own, 1:5 has no dimensions. x1 references 1:5 in a row of numbers. x2 references 1:5 in a column of numbers. x3 references 1:5 with each number in separate groups of one row and one column.

4. An early draft used this code to illustrate structure():

```r
structure(1:5, comment = "my attribute")
```

```
## [1] 1 2 3 4 5
```

```r
#> [1] 1 2 3 4 5
```
But when you print that object you don’t see the comment attribute. Why? Is the attribute missing, or is there something else special about it? (Hint: try using help.)
Answer: The attribute is not missing. Comment attributes are not printed.

# 3.4.5 Exercises

1. What sort of object does table() return? What is its type? What attributes does it have? How does the dimensionality change as you tabulate more variables?
Answer: table() returns objects interpreted as factors, which are the integer data type. It's attributes are the class, levels, and names. I don't know about the dimensionality question.

2. What happens to a factor when you modify its levels?

```r
f1 <- factor(letters)
levels(f1) <- rev(levels(f1))
```
Answer: At first I thought the data would stay the same and only the levels would change, but printing f1 before and after shows both being reversed when the levels are reversed. 

3. What does this code do? How do f2 and f3 differ from f1?

```r
f2 <- rev(factor(letters))

f3 <- factor(letters, levels = rev(letters))
```
Answer: The code creates two factor vectors called f2 and f3. f2 contains the alphabet reversed with the levels as the alphabet not reversed. f3 contains the alphabet but the levels are the alphabet in reverse. These are both different from f1 in which the data and levels are in the same order. 

# 3.5.4 Exercises

1. List all the ways that a list differs from an atomic vector.
Answer: Each element in a list can be any type, whereas atomic vectors must have all the same type. Lists reference data rather than containing it. Lists are recursive because they can contain lists. 

2. Why do you need to use unlist() to convert a list to an atomic vector? Why doesn’t as.vector() work?
Answer: I think it might be that as.vector() doesn't account for recursive structure in lists, whereas unlist() does.

3. Compare and contrast c() and unlist() when combining a date and date-time into a single vector.
Answer: I think both methods would remove the attributes from the data. unlist() would result in a vector that is no longer in a list format, whereas c() could result in a list. 

# 3.6.8 Exercises

1. Can you have a data frame with zero rows? What about zero columns?
Answer: Yes for both. If it has zero of one is also must have zero of the other. This would just be an empty data frame. 

2. What happens if you attempt to set rownames that are not unique?
Answer: I think they would be adjusted by make.names to make them unique. 

3. If df is a data frame, what can you say about t(df), and t(t(df))? Perform some experiments, making sure to try different column types.
Answer: I think df would be coerced to a matrix before being transposed. 

4. What does as.matrix() do when applied to a data frame with columns of different types? How does it differ from data.matrix()?
Answer: as.matrix() would return a character matrix if all data is atomic. Other data would be coerced using the coercion hierarchy. data.matrix() is different because instead of creating a character matrix, it creates a numeric matrix. 
