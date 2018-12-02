How Work in R Gets Shared
================
Steven M. Mortimer
12/11/2018

-   [Sharing with R Users](#sharing-with-r-users)
    -   [R Project Folder Structures](#r-project-folder-structures)
    -   [GitHub + Git flow](#github-git-flow)
    -   [R Package Structure](#r-package-structure)
-   [Sharing with non-R Users](#sharing-with-non-r-users)
    -   [Google Sheets](#google-sheets)
    -   [RMarkdown](#rmarkdown)
    -   [Shiny + HTML/CSS/JavaScript](#shiny-htmlcssjavascript)

Sharing with R Users
====================

<center>
<img src="./img/xkcd-documents.png" />
</center>
<br>

R Project Folder Structures
---------------------------

Many programming languages and frameworks use folder structures for configuration and extensibility. To not use a standard would mean reinventing the wheel each time and risk confusing/slowing down collaborators. This problem has been tackled by many individuals in the R community. Based on their suggestions I have adapted the following structure to be quick and easy to use:

<center>
<img src="./img/folder-structure.png" />
</center>
<br>

The key features of this structure is that you physically separate the inputs, intermediate data, and outputs into three different folders:

1.  `data`: Folder for put raw, unprocessed data
2.  `cache`: Folder for intermediate data, if needed
3.  `output`: Folder for cleaned data, plots, etc.

There are two remaining folders in the structure:

1.  `docs`: Folder for supporting articles, explanations, and other documentation
2.  `R`: Folder for R scripts that hold global settings or functions if needed

In the top level are your analytical R scripts. Oftentimes these scripts can be long and complicated. I recommend breaking up any sequential steps into smaller files that numbered so that the order to run them is obvious and then call each of them using the `source()` function. This way you can create a file called `main.R` that you can configure to run all of your analysis from one file. An example of a `main.R` file might look like this:

``` r
# set runtime options and load packages ------------------------------

options(stringsAsFactors = FALSE, scipen = 99)

suppressMessages(library(here)) # to manage file paths
suppressMessages(library(tidyverse)) # to process data

# load R scripts that contains variables and functions for the analysis
source(here::here("byob-study", "R", "globals.R"))
source(here::here("byob-study", "R", "helpers.R"))

# create a folder for storing the output of the analysis from today
# this way your output doesn't accidentally get overwritten from another time
todays_date_formatted <- format(Sys.Date(), '%Y%m%d')
dir.create(here::here('output', todays_date_formatted), showWarnings = FALSE)

# sourcing scripts to run analysis -----------------------------------

source(here::here('01-wrangle.R'))
source(here::here('02-model.R'))
```

### Caching

Sometimes your project will involve a very large dataset or a sequence of complex and long-running processing steps. "Caching" is a term for saving off your analysis mid-way through the process so that you can restart it exactly from that particular spot. R Markdown documents have this built into them, but you can also do this from any R script by using the package **simpleCache**. **simpleCache** was created at the University of Virginia in part by the UVA R Users Group leader, VP (Pete) Nagraj. Below is an example that uses a simulation to show how the standard error is calculated. The value is stored in the variable `std_error`. The caching process will check the `cache` folder and if the object exists, then that block of code is not run and the cached output is loaded from the cache folder. If the cached object is not found, then the code runs, creates the object, and saves it to the cache folder.

``` r
suppressMessages(library(here))  # for file management
suppressMessages(library(simpleCache)) # for caching long running parts of the script 

# set the cache directory
setCacheDir(here::here('cache'))
# delete the files if you want to refresh the cache or run `deleteCaches()` 

# the mean of 1000 standard normal observations has a standard deviation 
# that should approach 1/sqrt(1000) = 0.0316
simpleCache("std_error", {
  n_sample_avgs <- numeric(100)
  for(i in 1:100){
    n_sample_avgs[i] <- mean(rnorm(1000, 0, 1))
  }
  sd(n_sample_avgs)
})
```

### Resources

Here is a list of resources to help you review and inform your approach:

-   [**RProjectTemplate**](http://projecttemplate.net/architecture.html) by John Myles White
-   [**pRojects**](https://itsalocke.com/projects/) by Steph Locke
-   [**new-project-template**](https://github.com/pavopax/new-project-template) by Paul Paczuski
-   [**Cookiecutter Data Science**](https://drivendata.github.io/cookiecutter-data-science/#directory-structure) (it is for Python, but applies to R)

GitHub + Git flow
-----------------

Once you have worked hard on creating a project in R you typically want to do 2 things: 1) Backup the work you've done and 2) Share it with others. GitHub is the de facto place to accomplish both of these tasks.

-   Pull from Git Scenarios Talk (walk through examples?)

R Package Structure
-------------------

-   Why package
-   install from github

### Package Architecture

-   description
-   man/roxygen
-   r code

### Package Testing/Integrity

-   tests
-   travis
-   codecov

### Resources

Here are two of the best resources to help you create a package:

-   [**R Packages**](http://r-pkgs.had.co.nz) by Hadley Wickham
-   [*Writing an R Package from scratch*](https://hilaryparker.com/2014/04/29/writing-an-r-package-from-scratch/) by Hilary Parker

Sharing with non-R Users
========================

Google Sheets
-------------

-   Example of how to do it (create a public spreadsheet and let people push to it)
-   talk about auth a little
-   Show Version History benefit

RMarkdown
---------

-   describe markdown
-   describe chunk
-   show Caching
-   show how this document works (Rmd -&gt; github md)

Shiny + HTML/CSS/JavaScript
---------------------------

-   talk about ui.R, server.R

### Reactivity

### Shiny App Example

-   pull example from Packt book

### Shiny App Tips

-   give pro tips on building a shiny app