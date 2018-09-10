Hacking It In the Tidyverse
================
Steven M. Mortimer
9/14/2018

-   [Functions](#functions)
    -   [Naming functions](#naming-functions)
    -   [Basic function structure](#basic-function-structure)
    -   [Where functions exist](#where-functions-exist)
    -   [Specify function values with return()](#specify-function-values-with-return)
    -   [Hiding the output of your function with invisible()](#hiding-the-output-of-your-function-with-invisible)
    -   [Specifying default function argument values](#specifying-default-function-argument-values)
    -   [Argument matching in functions](#argument-matching-in-functions)
    -   [Use NULL as the default value for missing arguments](#use-null-as-the-default-value-for-missing-arguments)
    -   [What do those dots (...) mean?](#what-do-those-dots-...-mean)
    -   [Writing recursive functions](#writing-recursive-functions)
-   [Tidyverse Tricks](#tidyverse-tricks)
    -   [lubridate](#lubridate)
    -   [group\_by](#group_by)
    -   [spread/gather](#spreadgather)
    -   [rename\_all](#rename_all)
    -   [map\_df](#map_df)

Functions
=========

Naming functions
----------------

> There are only two hard things in Computer Science: cache invalidation and naming things. -- Phil Karlton

Remember 2 things:

1.  snake\_case - all lowercase, underscore separated
2.  variables are nouns and functions are verbs

Examples:

-   **dplyr** - `mutate()`, `select()`, `sample()`, `slice()`
-   **httr** - `GET()`, `POST()`, `PUT()`, `PATCH()`
    vs.
-   **dplyr** - `diamonds`, `mtcars`, `nycflights13`

Basic function structure
------------------------

Functions have 3 things:

1.  environment (a name puts it in the current, global environment)
2.  arguments (usually)
3.  body (aka logic)

Here is a simple function:

``` r
rename <- function(x){
  paste0(x, " - v2")
}
```

Here is what it does:

``` r
rename("cat")
```

    ## [1] "cat - v2"

Where functions exist
---------------------

Functions exist in the current, Global environment. They overwrite package functions. Try loading the **dplyr** package after we've created our function called `rename()`.

``` r
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following object is masked _by_ '.GlobalEnv':
    ## 
    ##     rename

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

Just to clear up our environment, let's remove the function and restart the R session, which unloads the **dplyr** package.

``` r
# Get rid of our test function
rm(rename)
# also run SHIFT+CMD+F10 to restart your R session
```

Specify function values with return()
-------------------------------------

A function returns, by default the last evaluated value.

Look at the differences in the next three functions and what they return.

**Function \#1**

``` r
# this function returns nothing
square_root <- function(x){
  x <- sqrt(x)
}
square_root(4) # wait, nothing happened?
```

``` r
# let's assign the value from this function
y <- square_root(4)
y # it does actually return something!
```

    ## [1] 2

**Function \#2**

``` r
# this function prints to the console
square_root <- function(x){
  sqrt(x)
}
square_root(4)
```

    ## [1] 2

**Function \#3**

``` r
# this function works
square_root <- function(x){
  x <- sqrt(x)
  return(x)
}
square_root(4)
```

    ## [1] 2

**Lession**: Try be explicit in what your functions return.

There is a raging debate on StackOverflow [here](https://stackoverflow.com/questions/11738823/explicitly-calling-return-in-a-function-or-not), but see the following example. It's not clear what the function returns.

``` r
square_root <- function(x){
  if(x < 0){
    1
  } else {
    sqrt(x)
  }
}
```

A better version of this function makes it very clear that a result (`res`) gets returned. That result is either 0 or the square root based on the value of the input.

``` r
square_root <- function(x){
  if(x < 0){
    res <- 0
  } else {
    res <- sqrt(x)
  }
  return(res)
}
```

Hiding the output of your function with invisible()
---------------------------------------------------

If you look at the iris dataset you can see that the values for the `Sepal.Length` variable do not exceed 7.9.

``` r
iris_dat <- iris
# The max Sepal Length in the iris dataset is 7.9
summary(iris_dat$Sepal.Length)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##   4.300   5.100   5.800   5.843   6.400   7.900

Let's say that we need a function that fixes values in the dataset if they are above a certain value. We'll call that function `cap_sepal_lengths()`. It's defined like this:

``` r
cap_sepal_lengths <- function(iris_dat, max_sep_len=8){
  fixes_made <- FALSE
  if(any(iris_dat$Sepal.Length > max_sep_len)){
    iris_dat$Sepal.Length[iris_dat$Sepal.Length > max_sep_len] <<- max_sep_len
    fixes_made <- TRUE
  }
  invisible(fixes_made)
}
```

When you run the function it doesn't output anything.

``` r
cap_sepal_lengths(iris_dat)
```

When you run the function and assign the value to a variable (`fix_check1`), then you can see that the value is `FALSE`. This means that when we ran the function, none of the values in the dataset needed to be capped.

``` r
fix_check1 <- cap_sepal_lengths(iris_dat)
fix_check1
```

    ## [1] FALSE

Now consider a second example, where want to cap all values at 6.

``` r
fix_check2 <- cap_sepal_lengths(iris_dat, max_sep_len=6)
fix_check2
```

    ## [1] TRUE

In this example you can see that the returned value is `TRUE` meaning that some values did end up needing fixing. The call to `summary` verifies that all values no longer exceed 6.

``` r
summary(iris_dat$Sepal.Length)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##   4.300   5.100   5.800   5.571   6.000   6.000

**Lesson**: Sometimes functions should have side-effect behaviors and `invisible()` can report back the status for us if we are interested in capturing and using.

Specifying default function argument values
-------------------------------------------

It's usually a good idea to specify default values wherever possible. It makes it quicker to use a function that you've created. These values should be provided in your function definition.

``` r
square_root <- function(x = 4){
  return(sqrt(x))
}
square_root()
```

    ## [1] 2

``` r
square_root(25)
```

    ## [1] 5

In this case, you can specify the `x2` default argument by using the `x` argument. This is called

``` r
square_root_plus <- function(x, x2=x^2){
  return(sqrt(x+x2))
}
square_root_plus(4) # sqrt(4 + 16)
```

    ## [1] 4.472136

``` r
square_root_plus(x=4, x2=12) # sqrt(4 + 12)
```

    ## [1] 4

Argument matching in functions
------------------------------

When calling a function you can specify arguments by position, by complete name, or by partial name. Arguments are matched first by exact name (perfect matching), then by prefix matching, and finally by position.

``` r
paste_letters <- function(one="1", two="2", three="3"){
  return(paste(one, two, three, sep="-"))
}
paste_letters("a", "b", "c")
```

    ## [1] "a-b-c"

``` r
paste_letters("a", , "c") # weird positional matching
```

    ## [1] "a-2-c"

``` r
paste_letters(tw="a", th="b", "c") # prefix matching
```

    ## [1] "c-a-b"

``` r
try(paste_letters(t="a", t="b", "c")) # prefix matching
```

Use NULL as the default value for missing arguments
---------------------------------------------------

A lot of times you'll have arguments that don't get used. The best way to deal with these is to use `NULL`.

``` r
paste_letters <- function(one="1", two="2", three=NULL){
  if(is.null(three)){
    res <- paste(one, two, sep="-")  
  } else {
    res <- paste(one, two, three, sep="-")
  }
  return(res)
}
paste_letters("a", "b")
```

    ## [1] "a-b"

What do those dots (...) mean?
------------------------------

The dots are passthrough mechanism to provide any other named arguments that the function doesn't have explicity named. The body of the function will specify what to do with them. Usually they will be used as the inputs to another function. Check out the `lm()` function body to see where the dots go.

``` r
lm
```

    ## function (formula, data, subset, weights, na.action, method = "qr", 
    ##     model = TRUE, x = FALSE, y = FALSE, qr = TRUE, singular.ok = TRUE, 
    ##     contrasts = NULL, offset, ...) 
    ## {
    ##     ret.x <- x
    ##     ret.y <- y
    ##     cl <- match.call()
    ##     mf <- match.call(expand.dots = FALSE)
    ##     m <- match(c("formula", "data", "subset", "weights", "na.action", 
    ##         "offset"), names(mf), 0L)
    ##     mf <- mf[c(1L, m)]
    ##     mf$drop.unused.levels <- TRUE
    ##     mf[[1L]] <- quote(stats::model.frame)
    ##     mf <- eval(mf, parent.frame())
    ##     if (method == "model.frame") 
    ##         return(mf)
    ##     else if (method != "qr") 
    ##         warning(gettextf("method = '%s' is not supported. Using 'qr'", 
    ##             method), domain = NA)
    ##     mt <- attr(mf, "terms")
    ##     y <- model.response(mf, "numeric")
    ##     w <- as.vector(model.weights(mf))
    ##     if (!is.null(w) && !is.numeric(w)) 
    ##         stop("'weights' must be a numeric vector")
    ##     offset <- as.vector(model.offset(mf))
    ##     if (!is.null(offset)) {
    ##         if (length(offset) != NROW(y)) 
    ##             stop(gettextf("number of offsets is %d, should equal %d (number of observations)", 
    ##                 length(offset), NROW(y)), domain = NA)
    ##     }
    ##     if (is.empty.model(mt)) {
    ##         x <- NULL
    ##         z <- list(coefficients = if (is.matrix(y)) matrix(, 0, 
    ##             3) else numeric(), residuals = y, fitted.values = 0 * 
    ##             y, weights = w, rank = 0L, df.residual = if (!is.null(w)) sum(w != 
    ##             0) else if (is.matrix(y)) nrow(y) else length(y))
    ##         if (!is.null(offset)) {
    ##             z$fitted.values <- offset
    ##             z$residuals <- y - offset
    ##         }
    ##     }
    ##     else {
    ##         x <- model.matrix(mt, mf, contrasts)
    ##         z <- if (is.null(w)) 
    ##             lm.fit(x, y, offset = offset, singular.ok = singular.ok, 
    ##                 ...)
    ##         else lm.wfit(x, y, w, offset = offset, singular.ok = singular.ok, 
    ##             ...)
    ##     }
    ##     class(z) <- c(if (is.matrix(y)) "mlm", "lm")
    ##     z$na.action <- attr(mf, "na.action")
    ##     z$offset <- offset
    ##     z$contrasts <- attr(x, "contrasts")
    ##     z$xlevels <- .getXlevels(mt, mf)
    ##     z$call <- cl
    ##     z$terms <- mt
    ##     if (model) 
    ##         z$model <- mf
    ##     if (ret.x) 
    ##         z$x <- x
    ##     if (ret.y) 
    ##         z$y <- y
    ##     if (!qr) 
    ##         z$qr <- NULL
    ##     z
    ## }
    ## <bytecode: 0x7ff72aad0978>
    ## <environment: namespace:stats>

They are provided to the `lm.fit()` or `lm.wfit()` functions. If you look at those functions using `?lm.fit`, you'll see that there are some arguments that we could be specifying in our original `lm()` call to control how things work. For exmaple, the `tol` argument can be specified.

``` r
lm(mpg ~ wt, data=mtcars)
```

    ## 
    ## Call:
    ## lm(formula = mpg ~ wt, data = mtcars)
    ## 
    ## Coefficients:
    ## (Intercept)           wt  
    ##      37.285       -5.344

``` r
lm(mpg ~ wt, data=mtcars, tol=1e-3)
```

    ## 
    ## Call:
    ## lm(formula = mpg ~ wt, data = mtcars, tol = 0.001)
    ## 
    ## Coefficients:
    ## (Intercept)           wt  
    ##      37.285       -5.344

If you create a function like this, try to always direct users to where the dots are going, so they can determine what arguments will actually work when they pass them in.

``` r
# instead of having to specify a function like this:
messaged_mean <- function(x, trim, na.rm=FALSE){
  y <- mean(x=x, trim=trim, na.rm=na.rm)
  message(sprintf("The mean is: %1.3f", y))
  invisible(y)
}
# you can specify it like this in order to allow the passthrough effect
messaged_mean <- function(x, ...){
  y <- mean(x=x, ...)
  message(sprintf("The mean is: %1.3f", y))
  invisible(y)
}
messaged_mean(x=c(1:10,50), trim=0)
```

    ## The mean is: 9.545

``` r
messaged_mean(x=c(1:10,50), trim=.1)
```

    ## The mean is: 6.000

Writing recursive functions
---------------------------

It's possible to have functions call themselves! This is really hard to get your head around, but sometimes very useful. Let's say you need to perform something over and over until a certain point. In this example, we'll take half a number until it becomes a fraction.

``` r
half_it <- function(x, verbose=TRUE){
  if(verbose){
   message(sprintf("x = %s", x))
  }
  if(x < 1){
    res <- 1
  } else {
    res <- half_it(x = x/2)
  }
  return(res)
}
half_it(10)
```

    ## x = 10

    ## x = 5

    ## x = 2.5

    ## x = 1.25

    ## x = 0.625

    ## [1] 1

This type of recursion is common when dealing with nested lists. For example, parse a list for however many nested levels. This is common when paginating against an API. Pagination refers to going through each page of results until you get to the final page.

I highly recommend a quick read by Hadley on functions that is available here: <http://adv-r.had.co.nz/Functions.html#function-arguments>

Tidyverse Tricks
================

lubridate
---------

group\_by
---------

spread/gather
-------------

rename\_all
-----------

map\_df
-------
