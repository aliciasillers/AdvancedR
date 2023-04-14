---
title: "chapter 14"
output: 
  html_document:
    keep_md: TRUE
date: "2023-04-14"
---




```r
library("R6")
```

```
## Warning: package 'R6' was built under R version 4.2.3
```


#14.3 Notes

The private argument of the R6 class works the same way as the public argument: you give it a named list of methods (functions) and fields (not functions). Fields and methods defined in private are available within the methods using private$. You cannot access them outside the class. 


```r
Person <- R6Class("Person", 
  public = list(
    initialize = function(name, age = NA) {
      private$name <- name
      private$age <- age
    },
    print = function(...) {
      cat("Person: \n")
      cat("  Name: ", private$name, "\n", sep = "")
      cat("  Age:  ", private$age, "\n", sep = "")
    }
  ),
  private = list(
    age = NA,
    name = NULL
  )
)

hadley3 <- Person$new("Hadley")
hadley3
```

```
## Person: 
##   Name: Hadley
##   Age:  NA
```

```r
#> Person: 
#>   Name: Hadley
#>   Age:  NA
hadley3$name
```

```
## NULL
```

```r
#> NULL
```

Active fields are implemented using active bindings. Each active binding takes one argument: value. 


```r
Rando <- R6::R6Class("Rando", active = list(
  random = function(value) {
    if (missing(value)) {
      runif(1)  
    } else {
      stop("Can't set `$random`", call. = FALSE)
    }
  }
))
x <- Rando$new()
x$random
```

```
## [1] 0.3041393
```

```r
#> [1] 0.0808
x$random
```

```
## [1] 0.4685788
```

```r
#> [1] 0.834
x$random
```

```
## [1] 0.5948343
```

```r
#> [1] 0.601
```


#14.3 Exercises

1. Create a bank account class that prevents you from directly setting the account balance, but you can still withdraw from and deposit to. Throw an error if you attempt to go into overdraft.


```r
Account <- R6Class("Account",
  active = list(
    deposit = function(value){
      if(missing(value)){0}
    },
    withdraw = function(value2){
      if(missing(value2)){0} 
    }
  ),
  public = list(
    initialize = function(balance){
      private$balance <- deposit - withdraw
    },
    print = function(...){
      cat("Balance: ", private$balance, "\n", sep="")
    }
  ),
  private = list(
    balance = NULL
  )
)
```

2. Create a class with a write-only $password field. It should have $check_password(password) method that returns TRUE or FALSE, but there should be no way to view the complete password.

3. Extend the Rando class with another active binding that allows you to access the previous random value. Ensure that active binding is the only way to access the value.

4. Can subclasses access private fields/methods from their parent? Perform an experiment to find out.

