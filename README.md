## Prerequisites

Before we can get started, please install R and work through the next sections to familiarize with some concepts.

### Install or Update R 

If you have no installation of R, install the latest version from [https://cran.rstudio.com/](https://cran.rstudio.com/). 

* Some hints for installing R under [macOS](misc_notes#installation-under-macos).

Most of the code will probably run also with older versions of R. However, if you decide to update R, beware that  you will need to re-install all packages.

### Install or Update R Packages

Once you installed R, please run the following code _line by line_.

``` r
update.packages()

install.packages("tidyverse")
install.packages("magrittr")
install.packages("data.table")

packageVersion("tidyr")
```

Make sure that you have `tidyr` version ‘1.0.0’ installed. 

### Installing RStudio

We will be using an Integrated Development Environment (IDE) called **RStudio** to interact with R. However, there is nothing to prohibit using R at the command line or in some other interface. You can download RStudio for free from [https://rstudio.com/](https://rstudio.com/products/rstudio/download/#download).

> The RStudio GUI has multiple ‘panes’. Except for the ‘Console’ pane (by default in the lower left corner), the other panes (‘Environment’, ‘History’, ‘Files’, ‘Plots’, ‘Packages’, ‘Help’ etc.) are simply for convenience. If you choose to run R outside RStudio, the interaction will be _identical_ to working in the ‘Console’ pane.

### Getting Started

The following introductions are intended to give you some ‘background’ of R. So, you do not need to understand every line of code or recall everything from the lessons. Certainly, some things might seem puzzling right now, but this will soon disappear.

At the end of each introduction, you will find some questions for self-study.

* [Introductory Part 1](part_01-basic_interactions.md) will explore how to interact with R. _(Recommended.)_

* [Introductory Part 2](part_02-data_structures.md) will introduce some common data types in R. _(Recommended.)_

* [Introductory Part 3](part_03-working_with_strings.md) will explore how to work with text in R. _(Optional.)_

You may wish to consult the [answer key](answer_key) on certain problems.

A cheat sheet for the introduction in [RMD](part_00-cheat_sheet.Rmd) or [DOCX](part_00-cheat_sheet.docx) format at your disposal.

## Syllabus

We focus on using packages from the [`tidyverse`](https://www.tidyverse.org).

1. [Working with Tables](part_10-working_with_tables.md)
    * Importing and Tidying Tabulated Data
    * Manipulating and Creating Rows and Columns
    * Summarizing Tabulated Data
2. Making Figures
3. Good Practices
4. Import, Export, Save

## Acknowledgements

Some of the material has been adapted from Sean Davi's [Introduction to R](https://seandavi.github.io/ITR/index.html) and Patrick Burn's [Impatient R](https://www.burns-stat.com/documents/tutorials/impatient-r/). 