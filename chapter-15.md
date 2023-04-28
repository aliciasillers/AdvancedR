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

#15.4 Notes

To create a new S4 generic, call setGeneric() with a function that calls standardGeneric():

```r
setGeneric("myGeneric", function(x) standardGeneric("myGeneric"))
```

```
## [1] "myGeneric"
```
It is bad practice to use {} in the generic as it triggers a special case that is more expensive, and generally best avoided    

The signature argument allows you to control the arguments that are used for method dispatch. If signature is not supplied, all arguments (apart from ...) are used.

A generic isn’t useful without some methods, and in S4 you define methods with setMethod(). There are three important arguments: the name of the generic, the name of the class, and the method itself.

```r
setMethod("myGeneric", "Person", function(x) {
  # method implementation
})
```


```r
setMethod("show", "Person", function(object) {
  cat(is(object)[[1]], "\n",
      "  Name: ", object@name, "\n",
      "  Age:  ", object@age, "\n",
      sep = ""
  )
})
```

all user-accessible slots should be accompanied by a pair of accessors. If the slot is unique to the class, this can just be a function:

```r
person_name <- function(x) x@name
```
Typically, however, you’ll define a generic so that multiple classes can use the same interface:

```r
setGeneric("name<-", function(x, value) standardGeneric("name<-"))
```

```
## [1] "name<-"
```

```r
setMethod("name<-", "Person", function(x, value) {
  x@name <- value
  validObject(x)
  x
})
```

#15.4 Exercises

1. Add age() accessors for the Person class.

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
  validObject(x)
  x
})
```

2. In the definition of the generic, why is it necessary to repeat the name of the generic twice?   
Answer: they are arguments for two different purposes: setting generic, and defining method dispatch    

3. Why does the show() method defined in Section 15.4.3 use is(object)[[1]]?

Answer: 

4. What happens if you define a method with different argument names to the generic?    
Answer: It returns an error saying that methods can only add arguments to the generic if the generic has '...' as an argument    

#15.5 Exercises

1. Draw the method graph for f(😅, 😽)    
Answer: Method graph is in repository under the name "Screenshot 2023-04-28 082130.png"

2. Draw the method graph for f(😃, 😉, 😙).

3. Take the last example which shows multiple dispatch over two classes that use multiple inheritance. What happens if you define a method for all terminal classes? Why does method dispatch not save us much work here?    
Answer: defining methods only for the terminal classes allows there to be methods for all classes, but there are many more terminal classes than other classes, so it does not save that much work. 

#15.6 Notes

In slots and contains you can use S4 classes, S3 classes, or the implicit class (Section 13.7.1) of a base type. To use an S3 class, you must first register it with setOldClass().

```r
setOldClass("data.frame")
setOldClass(c("ordered", "factor"))
setOldClass(c("glm", "lm"))
```

```r
setClass("factor",
  contains = "integer",
  slots = c(
    levels = "character"
  ),
  prototype = structure(
    integer(),
    levels = character()
  )
)
setOldClass("factor", S4Class = "factor")
```

```
## Warning in rm(list = what, pos = classWhere): object '.__C__factor' not found
```


#15.6 Exercises

1. What would a full setOldClass() definition look like for an ordered factor (i.e. add slots and prototype the definition above)?

```r
setClass("factor",
  contains = "integer",
  slots = c(
    levels = "character"
  ),
  prototype = structure(
    integer(),
    levels = character()
  )
)
setOldClass("factor", S4Class = "factor")
```

```
## Warning in rm(list = what, pos = classWhere): object '.__C__factor' not found
```

```r
setClass("ordered",
  contains = "factor",
  slots = c(
    levels = "character",
    ordered = "logical"
  ),
  prototype = structure(
    integer(),
    levels = character(),
    ordered = TRUE
  )
)
setOldClass("ordered", S4Class = "ordered")
```

```
## Warning in rm(list = what, pos = classWhere): object '.__C__ordered' not found
```

2. Define a length method for the Person class.

```r
setGeneric("length")
```

```
## [1] "length"
```

```r
setMethod("length", "Person", function(x) length(x@name))
```


