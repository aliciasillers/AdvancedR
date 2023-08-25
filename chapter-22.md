---
title: "chapter-22"
author: "Alicia Sillers"
date: "2023-08-22"
output: 
  html_document:
    keep_md: TRUE
---



# 22.2 Notes

four step debugging process: look up error message to see if there is a known solution, create a reproducible example to help diagnose the cause of the error, identify the line of code that is causing the error, solve and check 

# 22.3 Notes

traceback() shows you the sequence of calls that lead to the error, with the bottom call being the first one. they are clickable in rstudio and will take you to the corresponding line of code.   

you can use rlang::with_abort() and rlang::last_trace() to see the call tree

# 22.4 Notes

the interactive debugger allows you to pause execution of a function and interactively explore its state    
you can use rstudio's "Rerun and Debug" tool or insert the call browser() where you want a function to pause   

browser() commands:   
Next, n: executes the next step in the function. If you have a variable named n, you’ll need print(n) to display its value.    
Step into,  or s: works like next, but if the next step is a function, it will step into that function so you can explore it interactively.    
Finish,  or f: finishes execution of the current loop or function.    
Continue, c: leaves interactive debugging and continues regular execution of the function. This is useful if you’ve fixed the bad state and want to check that the function proceeds correctly.    
Stop, Q: stops debugging, terminates the function, and returns to the global workspace. Use this once you’ve figured out where the problem is, and you’re ready to fix it and reload the code.    
where: prints stack trace of active calls   

debug() inserts a browser statement in the first line of the specified function. undebug() removes it.

# 22.5 Notes

print debugging is where you insert numerous print statements to precisely locate the problem, and see the values of important variables   
