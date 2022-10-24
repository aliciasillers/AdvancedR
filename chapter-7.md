---
title: "chapter 7"
output: 
  html_document:
    keep_md: true
date: "2022-10-19"
---



# Quiz

1. List at least three ways that an environment differs from a list.    
My answer: There might be a wider range of things that environments can have in them than lists, but I am not sure.   
Textbook answer: There are four ways: every object in an environment must have a name; order doesn’t matter; environments have parents; environments have reference semantics.   

2. What is the parent of the global environment? What is the only environment that doesn’t have a parent?    
My answer: The parent of the global environment might be the project?      
Textbook answer: The parent of the global environment is the last package that you loaded. The only environment that doesn’t have a parent is the empty environment.   

3. What is the enclosing environment of a function? Why is it important?    
My answer: The environment in which the function was created.     
Textbook answer: The enclosing environment of a function is the environment where it was created. It determines where a function looks for variables.    

4. How do you determine the environment from which a function was called?     
My answer: I don't know   
Textbook answer: Use caller_env() or parent.frame().    

5. How are <- and <<- different?    
My answer: <- connects a name to an object. I'm not sure about <<-    
Textbook answer: <- always creates a binding in the current environment; <<- rebinds an existing name in a parent of the current environment.   

# 7.2.7 Exercises

1. List three ways in which an environment differs from a list.   
Answer: Every object in an environment must be bound to a name or else it will be deleted, there is no order in environments, and environments have parent environments which can be used for lexical scoping. 

2. Create an environment as illustrated by the picture.   
Answer:

```r
library(rlang)
loop <- env()
loop <- env(loop)
env_parent(loop)
```

```
## <environment: 0x000001fbac341d30>
```

3.Create a pair of environments as illustrated by the picture.    
Answer:

```r
dedoop <- env(loop)
loop <- env(dedoop)
```

4. Explain why e[[1]] and e[c("a", "b")] don’t make sense when e is an environment.   
Answer: Environments are not vectors, so you cannot call objects using numeric indices. 

5. Create a version of env_poke() that will only bind new names, never re-bind old names. Some programming languages only do this, and are known as single assignment languages.   
Answer:

```r
env_poke_2 <- function(e, n, v){
  if (exists(n) == TRUE) {
    return("error: name already exists")
  }
  else {
    env_poke(e, n, v)
  }
}

env_poke_2(loop, "zebra", 100)
```

6. What does this function do? How does it differ from <<- and why might you prefer it?

```r
rebind <- function(name, value, env = caller_env()) {
  if (identical(env, empty_env())) {
    stop("Can't find `", name, "`", call. = FALSE)
  } else if (env_has(env, name)) {
    env_poke(env, name, value)
  } else {
    rebind(name, value, env_parent(env))
  }
}
#rebind("a", 10)
#> Error: Can't find `a`
a <- 5
rebind("a", 10)
a
```

```
## [1] 10
```

```r
#> [1] 10
```
Answer: This function replaces the value of an object bound to a name. It looks in the current environment first and then the parent environment, which might be preferable to <<-, which does not look in the current environment. 

# 7.3.1 Exercises

1. Modify where() to return all environments that contain a binding for name. Carefully think through what type of object the function will need to return.   
Answer:

```r
where_2 <- function(name, env = caller_env()){
  all_envs <- c()
  i <- 1
  while (!identical(env, empty_env())) {
  if (env_has(env, name)) {
    # Success
    all_envs[i] <- c(envs)
    i <- i + 1
    # Recursive
    where_2(name, env_parent(env))
  } else {
    # Recursive 
    where_2(name, env_parent(env))
  }
  }
}
```

2. Write a function called fget() that finds only function objects. It should have two arguments, name and env, and should obey the regular scoping rules for functions: if there’s an object with a matching name that’s not a function, look in the parent. For an added challenge, also add an inherits argument which controls whether the function recurses up the parents or only looks in one environment.    
Answer:

```r
fget <- function(name, env = caller_env()) {
  if (identical(env, empty_env())) {
    stop("no match found")
  } else if (env_has(env, name) == TRUE && is.function(name) == TRUE) {
    # success 
    return(name)
  } else {
    # recursive case
    fget(name, env = env_parent(env))
  }
}

#fget("where_2")
```

