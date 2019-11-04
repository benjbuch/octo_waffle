# R Workshop

## Prerequisites

Install **R** from
[https://cran.rstudio.com/](https://cran.rstudio.com). Any version
starting from R version 3.6.0 (2019-04-26) will (probably) do for this
workshop.

We will be using an Integrated Development Environment (IDE) called
**RStudio** to interact with R. However, there is nothing to prohibit
using R at the command line or in some other interface. You can download
RStudio for free from
[https://rstudio.com/products/rstudio/download/](https://rstudio.com/products/rstudio/download/#download).

The RStudio GUI has multiple ‘panes’. Except for the ‘Console’ pane (by
default in the lower left corner), the other panes (‘Environment’,
‘History’, ‘Files’, ‘Plots’, ‘Packages’, ‘Help’ etc.) are simply for
convenience. If you choose to run R outside RStudio, the interaction
will be *identical* to working in the ‘Console’ pane.

Once you installed R, please run the following lines of code from the
‘Console’ pane.

``` r
install.packages("tidyverse")
install.packages("data.table")
```

Note: To make use of `data.table`’s parallel processing capability, Mac
users will need a version of OpenMP to be installed on their machine.
They should follow the instructions on
[https://github.com/Rdatatable/data.table/wiki/Installation/](https://github.com/Rdatatable/data.table/wiki/Installation#openmp-enabled-compiler-for-mac).

## Syllabus

