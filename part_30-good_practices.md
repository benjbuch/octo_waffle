Good Practices
================

  - [Code Documentation](#code-documentation)
      - [Comments, Spaces, Names](#comments-spaces-names)
      - [R Scripts vs R Markdown
        Documents](#r-scripts-vs-r-markdown-documents)
      - [Version Control](#version-control)
  - [Coding](#coding)
  - [Project Organization](#project-organization)

## Code Documentation

> “Like all programming languages, R isn’t just meant to be read by a
> computer, it’s also meant to be read by other humans—or very
> well-trained dolphins.” ([Nathaniel D. Phillips. *YaRrr\! The Pirate’s
> Guide to R,*
> 2018.](https://bookdown.org/ndphillips/YaRrr/a-brief-style-guide-commenting-and-spacing.html))

### Comments, Spaces, Names

*Use comments*

  - to remind you (and others) of why you did something that way,
  - to remind you (and others) of which ways it did not work (and
    potentially why),
  - to structure your code.

Here is an example how an R script could look like.

``` r
# 
#  title: My nicely commented R script
# author: Pirate Jack
#   date: None Today 
# 

# Step 1: Load Packages ----

library(tidyverse)

# Step 2: Load and Tidy Data ----

island_data <- data.frame(area = datasets::islands) %>% 
  # (whatever you say)
  rownames_to_column(var = "name")

# Step 3: Analysis ----

# What is the smallest island to go for treasure hunting?
island_data %>% filter(area == min(area))

# What are the four smallest islands?
island_data %>% top_n(4, -area)
```

Note that ending a commented line with more than four dashes (`----`),
hashes (`####`), or equal signs (`====`), will create an index entry in
RStudio.

*Use space\!* And include meaningful breakpoints for assigning objects.

``` r
# shitty looking code ...
data.frame(area=datasets::islands)%>%rownames_to_column(var="name")%>%filter(area==min(area))
```

*Use meaningful names\!*

``` r
# probably hard to remember after four months passed ...
isldt <- data.frame(a = datasets::islands) %>% 
  rownames_to_column(var = "nm")
```

Naming conventions in R are famously anarchic. Different styles for
naming objects may even coexist in the same package. However, it is a
good idea to get into the habit of consistent and clear writing in any
language, and R is no exception.

There are five naming conventions to choose from. So, have your pick\!

  - `alllowercase`,
  - `period.separated`,
  - `underscore_separated` (my favorite\!),
  - `lowerCamelCase`,
  - `UpperCamelCase`.

### R Scripts vs R Markdown Documents

Both, R scripts and R markdown documents serve as a means to write down
and reproduce your analysis (even after years to come). This makes your
analyses transparent, easily shareable, and reproducible. No matter
which project you are working on, use at least one means to document
your work.

| Parameter    | R Script    | R Markdown                                     |
| ------------ | ----------- | ---------------------------------------------- |
| organization | by comments | by sections, in chunks                         |
| description  | by comments | interspresed text and visuals (tables, graphs) |
| focus        | code        | documentation                                  |
| input/output | any kind    | any kind                                       |
| as pipline   | yes         | no/rarely                                      |

Here is a suggestions how to get the most of both:

1.  Use an external script for all substantial analysis tasks and to
    save a bunch of tables as `*.RData` or `*.rds` files; Then
2.  use a markdown in which you use the tables to make graphs or print
    summaries, on which you can comment.

This approach makes debugging and versioning of code (as R script) a lot
easier, and rendering the R markdown files a lot faster.

### Version Control

RStudio seamlessly integrates with version control systems such as `svn`
or `git`.

## Coding

*Parameterize\!*

It’s very helpful to define some values that you are frequently using
during the analysis or during plotting in the ‘header’ of your script.
This could be for example thresholds for filtering, frequently used
expressions, colors or shapes designated for a specific variable or
value.

*Rewrite\!*

After some rounds of evolution of your script, it might be helpful to go
through it from the top to the bottom and tidy up your code from times
to times.

## Project Organization

See [this chapter](https://r4ds.had.co.nz/workflow-projects.html) in G.
Grolemund’s and H. Wickham’s *R for Data Science*.
