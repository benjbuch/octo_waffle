Working with Tables
================

A ‘table’—in R called a `data.frame`—is organized in columns and rows.
Each column of a `data.frame` can hold only a *single* [data
type](part_02-data_structures.md#data-types-in-r), but columns can have
*different* data types. `data.frame` objects are thus created *by
column*. Technically speaking, a [`data.frame` is a
`list`](part_02-data_structures.html#advanced-data-structures-classes)
in which each element of the list is an atomic vector of the same
length.

There are (at least) three different flavours to work with a
`data.frame` objects in R:

1.  Base R uses `list`-like syntax; this can however easily get
    cumbersome to work with.
2.  The\*`data.table` package uses SQL-like syntax to access
    `data.table` objects, a class similar to `data.frame`, but built for
    maximum efficiency.
3.  The `dplyr` package uses language-like syntax to describe the
    actions applied to `tibble` objects, another class similar to
    `data.frame`.

We will focus on using `dplyr`, which is part of the `tidyverse`.

``` r
library(tidyverse)
```

    ## ── Attaching packages ──────────────────────────────────────────────────────────────────────────── tidyverse 1.2.1 ──

    ## ✔ ggplot2 3.2.1     ✔ purrr   0.3.3
    ## ✔ tibble  2.1.3     ✔ dplyr   0.8.3
    ## ✔ tidyr   1.0.0     ✔ stringr 1.4.0
    ## ✔ readr   1.3.1     ✔ forcats 0.4.0

    ## ── Conflicts ─────────────────────────────────────────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

> Note that the `tidyverse` makes extensive use of ‘quasiquotation’.
> This means that you can refer e.g. to column names (which are usually
> of type `character`) without using quotation marks (`""` or `''`). But
> beware, some arguments accept `character` vectors only. These must be
> quoted.

> Related to this, you might see underscore versions of some `tidyverse`
> commands popping up in RStudio’s command completion such as `spread_`
> for `spread`. Ignore them (and any resource on the internet that urges
> you to use them), they are superfluous
> (`` ?tidyr::`deprecated-se` ``).

The `tidyverse` also imports [the pipe
`%>%`](part_01-basic_interactions.html#using-a-pipe) from the `magrittr`
package.

## Motivation: A Typical Table in Biosciences

*“The plate is the experimental default table in the Biosciences.”*

In the worst case, the data acquired from each position (well, spot,
etc.) on the plate was saved like this:

|          |   A   |    B    |    C    |    D    |    E    |
| :------: | :---: | :-----: | :-----: | :-----: | :-----: |
| <b>1</b> | 0.016 |  0.941  | \-0.307 | \-0.002 |  0.007  |
| <b>2</b> | 0.049 | \-0.063 |  0.301  |  0.150  | \-0.704 |
| <b>3</b> | 6.477 |  3.839  |  3.064  |  0.479  |  0.182  |
| <b>4</b> | 5.755 |  3.728  |  2.153  | \-0.080 |  0.204  |

Certainly, the sample assignments were documented (somewhere), so that
we know *the actual data* should be annotated like this.

|                     |                     | conc\_1 | conc\_2 | conc\_3 | conc\_4 | conc\_0 |
| :-----------------: | :-----------------: | :-----: | :-----: | :-----: | :-----: | :-----: |
|   <b>control</b>    | <b>replicate\_1</b> |  0.016  |  0.941  | \-0.307 | \-0.002 |  0.007  |
|   <b>control</b>    | <b>replicate\_2</b> |  0.049  | \-0.063 |  0.301  |  0.150  | \-0.704 |
| <b>treatment\_A</b> | <b>replicate\_1</b> |  6.477  |  3.839  |  3.064  |  0.479  |  0.182  |
| <b>treatment\_A</b> | <b>replicate\_2</b> |  5.755  |  3.728  |  2.153  | \-0.080 |  0.204  |

Imagine this data appearing in some spreadsheet software.

1.  How easily could you incorporate the concentrations actually used in
    the experiment, which were conc\_1 = 1000 µM, conc\_2 = 100 µM,
    conc\_3 = 10 µM, conc\_4 = 1 µM, conc\_0 = 0 µM?
2.  Would you be able to calculate the mean and standard deviation of
    the replicates for all samples?
3.  Would it take long to plot the dependence of the measured value on
    the concentration?
4.  Could you easily exclude a set of replicates and check the outcome?
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

The data for this plate has been saved in a file called ‘plates.RData’.
When you open this file, there should be (at least) one object called
`plate_1` and a named vector called `dose` in the ‘Global Environment’.

``` r
load(file = "./part_10-working_with_tables_files/plates.RData")
```

  - Which class and data type do these objects belong to?

## Tidy Data

Data can be tabulated in one of two ways: A tidy and a messy one.

> If each variable forms a column, and each row represents a single
> observation, and each cell a single value, we refer to this as **tidy
> data**.

Having your data tidied is crucial for facilitating data manipulation,
modelling, and visualization\!

Here are some guidelines:

1.  Don’t use values as column headers, all column headers should be
    variable names.
2.  Don’t put multiple variables in a single column.
3.  Don’t store variables in both rows and columns.
4.  Avoid storing multiple types of observational units in the same
    table.

Typically, if the table with your data is wider than long (if it is in
the ‘wide format’), it’s likely to encounter messy data. For example,
`plate_1` is in one (of many possible) messy formats: The concentration
is spread along the header row, not in a separate column.

A tidyer ‘wide format’ of the same data would look like that:

|    replicate\_id    | concentration  | control | treatment\_A |
| :-----------------: | :------------: | :-----: | :----------: |
| <b>replicate\_1</b> | <b>conc\_1</b> |  0.016  |    6.477     |
| <b>replicate\_1</b> | <b>conc\_2</b> |  0.941  |    3.839     |
| <b>replicate\_1</b> | <b>conc\_3</b> | \-0.307 |    3.064     |
| <b>replicate\_1</b> | <b>conc\_4</b> | \-0.002 |    0.479     |
| <b>replicate\_1</b> | <b>conc\_0</b> |  0.007  |    0.182     |
| <b>replicate\_2</b> | <b>conc\_1</b> |  0.049  |    5.755     |
| <b>replicate\_2</b> | <b>conc\_2</b> | \-0.063 |    3.728     |

*(table abridged)*

The longer the table, the tidyer the data. Here is the ‘long format’ of
`plate_1`:

|   sample\_id   |    replicate\_id    | concentration  | intensity |
| :------------: | :-----------------: | :------------: | :-------: |
| <b>control</b> | <b>replicate\_1</b> | <b>conc\_1</b> |   0.016   |
| <b>control</b> | <b>replicate\_1</b> | <b>conc\_2</b> |   0.941   |
| <b>control</b> | <b>replicate\_1</b> | <b>conc\_3</b> |  \-0.307  |
| <b>control</b> | <b>replicate\_1</b> | <b>conc\_4</b> |  \-0.002  |
| <b>control</b> | <b>replicate\_1</b> | <b>conc\_0</b> |   0.007   |
| <b>control</b> | <b>replicate\_2</b> | <b>conc\_1</b> |   0.049   |
| <b>control</b> | <b>replicate\_2</b> | <b>conc\_2</b> |  \-0.063  |
| <b>control</b> | <b>replicate\_2</b> | <b>conc\_3</b> |   0.301   |
| <b>control</b> | <b>replicate\_2</b> | <b>conc\_4</b> |   0.150   |

*(table abridged)*

The `tidyr` package (part of the `tidyverse`) provides functions to help
you tidy messy data. In this R session, the package has already been
attached to your namespace.

### Interconverting Wide and Long Table Formats

The degree to which you want to make your table ‘longer’ than ‘wide’
depends on the manipulation you want to perform. Typically, the ‘longest
format’ serves as the linchpin to produce the ‘wider formats’.

For your convenience, this is an overview of the different functions
that are around to interconvert wide and long table formats. (In case
you encounter one of these or need help on related topics.)

| action                           | `tidyr` ≥ v1.0   | `tidyr` \< v1.0 | `data.table` | `reshape`/`reshape2` |
| -------------------------------- | ---------------- | --------------- | ------------ | -------------------- |
| make wide table long (‘melting’) | `pivot_longer()` | `gather()`      | `melt()`     | `melt()`             |
| make long table wide (‘casting’) | `pivot_wider()`  | `spread()`      | `dcast()`    | `acast()`, `dcast()` |

As many people don’t find the other function names intuitive, we will
use `tidyr::pivot_longer(...)` and `tidyr::pivot_wider(...)`.

``` r
plate_1 %>% 
  # make wide table long
  pivot_longer(cols = conc_1:conc_0,        # selection of columns
               names_to  = "concentration", # name that describes the parameter in the selected headers
               values_to = "intensity"      # name that describes the value in the selected cells
  ) %>% 
  # print only top rows
  head(3)
```

    ## # A tibble: 3 x 4
    ##   sample_id replicate_id concentration intensity
    ##   <chr>     <chr>        <chr>             <dbl>
    ## 1 control   replicate_1  conc_1            0.016
    ## 2 control   replicate_1  conc_2            0.941
    ## 3 control   replicate_1  conc_3           -0.307

Instead of selecting the columns `conc_1:conc_0`, we could have
specified the columns which *not* to gather; these are somtimes called
the ‘identifying (id) columns’. The following commands evaluate to the
same result.

``` r
plate_1 %>% 
  # select by column index
  pivot_longer(cols = 3:7)

plate_1 %>% 
  # select by name range
  pivot_longer(cols = conc_1:conc_0)

plate_1 %>% 
  # select explicitly positive
  pivot_longer(cols = c("conc_1", "conc_2", "conc_3", "conc_4", "conc_0"))

plate_1 %>% 
  # select explicitly negative
  pivot_longer(cols = -c("sample_id", "replicate_id"))

plate_1 %>% 
  # select using pattern
  pivot_longer(cols = starts_with("conc"))
```

We will see more ways to select column names later.

For now, let’s next spread the gathered representation of `plate_1` into
a wide format\! Maybe, we would like to see the measured intensities for
`control` and `treatment_A` side-by-side given each concentration and
the replicate.

``` r
plate_1 %>% 
  # make wide table long
  pivot_longer(conc_1:conc_0, names_to = "concentration", values_to = "intensity") %>% 
  # make long table wide
  pivot_wider(names_from  = "sample_id", # column to use content as new column headers
              values_from = "intensity"  # column to use content to populate the new cells with
             ) %>% 
  # print only top rows
  head(3) 
```

    ## # A tibble: 3 x 4
    ##   replicate_id concentration control treatment_A
    ##   <chr>        <chr>           <dbl>       <dbl>
    ## 1 replicate_1  conc_1          0.016        6.48
    ## 2 replicate_1  conc_2          0.941        3.84
    ## 3 replicate_1  conc_3         -0.307        3.06

### Uniting and Splitting Columns

The `tidyr::unite(...)` function takes multiple column names and pastes
the column contents together.

``` r
plate_1 %>% 
  unite(col = "experiment",      # column to create
        sample_id, replicate_id, # columns to paste, can be unquoted
        sep = "/"                # separator, "_" is the default
  ) %>% 
  # print only top rows
  head(3)
```

    ##                experiment conc_1 conc_2 conc_3 conc_4 conc_0
    ## 1     control/replicate_1  0.016  0.941 -0.307 -0.002  0.007
    ## 2     control/replicate_2  0.049 -0.063  0.301  0.150 -0.704
    ## 3 treatment_A/replicate_1  6.477  3.839  3.064  0.479  0.182

The `tidyr::separate(...)` function does the reverse. By default, any
non-alphanumeric character will be used to split the column. Note that
missing pieces will be replaced with `NA`.

``` r
plate_1 %>% 
  separate(col  = sample_id,                 # column to split, can be unquoted
           into = c("treatment", "specimen") # names of the new columns
  ) %>% 
  # print only top rows
  head(3)
```

    ## Warning: Expected 2 pieces. Missing pieces filled with `NA` in 2 rows [1,
    ## 2].

    ##   treatment specimen replicate_id conc_1 conc_2 conc_3 conc_4 conc_0
    ## 1   control     <NA>  replicate_1  0.016  0.941 -0.307 -0.002  0.007
    ## 2   control     <NA>  replicate_2  0.049 -0.063  0.301  0.150 -0.704
    ## 3 treatment        A  replicate_1  6.477  3.839  3.064  0.479  0.182

``` r
plate_1 %>% 
  unite("experiment", sample_id, replicate_id, sep = "/") %>% 
  separate(experiment, into = str_c("piece_", 1:5)) %>% 
  # print only top rows
  head(3)
```

    ## Warning: Expected 5 pieces. Missing pieces filled with `NA` in 4 rows [1,
    ## 2, 3, 4].

    ##     piece_1   piece_2   piece_3 piece_4 piece_5 conc_1 conc_2 conc_3
    ## 1   control replicate         1    <NA>    <NA>  0.016  0.941 -0.307
    ## 2   control replicate         2    <NA>    <NA>  0.049 -0.063  0.301
    ## 3 treatment         A replicate       1    <NA>  6.477  3.839  3.064
    ##   conc_4 conc_0
    ## 1 -0.002  0.007
    ## 2  0.150 -0.704
    ## 3  0.479  0.182
