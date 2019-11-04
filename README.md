# R Workshop


## Prerequisites

Before we can get started, you need to install R and famliarize with some concepts.

### Installing R 

Install the latest version of **R** from [https://cran.rstudio.com/](https://cran.rstudio.com). Most of the code will probably run also on older versions of R. Beware that you will need to re-install packages after updating R to a newver version.

We will be using an Integrated Development Environment (IDE) called **RStudio** to interact with R. However, there is nothing to prohibit using R at the command line or in some other interface. You can download RStudio for free from [https://rstudio.com/products/rstudio/download/](https://rstudio.com/products/rstudio/download/#download).

> The RStudio GUI has multiple ‘panes’. Except for the ‘Console’ pane (by default in the lower left corner), the other panes (‘Environment’, ‘History’, ‘Files’, ‘Plots’, ‘Packages’, ‘Help’ etc.) are simply for convenience. If you choose to run R outside RStudio, the interaction will be _identical_ to working in the ‘Console’ pane.

Once you installed R, please run the following lines of code.

``` r
install.packages("tidyverse")
install.packages("data.table")
```

_Note to macOS users:_ To use `data.table`’s parallel processing capability, you will need a version of OpenMP to be installed on your machine. Consider to follow the instructions on [https://github.com/Rdatatable/data.table/](https://github.com/Rdatatable/data.table/wiki/Installation#openmp-enabled-compiler-for-mac).

### Getting Started

In this [first part](part_01-basic_interactions.md), we will explore how to interact with R.




## Syllabus

1. 