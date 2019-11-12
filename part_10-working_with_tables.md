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

|          |   A   |    B    |    C    |    D    |    E    |
| :------: | :---: | :-----: | :-----: | :-----: | :-----: |
| <b>1</b> | 0.016 |  0.941  | \-0.307 | \-0.002 |  0.007  |
| <b>2</b> | 0.049 | \-0.063 |  0.301  |  0.150  | \-0.704 |
| <b>3</b> | 6.477 |  3.839  |  3.064  |  0.479  |  0.182  |
| <b>4</b> | 5.755 |  3.728  |  2.153  | \-0.080 |  0.204  |

Certainly, the sample assignment was documented (somewhere), so that we
know *the actual data* should be annotated like this.

|                     |                     | conc\_1 | conc\_2 | conc\_3 | conc\_4 | conc\_0 |
| :-----------------: | :-----------------: | :-----: | :-----: | :-----: | :-----: | :-----: |
|   <b>control</b>    | <b>replicate\_1</b> |  0.016  |  0.941  | \-0.307 | \-0.002 |  0.007  |
|   <b>control</b>    | <b>replicate\_2</b> |  0.049  | \-0.063 |  0.301  |  0.150  | \-0.704 |
| <b>treatment\_A</b> | <b>replicate\_1</b> |  6.477  |  3.839  |  3.064  |  0.479  |  0.182  |
| <b>treatment\_A</b> | <b>replicate\_2</b> |  5.755  |  3.728  |  2.153  | \-0.080 |  0.204  |

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
|      control |          0 µM | \-0.348 | 0.503 |
|      control |          1 µM |   0.074 | 0.107 |
|      control |         10 µM | \-0.003 | 0.430 |
|      control |        100 µM |   0.439 | 0.710 |
|      control |       1000 µM |   0.032 | 0.023 |
| treatment\_A |          0 µM |   0.193 | 0.016 |
| treatment\_A |          1 µM |   0.199 | 0.395 |
| treatment\_A |         10 µM |   2.609 | 0.644 |
| treatment\_A |        100 µM |   3.784 | 0.078 |
| treatment\_A |       1000 µM |   6.116 | 0.511 |

… and six more lines for a plot.

![](part_10-working_with_tables_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

So, let’s start\!
