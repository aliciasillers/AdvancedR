---
title: "chapter 15"
output: 
  html_document:
    keep_md: TRUE
date: "2023-04-21"
---




```r
library("methods")
```

#15.2 Notes

You define an S4 class by calling setClass() with the class name and a definition of its slots, and the names and classes of the class data

```r
setClass("Person", 
  slots = c(
    name = "character", 
    age = "numeric"
  )
)
```

Once the class is defined, you can construct new objects from it by calling new() with the name of the class and a value for each slot

```r
john <- new("Person", name = "John Smith", age = NA_real_)
```

```r
is(john)
```

```
## [1] "Person"
```

```r
#> [1] "Person"
john@name
```

```
## [1] "John Smith"
```

```r
#> [1] "John Smith"
slot(john, "age")
```

```
## [1] NA
```

```r
#> [1] NA
```

Here we’ll create a setter and getter for the age slot by first creating generics with setGeneric() and then defining methods with setMethod()

```r
setGeneric("age", function(x) standardGeneric("age"))
```

```
## [1] "age"
```

```r
setGeneric("age<-", function(x, value) standardGeneric("age<-"))
```

```
## [1] "age<-"
```

```r
setMethod("age", "Person", function(x) x@age)
setMethod("age<-", "Person", function(x, value) {
  x@age <- value
  x
})

age(john) <- 50
age(john)
```

```
## [1] 50
```

```r
#> [1] 50
```

#15.2 Exercises

1. lubridate::period() returns an S4 class. What slots does it have? What class is each slot? What accessors does it provide?

```r
lubridate::period()
```

```
## <Period[0]>
```

```r
class?Period
```

```
## starting httpd help server ... done
```
Answer: The period class has six slots: Data, minute, hour, day, month, and year. They are all numeric objects.    

2. What other ways can you find help for a method? Read ?"?" and summarize the details.
Answer: The "?" allows access to formal S4 method documentation by the method authors. methods?function will look for the documentation methods for the function. The "?" operator can also be called with doc_type supplied as method; in this case also, the topic argument is a function call, but the arguments are now interpreted as specifying the class of the argument

#15.3 Notes


```r
setClass("Person", 
  slots = c(
    name = "character", 
    age = "numeric"
  ), 
  prototype = list(
    name = NA_character_,
    age = NA_real_
  )
)

me <- new("Person", name = "Hadley")
str(me)
```

```
## Formal class 'Person' [package ".GlobalEnv"] with 2 slots
##   ..@ name: chr "Hadley"
##   ..@ age : num NA
```

```r
#> Formal class 'Person' [package ".GlobalEnv"] with 2 slots
#>   ..@ name: chr "Hadley"
#>   ..@ age : num NA
```

There is an additional argument for setClass(): contains. It specifies a class from which to inherit slots and behaviors. 

```r
setClass("Employee", 
  contains = "Person", 
  slots = c(
    boss = "Person"
  ),
  prototype = list(
    boss = new("Person")
  )
)

str(new("Employee"))
```

```
## Formal class 'Employee' [package ".GlobalEnv"] with 3 slots
##   ..@ boss:Formal class 'Person' [package ".GlobalEnv"] with 2 slots
##   .. .. ..@ name: chr NA
##   .. .. ..@ age : num NA
##   ..@ name: chr NA
##   ..@ age : num NA
```

```r
#> Formal class 'Employee' [package ".GlobalEnv"] with 3 slots
#>   ..@ boss:Formal class 'Person' [package ".GlobalEnv"] with 2 slots
#>   .. .. ..@ name: chr NA
#>   .. .. ..@ age : num NA
#>   ..@ name: chr NA
#>   ..@ age : num NA
```

To determine what classes an object inherits from, use is()

```r
is(new("Person"))
```

```
## [1] "Person"
```

```r
#> [1] "Person"
is(new("Employee"))
```

```
## [1] "Employee" "Person"
```

```r
#> [1] "Employee" "Person"
```

```r
is(john, "Person")
```

```
## [1] TRUE
```

```r
#> [1] TRUE
```

User-facing classes should always be paired with a user-friendly helper. A helper should always: have the same name as the class (myclass()), have a good user interface, include useful error messages, and finish by calling methods::new().     

A validator is used to check anything beyond the slots having the correct classes, which is checked automatically. The setValidity() function is used for this. 

```r
setValidity("Person", function(object) {
  if (length(object@name) != length(object@age)) {
    "@name and @age must be same length"
  } else {
    TRUE
  }
})
```

```
## Class "Person" [in ".GlobalEnv"]
## 
## Slots:
##                           
## Name:       name       age
## Class: character   numeric
## 
## Known Subclasses: "Employee"
```

#15.3 Exercises

1. Extend the Person class with fields to match utils::person(). Think about what slots you will need, what class each slot should have, and what you’ll need to check in your validity method.

```r
setClass("Person", 
  slots = c(
    name = "character", 
    age = "numeric",
    given = "character",
    family = "character",
    middle = "character",
    email = "character",
    role = "character",
    comment = "character",
    first = "character",
    last = "character"
  ), 
  prototype = list(
    name = NA_character_,
    age = NA_real_,
    given = NA_character_,
    family = NA_character_,
    middle = NA_character_,
    email = NA_character_,
    role = NA_character_,
    comment = NA_character_,
    first = NA_character_,
    last = NA_character_
  )
)

setValidity("Person", function(object) {
  if (object@role != "aut" | object@role != "com" | object@role != "cph" | object@role != "cre" | object@role != "ctb" | object@role != "ctr" | object@role != "dtc" | object@role != "fnd" | object@role != "rev" | object@role != "ths" | object@role != "trl") {
    "role must be one of the following values: aut, com, cph, cre, ctb, ctr, dtc, fnd, rev, ths, or trl"
  } else if (length(object@name) != length(object@role)) {
    "@name and @age must be same length"
  } else {  
    TRUE
  }  
})
```

```
## Class "Person" [in ".GlobalEnv"]
## 
## Slots:
##                                                                             
## Name:       name       age     given    family    middle     email      role
## Class: character   numeric character character character character character
##                                     
## Name:    comment     first      last
## Class: character character character
```


2. What happens if you define a new S4 class that doesn’t have any slots? (Hint: read about virtual classes in ?setClass.)  

```r
setClass("Test")
#test2 <- new("Test")
```
Answer: It seems to allow you to create the class but not to add objects to the class

3. Imagine you were going to reimplement factors, dates, and data frames in S4. Sketch out the setClass() calls that you would use to define the classes. Think about appropriate slots and prototype.

```r
?date
```


```r
setClass("Factor2", 
  slots = c(
    x = "character",
    levels = "numeric",
    labels = "ANY"
  ), 
  prototype = list(
    x = NA_character_,
    levels = NA_real_,
    labels = levels
  )
)

setClass("DataFrame",
  slots = c(
    data = "list",
    row.names = "character"
    ),
  prototype = list(
    data = list(),
    row.names = NA_character_
  )
)
```



```r
test3 <- "string"
if(test3 != "string" | test3 != "string2") {FALSE} else {TRUE}
```

```
## [1] FALSE
```

