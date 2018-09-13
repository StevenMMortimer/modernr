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
    -   [Functions with a dot in the name](#functions-with-a-dot-in-the-name)
    -   [Writing recursive functions](#writing-recursive-functions)
-   [Tidyverse Tricks](#tidyverse-tricks)
    -   [The new conditional select() and rename() functions](#the-new-conditional-select-and-rename-functions)
    -   [The everything() function for selecting columns](#the-everything-function-for-selecting-columns)
    -   [Using purrr's map() function](#using-purrrs-map-function)
    -   [lubridate](#lubridate)
    -   [group\_by](#group_by)
    -   [spread/gather](#spreadgather)
    -   [rename\_all](#rename_all)

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
    ## <bytecode: 0x7fb4b8169350>
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

Functions with a dot in the name
--------------------------------

You may notice that sometimes functions have a dot in the name. Most of the time the package developer is trying to make the name more readable, but this is not always the case. Look at the print function:

``` r
print
```

    ## function (x, ...) 
    ## UseMethod("print")
    ## <bytecode: 0x7fb4b7137518>
    ## <environment: namespace:base>

There's not much to the function and you can see that it includes those dots. This is basically defining a function called "print" that takes an object and some other unnamed arguments and does something. We're not sure what though. Now let's look at the function `print.data.frame`:

``` r
print.data.frame
```

    ## function (x, ..., digits = NULL, quote = FALSE, right = TRUE, 
    ##     row.names = TRUE) 
    ## {
    ##     n <- length(row.names(x))
    ##     if (length(x) == 0L) {
    ##         cat(sprintf(ngettext(n, "data frame with 0 columns and %d row", 
    ##             "data frame with 0 columns and %d rows"), n), "\n", 
    ##             sep = "")
    ##     }
    ##     else if (n == 0L) {
    ##         print.default(names(x), quote = FALSE)
    ##         cat(gettext("<0 rows> (or 0-length row.names)\n"))
    ##     }
    ##     else {
    ##         m <- as.matrix(format.data.frame(x, digits = digits, 
    ##             na.encode = FALSE))
    ##         if (!isTRUE(row.names)) 
    ##             dimnames(m)[[1L]] <- if (identical(row.names, FALSE)) 
    ##                 rep.int("", n)
    ##             else row.names
    ##         print(m, ..., quote = quote, right = right)
    ##     }
    ##     invisible(x)
    ## }
    ## <bytecode: 0x7fb4b820a0a0>
    ## <environment: namespace:base>

You can see that whenever you print a `data.frame` you can control the number of digits, quoting, etc. The body of the function contains the familiar logic of what gets printed to the screen when you look at a data.frame. The `print()` function is called a generic. When you run it R will look for a more specific version of print that knows how to handle the specific object type that you passed to it. In this case, it will print a data.frame according to the function `print.data.frame`. A couple simple examples of this at work are:

``` r
print(head(iris))
```

    ##   Sepal.Length Sepal.Width Petal.Length Petal.Width Species
    ## 1          5.1         3.5          1.4         0.2  setosa
    ## 2          4.9         3.0          1.4         0.2  setosa
    ## 3          4.7         3.2          1.3         0.2  setosa
    ## 4          4.6         3.1          1.5         0.2  setosa
    ## 5          5.0         3.6          1.4         0.2  setosa
    ## 6          5.4         3.9          1.7         0.4  setosa

``` r
print(Sys.time())
```

    ## [1] "2018-09-13 16:50:23 EDT"

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

Before we get started, let's load some required packages.

``` r
library(plyr)
library(dplyr)
library(purrr)
library(lubridate)
```

The new conditional select() and rename() functions
---------------------------------------------------

In newer versions of `dplyr` you have even better version of `select()` and `rename()`. Namely, `select_if`

The everything() function for selecting columns
-----------------------------------------------

Also, don't forget about the handy function `everything()`. This literally selects everything else that you haven't already grabbed in your `select()` function. This is really handy when you want to re-order your columns or set things up for later reporting.

``` r
iris %>% 
  select(Species, everything()) %>%
  as_tibble()
```

    ## # A tibble: 150 x 5
    ##    Species Sepal.Length Sepal.Width Petal.Length Petal.Width
    ##    <fct>          <dbl>       <dbl>        <dbl>       <dbl>
    ##  1 setosa          5.10        3.50         1.40       0.200
    ##  2 setosa          4.90        3.00         1.40       0.200
    ##  3 setosa          4.70        3.20         1.30       0.200
    ##  4 setosa          4.60        3.10         1.50       0.200
    ##  5 setosa          5.00        3.60         1.40       0.200
    ##  6 setosa          5.40        3.90         1.70       0.400
    ##  7 setosa          4.60        3.40         1.40       0.300
    ##  8 setosa          5.00        3.40         1.50       0.200
    ##  9 setosa          4.40        2.90         1.40       0.200
    ## 10 setosa          4.90        3.10         1.50       0.100
    ## # ... with 140 more rows

Using purrr's map() function
----------------------------

You'll hear R users talk about vectorized functions and how they are more efficient than running a loop. It's true that sometimes loops are necessary or they are easier to read. However, if you're trying to perform the same operations on everything in a `vector`, `list`, or row of a `data.frame`, then there are some functions to help you.

In the old days, there was a package called `plyr`. It had functions that took a specific type of input, then repeated an operation on each element. The documentation for the `llply` function says it all: &gt; For each element of a list, apply function, keeping results as a list.

``` r
llply(iris, summary)
```

    ## $Sepal.Length
    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##   4.300   5.100   5.800   5.843   6.400   7.900 
    ## 
    ## $Sepal.Width
    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##   2.000   2.800   3.000   3.057   3.300   4.400 
    ## 
    ## $Petal.Length
    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##   1.000   1.600   4.350   3.758   5.100   6.900 
    ## 
    ## $Petal.Width
    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##   0.100   0.300   1.300   1.199   1.800   2.500 
    ## 
    ## $Species
    ##     setosa versicolor  virginica 
    ##         50         50         50

If you have a `data.frame` and want to perform a function for each component of it, then just use `dlply`:

``` r
dlply(iris, .(Species), summary)
```

    ## $setosa
    ##   Sepal.Length    Sepal.Width     Petal.Length    Petal.Width   
    ##  Min.   :4.300   Min.   :2.300   Min.   :1.000   Min.   :0.100  
    ##  1st Qu.:4.800   1st Qu.:3.200   1st Qu.:1.400   1st Qu.:0.200  
    ##  Median :5.000   Median :3.400   Median :1.500   Median :0.200  
    ##  Mean   :5.006   Mean   :3.428   Mean   :1.462   Mean   :0.246  
    ##  3rd Qu.:5.200   3rd Qu.:3.675   3rd Qu.:1.575   3rd Qu.:0.300  
    ##  Max.   :5.800   Max.   :4.400   Max.   :1.900   Max.   :0.600  
    ##        Species  
    ##  setosa    :50  
    ##  versicolor: 0  
    ##  virginica : 0  
    ##                 
    ##                 
    ##                 
    ## 
    ## $versicolor
    ##   Sepal.Length    Sepal.Width     Petal.Length   Petal.Width   
    ##  Min.   :4.900   Min.   :2.000   Min.   :3.00   Min.   :1.000  
    ##  1st Qu.:5.600   1st Qu.:2.525   1st Qu.:4.00   1st Qu.:1.200  
    ##  Median :5.900   Median :2.800   Median :4.35   Median :1.300  
    ##  Mean   :5.936   Mean   :2.770   Mean   :4.26   Mean   :1.326  
    ##  3rd Qu.:6.300   3rd Qu.:3.000   3rd Qu.:4.60   3rd Qu.:1.500  
    ##  Max.   :7.000   Max.   :3.400   Max.   :5.10   Max.   :1.800  
    ##        Species  
    ##  setosa    : 0  
    ##  versicolor:50  
    ##  virginica : 0  
    ##                 
    ##                 
    ##                 
    ## 
    ## $virginica
    ##   Sepal.Length    Sepal.Width     Petal.Length    Petal.Width   
    ##  Min.   :4.900   Min.   :2.200   Min.   :4.500   Min.   :1.400  
    ##  1st Qu.:6.225   1st Qu.:2.800   1st Qu.:5.100   1st Qu.:1.800  
    ##  Median :6.500   Median :3.000   Median :5.550   Median :2.000  
    ##  Mean   :6.588   Mean   :2.974   Mean   :5.552   Mean   :2.026  
    ##  3rd Qu.:6.900   3rd Qu.:3.175   3rd Qu.:5.875   3rd Qu.:2.300  
    ##  Max.   :7.900   Max.   :3.800   Max.   :6.900   Max.   :2.500  
    ##        Species  
    ##  setosa    : 0  
    ##  versicolor: 0  
    ##  virginica :50  
    ##                 
    ##                 
    ##                 
    ## 
    ## attr(,"split_type")
    ## [1] "data.frame"
    ## attr(,"split_labels")
    ##      Species
    ## 1     setosa
    ## 2 versicolor
    ## 3  virginica

Technically, this is not part of the tidyverse, but the inspiration for the tidyverse was built upon packages like **plyr**. Actually the `map()` functions in the tidyverse behave exactly the same:

``` r
map(iris, summary)
```

    ## $Sepal.Length
    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##   4.300   5.100   5.800   5.843   6.400   7.900 
    ## 
    ## $Sepal.Width
    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##   2.000   2.800   3.000   3.057   3.300   4.400 
    ## 
    ## $Petal.Length
    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##   1.000   1.600   4.350   3.758   5.100   6.900 
    ## 
    ## $Petal.Width
    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##   0.100   0.300   1.300   1.199   1.800   2.500 
    ## 
    ## $Species
    ##     setosa versicolor  virginica 
    ##         50         50         50

``` r
iris %>% split(.$Species) %>% map(summary)
```

    ## $setosa
    ##   Sepal.Length    Sepal.Width     Petal.Length    Petal.Width   
    ##  Min.   :4.300   Min.   :2.300   Min.   :1.000   Min.   :0.100  
    ##  1st Qu.:4.800   1st Qu.:3.200   1st Qu.:1.400   1st Qu.:0.200  
    ##  Median :5.000   Median :3.400   Median :1.500   Median :0.200  
    ##  Mean   :5.006   Mean   :3.428   Mean   :1.462   Mean   :0.246  
    ##  3rd Qu.:5.200   3rd Qu.:3.675   3rd Qu.:1.575   3rd Qu.:0.300  
    ##  Max.   :5.800   Max.   :4.400   Max.   :1.900   Max.   :0.600  
    ##        Species  
    ##  setosa    :50  
    ##  versicolor: 0  
    ##  virginica : 0  
    ##                 
    ##                 
    ##                 
    ## 
    ## $versicolor
    ##   Sepal.Length    Sepal.Width     Petal.Length   Petal.Width   
    ##  Min.   :4.900   Min.   :2.000   Min.   :3.00   Min.   :1.000  
    ##  1st Qu.:5.600   1st Qu.:2.525   1st Qu.:4.00   1st Qu.:1.200  
    ##  Median :5.900   Median :2.800   Median :4.35   Median :1.300  
    ##  Mean   :5.936   Mean   :2.770   Mean   :4.26   Mean   :1.326  
    ##  3rd Qu.:6.300   3rd Qu.:3.000   3rd Qu.:4.60   3rd Qu.:1.500  
    ##  Max.   :7.000   Max.   :3.400   Max.   :5.10   Max.   :1.800  
    ##        Species  
    ##  setosa    : 0  
    ##  versicolor:50  
    ##  virginica : 0  
    ##                 
    ##                 
    ##                 
    ## 
    ## $virginica
    ##   Sepal.Length    Sepal.Width     Petal.Length    Petal.Width   
    ##  Min.   :4.900   Min.   :2.200   Min.   :4.500   Min.   :1.400  
    ##  1st Qu.:6.225   1st Qu.:2.800   1st Qu.:5.100   1st Qu.:1.800  
    ##  Median :6.500   Median :3.000   Median :5.550   Median :2.000  
    ##  Mean   :6.588   Mean   :2.974   Mean   :5.552   Mean   :2.026  
    ##  3rd Qu.:6.900   3rd Qu.:3.175   3rd Qu.:5.875   3rd Qu.:2.300  
    ##  Max.   :7.900   Max.   :3.800   Max.   :6.900   Max.   :2.500  
    ##        Species  
    ##  setosa    : 0  
    ##  versicolor: 0  
    ##  virginica :50  
    ##                 
    ##                 
    ## 

Just like the **plyr** functions indicate the input and output in the function name \*\*purrr\* functions are the same way. The `map()` function takes a list and returns a list. However, `map_df()` will take a list and try to return a `data.frame`, casting the value as a `data.frame` and binding together if possible.

``` r
iris %>% select_if(is.numeric) %>% map_df(summary)
```

    ## # A tibble: 6 x 4
    ##   Sepal.Length Sepal.Width Petal.Length Petal.Width
    ##          <dbl>       <dbl>        <dbl>       <dbl>
    ## 1         4.30        2.00         1.00       0.100
    ## 2         5.10        2.80         1.60       0.300
    ## 3         5.80        3.00         4.35       1.30 
    ## 4         5.84        3.06         3.76       1.20 
    ## 5         6.40        3.30         5.10       1.80 
    ## 6         7.90        4.40         6.90       2.50

To show a more realistic example, let's say that we need to load the iris dataset and add a new column with the dataset number. Here is the loop way:

``` r
all_dat <- NULL
for (i in 1:5){
  this_iris <- iris
  this_iris$dat_numb <- i
  all_dat <- rbind(all_dat, this_iris)
}
```

The tidyverse, **purrr** way would be:

``` r
map_df(1:5, ~as_tibble(mutate(iris, dat_numb=.)))
```

    ## # A tibble: 750 x 6
    ##    Sepal.Length Sepal.Width Petal.Length Petal.Width Species dat_numb
    ##           <dbl>       <dbl>        <dbl>       <dbl> <fct>      <int>
    ##  1         5.10        3.50         1.40       0.200 setosa         1
    ##  2         4.90        3.00         1.40       0.200 setosa         1
    ##  3         4.70        3.20         1.30       0.200 setosa         1
    ##  4         4.60        3.10         1.50       0.200 setosa         1
    ##  5         5.00        3.60         1.40       0.200 setosa         1
    ##  6         5.40        3.90         1.70       0.400 setosa         1
    ##  7         4.60        3.40         1.40       0.300 setosa         1
    ##  8         5.00        3.40         1.50       0.200 setosa         1
    ##  9         4.40        2.90         1.40       0.200 setosa         1
    ## 10         4.90        3.10         1.50       0.100 setosa         1
    ## # ... with 740 more rows

There are two reasons why this would be preferable: 1) It is shorter and 2) it's 10x slower to do it as a loop! The tradeoff is that you need to have a stronger grasp of how the function is operating.

lubridate
---------

group\_by
---------

spread/gather
-------------

rename\_all
-----------
