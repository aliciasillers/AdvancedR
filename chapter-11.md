---
title: "chapter-11"
output: 
  html_document:
    keep_md: TRUE
date: "2023-02-19"
---



```r
library(purrr)
library(memoise)
```

#11.2.3 Exercises

1. Base R provides a function operator in the form of Vectorize(). What does it do? When might you use it?   
Answer: Vectorize() takes a function as an argument and creates a function wrapper that returns a vector of the outputs of the input function. This could be useful for many scenarios in which a function returns multiple output values that the user wants in the form of a vector. For example, if a function contains a for loop computing values in a list to make a new column for a data frame, or if you are using map() to perform a calculation on all the values in a list and want the results in a vector.    

2. Read the source code for possibly(). How does it work?   
Answer: possibly() takes a function and another value and returns that value instead of an error when something does not work. This way, the code still runs, which can be helpful if you are applying a function to multiple arguments. However, this results in the user having less information about where and why there are errors. It works by capturing the error with trycatch and then returning the default value.

3. Read the source code for safely(). How does it work?   
Answer: safely() takes a function and another value and returns two lists: one of the returned values from where the code was successful and one of the errors where the code was not successful. The code still runs and you can see where the errors were encountered. It works by capturing errors with trycatch and creating a list with the function output and captured errors. 

#11.3.1 Exercises

1. Weigh the pros and cons of download.file %>% dot_every(10) %>% delay_by(0.1) versus download.file %>% delay_by(0.1) %>% dot_every(10).

```r
urls <- c(
  "adv-r" = "https://adv-r.hadley.nz", 
  "r4ds" = "http://r4ds.had.co.nz/"
  # and many many more
)
path <- paste(tempdir(), names(urls), ".html")

delay_by <- function(f, amount) {
  force(f)
  force(amount)
  
  function(...) {
    Sys.sleep(amount)
    f(...)
  }
}

dot_every <- function(f, n) {
  force(f)
  force(n)
  
  i <- 0
  function(...) {
    i <<- i + 1
    if (i %% n == 0) cat(".")
    f(...)
  }
}
```

```r
walk2(
  urls, path, 
  download.file %>% dot_every(10) %>% delay_by(0.1), 
  quiet = TRUE
)
```

```r
walk2(
  urls, path, 
  download.file %>% delay_by(0.1) %>% dot_every(10), 
  quiet = TRUE
)
```
Answer: Could change when the dots appear and could affect how long it takes    

2. Should you memoise file.download()? Why or why not?       
Answer: I think you shouldn't because it would use a lot more memory and you probably wouldnt need to download the same files many times.    

3. Create a function operator that reports whenever a file is created or deleted in the working directory, using dir() and setdiff(). What other global function effects might you want to track?

```r
file.track <- function(f){
  
}
```

4. Write a function operator that logs a timestamp and message to a file every time a function is run.

```r
fun.track <- function(f, track){
  force(f)
  track <- file()
  function(...){
    cat(paste0("function ", f, " run at ", date()), file = track, append = TRUE)
    f(...)
  }
}
```

5. Modify delay_by() so that instead of delaying by a fixed amount of time, it ensures that a certain amount of time has elapsed since the function was last called. That is, if you called g <- delay_by(1, f); g(); Sys.sleep(2); g() there shouldn’t be an extra delay.

```r
delay_by2 <- function(f, amount){
  force(f)
  force(amount)
  elapsed <- 0
  function(...){
    if (Sys.time() - elapsed < amount){
      Sys.sleep(amount - (Sys.time() - elapsed))
    }
    f(...)
    on.exit(elapsed <- Sys.time())
  }
}
```

