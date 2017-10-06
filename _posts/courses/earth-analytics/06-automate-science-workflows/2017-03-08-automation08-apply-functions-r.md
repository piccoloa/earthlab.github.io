---
layout: single
title: "Use lapply in R Instead of For Loops to Process .csv files - Efficient Coding in R"
excerpt: "Learn how to take code in a for loop and convert it to be used in an apply function. Make your R code more efficient and expressive programming."
authors: ['Leah Wasser', 'Bryce Mecum', 'Max Joseph']
modified: '2017-10-06'
category: [courses]
class-lesson: ['automating-your-science-r']
permalink: /courses/earth-analytics/automate-science-workflows/use-apply-functions-for-efficient-code-r/
nav-title: 'lapply vs for loops'
week: 6
course: "earth-analytics"
sidebar:
  nav:
author_profile: false
comments: true
topics:
  reproducible-science-and-programming: ['literate-expressive-programming', 'functions']
order: 7
redirect_from:
---

{% include toc title="In This Lesson" icon="file-text" %}

<div class='notice--success' markdown="1">

## <i class="fa fa-graduation-cap" aria-hidden="true"></i> Learning Objectives

After completing this tutorial, you will be able to:

* Use the `lapply()` function in `R` to automate your code.

## <i class="fa fa-check-square-o fa-2" aria-hidden="true"></i> What you need

You will need a computer with internet access to complete this lesson.

</div>

In the previous lessons, we learned how to use for loops to perform tasks that
you want to implement over and over - for example on a set of files. For loops are
a good start to automating your code. However if you want to scale this automation
to process more and / or larger files, the `R` `apply` family of functions are useful
to know about.

Apply functions perform a task over and over - on a list, vector, etc. So, for example
you can use the lapply function (list apply) on the list of file names that
you generate when using `list.files()`.

## Why use apply vs for loops

There are several good reasons to use the apply family of functions.

**1. They make your code more expressive and in turn easier to read:**

Here's what the master, Hadley Wikham has to say about expressive code and the apply

> The point of the apply (and plyr) family of functions is not speed, but expressiveness. They also tend to prevent bugs because they eliminate the book keeping code needed with loops.
>
> Lately, answers on stackoverflow have over-emphasised speed. Your code will get faster on its own as computers get faster and R-core optimises the internals of R. Your code will never get more elegant or easier to understand on its own.
>
> In this case you can have the best of both worlds: an elegant answer using vectorisation that is also very fast, (million > 0) * 2 - 1.
> <a href="https://stackoverflow.com/questions/5533246/why-is-apply-method-slower-than-a-for-loop-in-r/5538309#5538309" target="_blank">Source: Hadley Wikham, stackoverflow comment</a>

And another quote:

> A common reflex is to use a function in the apply family. This is not vectorization,
> it is loop-hiding. The apply function has a for loop in its definition -- Source: Patrick Burns - the Inferno.

**2. They make it easier to parallelize your code:** Most computers these days have
more than one core that can be used to process your data. However, by default,
most functions in R only take advantage of one core on your machine. This means your
computer course process things faster. Parallelized code refers to code that is
optimized to use the cores available to it on a machine. While this topic is out
of the scope of this class - it's important to know about if you ever need to process
large amounts of data - particularly in a cloud or high performance computing environment.

**3. They make your code just a bit faster** At the bottom of this lesson you'll see
a quick benchmark test where we see whether the apply version of a for loop is faster
or not. The apply functions do run a for loop in the background. However they
often do it in the C programming language (which is used to build R). This does
make the apply functions a few milliseconds faster than regular for loops.
However, this is not the main reason to use apply functions!



```r
# view code for the lapply function
lapply
## function (X, FUN, ...) 
## {
##     FUN <- match.fun(FUN)
##     if (!is.vector(X) || is.object(X)) 
##         X <- as.list(X)
##     .Internal(lapply(X, FUN))
## }
## <bytecode: 0x10382ea08>
## <environment: namespace:base>
```



```r
library(parallel)
# how many cores are on this machine
detectCores()
## [1] 8
```

## Use lapply to Process Lists of Files

Next, let's look at an example of using lapply to perform the same task that
you performed in the previous lesson. To do this we will need to

1. write a function that performs all of the tasks that we executed in our for loop
2. call the apply function and tell it to use the function that we created in step 1.

To get started, call the lubridate and dplyr libraries like you did in the previous lessons.


```r
library(lubridate)
library(dplyr)
```

### How lapply Works

