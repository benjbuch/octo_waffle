Working with Tables
================

A ‘table’—in R called a `data.frame`—is data organized in columns and
rows.

There are (at least) three different flavours to work with a
`data.frame` object in R:

1.  Base R uses `list`-like syntax, which can quickly get cumbersome to
    work with.
2.  The `data.table` package uses SQL-like syntax, which is built for
    maximum efficiency.
3.  The `dplyr` package uses language-like syntax to describe each
    action.

Each of these flavours comes with its own advantages and disadvantages.
Some people may prefer to work with one or the other and you may even
see mixed syntax being used. For this workshop, we will focus on using
`dplyr`, which is part of the `tidyverse`. And we try to be rather
puristic …

``` r
library(tidyverse)
```

    ## ── Attaching packages ────────────────────────────────────────────────────────────────────────────────── tidyverse 1.2.1 ──

    ## ✔ ggplot2 3.2.1     ✔ purrr   0.3.3
    ## ✔ tibble  2.1.3     ✔ dplyr   0.8.3
    ## ✔ tidyr   1.0.0     ✔ stringr 1.4.0
    ## ✔ readr   1.3.1     ✔ forcats 0.4.0

    ## ── Conflicts ───────────────────────────────────────────────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

Note that the `tidyverse` imports the [pipe
`%>%`](part_01-basic_interactions.html#using-a-pipe) from the `magrittr`
package.

## Motivation: A Typical Table in Biosciences

*“The plate is the experimental default table in the Biosciences.”*

In worst case, the data acquired from each position on the plate was
saved like this:

|          |    A    |   B   |    C    |    D    |    E    |
| :------: | :-----: | :---: | :-----: | :-----: | :-----: |
| <b>1</b> | \-0.986 | 0.349 | \-0.377 | \-0.165 | \-0.402 |
| <b>2</b> |  0.480  | 0.230 | \-0.280 |  0.404  |  0.037  |
| <b>3</b> |  5.992  | 4.711 |  2.504  |  0.040  | \-0.211 |
| <b>4</b> |  6.521  | 3.574 |  1.757  |  1.482  |  0.081  |

Certainly, the sample assignment was documented (somewhere), so that we
know *the actual data* should be annotated like this.

|                     |                     | conc\_1 | conc\_2 | conc\_3 | conc\_4 | conc\_0 |
| :-----------------: | :-----------------: | :-----: | :-----: | :-----: | :-----: | :-----: |
|   <b>control</b>    | <b>replicate\_1</b> | \-0.986 |  0.349  | \-0.377 | \-0.165 | \-0.402 |
|   <b>control</b>    | <b>replicate\_2</b> |  0.480  |  0.230  | \-0.280 |  0.404  |  0.037  |
| <b>treatment\_A</b> | <b>replicate\_1</b> |  5.992  |  4.711  |  2.504  |  0.040  | \-0.211 |
| <b>treatment\_A</b> | <b>replicate\_2</b> |  6.521  |  3.574  |  1.757  |  1.482  |  0.081  |

Imagine this data appearing in some spreadsheet software.

1.  Would you be able to calculate the mean and standard deviation over
    the replicates for all samples?
2.  How easily could you incorporate the concentrations used in the
    experiment, which were conc\_1 = 1000 µM, conc\_2 = 100 µM, conc\_3
    = 10 µM, conc\_4 = 1 µM, conc\_0 = 0 µM? Also for plotting?
3.  Could you quickly exclude a set of replicates and check for bias?
    How about adding another set of observations and answer all the same
    questions?

The following answer takes just four lines of code in R …

|   sample\_id | concentration |    mean |    sd |
| -----------: | ------------: | ------: | ----: |
|      control |          0 µM | \-0.183 | 0.310 |
|      control |          1 µM |   0.120 | 0.402 |
|      control |         10 µM | \-0.328 | 0.069 |
|      control |        100 µM |   0.290 | 0.084 |
|      control |       1000 µM | \-0.253 | 1.037 |
| treatment\_A |          0 µM | \-0.065 | 0.206 |
| treatment\_A |          1 µM |   0.761 | 1.020 |
| treatment\_A |         10 µM |   2.131 | 0.528 |
| treatment\_A |        100 µM |   4.143 | 0.804 |
| treatment\_A |       1000 µM |   6.256 | 0.374 |

… and six more lines for a plot.

![](part_10-working_with_tables_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

So, let’s start\!