> lapply takes a vector (or list) as its first argument (in this case a vector of the continent names), then a function as its second argument. This function is then executed on every element in the first argument. This is very similar to a for loop: first, cc stores the first continent name, “Asia”, then runs the code in the function body, then cc stores the second continent name, and runs the function body, and so on. The code in the function body can be thought of in exactly the same way as the body of the for loop. The result of the last line is then returned to lapply, which combines the results into a list. --Software Carpentry



In the previous lessons, we created a list of files in a directory.




Create a function that performs all of the tasks performed in the `for loop` above.
Note that you


```r
# create a function that performs all of the tasks performed in the for loop above
summarize_data <- function(a_csv, the_dir) {
  # open the data, fix the date and add a new column
  the_data <- read.csv(a_csv, header = TRUE, na.strings = 999.99) %>%
      mutate(DATE = as.POSIXct(DATE, tz = "America/Denver", format = "%Y-%m-%d %H:%M:%S"),
              # add a column with precip in mm - you did this using a function previously
             precip_mm = (HPCP * 25.4))

  # write the csv to a new file
  write.csv(the_data, file = paste0(the_dir, "/", basename(a_csv)),
            na = "999.99")
}
```

As you did above, make sure your output directory is created.
Then use `list.files()` to get a list of all of the files that you'd like to process.


```r

the_dir_ex <- "data/week_06/outputs/example"
check_create_dir(the_dir_ex)
# get a list of all files that you want to process
# you can use a list with the lapply function
all_precip_files <- list.files("data/week_06", pattern = "*.csv",
                             full.names = TRUE)

```

Now you can perform the same task that you performed above in a loop with one
line of code (ok two if you break
them up for readability).


```r

lapply(all_precip_files, FUN = summarize_data,
       the_dir = the_dir_ex)
## [[1]]
## NULL
## 
## [[2]]
## NULL
## 
## [[3]]
## NULL
## 
## [[4]]
## NULL
## 
## [[5]]
## NULL
## 
## [[6]]
## NULL
## 
## [[7]]
## NULL
## 
## [[8]]
## NULL
## 
## [[9]]
## NULL
## 
## [[10]]
## NULL
## 
## [[11]]
## NULL
```



```r

# turn off the output empty list
invisible(lapply(all_precip_files, (FUN = summarize_data),
       the_dir = the_dir_ex))
```



## Are Apply Function Faster Than For Loops?

As promised let's test our code to see whether the `lapply()` function is in fact
faster than the `for loop`.



```r
# let's see what approach is faster
library(microbenchmark)
microbenchmark(invisible(lapply(all_precip_files, (FUN = summarize_data),
       the_dir = the_dir_ex)))
## Unit: milliseconds
##                                                                               expr
##  invisible(lapply(all_precip_files, (FUN = summarize_data), the_dir = the_dir_ex))
##       min       lq     mean   median       uq      max neval
##  103.1938 106.1875 110.2808 108.3014 110.6964 180.3931   100
```



```r

# print the name of each file
microbenchmark(for (file in all_precip_files) {
  # read in the csv
  the_data <- read.csv(file, header = TRUE, na.strings = 999.99) %>%
    mutate(DATE = as.POSIXct(DATE, tz = "America/Denver", format = "%Y-%m-%d %H:%M:%S"),
            # add a column with precip in mm
           precip_mm = in_to_mm(HPCP))

  # write the csv to a new file
  write.csv(the_data, file = paste0(the_dir, "/", basename(file)),
            na = "999.99")

})
## Unit: milliseconds
##                                                                                                                                                                                                                                                                                                                                           expr
##  for (file in all_precip_files) {     the_data <- read.csv(file, header = TRUE, na.strings = 999.99) %>%          mutate(DATE = as.POSIXct(DATE, tz = "America/Denver",              format = "%Y-%m-%d %H:%M:%S"), precip_mm = in_to_mm(HPCP))     write.csv(the_data, file = paste0(the_dir, "/", basename(file)),          na = "999.99") }
##       min       lq     mean   median       uq      max neval
##  104.0861 106.9016 109.4824 108.3587 109.9775 170.9662   100
```

Is it faster on average? Perhaps just by a few milliseconds?


<div class="notice--info" markdown="1">

## Additional Resources

* <a href = "http://resbaz.github.io/r-intermediate-gapminder/17-apply.html
" target = "_blank">Software Carpentry apply lesson </a>
* <a href = "http://www.burns-stat.com/pages/Tutor/R_inferno.pdf
" target = "_blank">The R Inferno - Patrick Burns </a>


</div>