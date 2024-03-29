Importing and Tidying Data
================

  - [Importing Data from Delimited
    Files](#importing-data-from-delimited-files)
      - [Importing Data from a Single
        File](#importing-data-from-a-single-file)
      - [Combining Data from Multiple Files with
        `row_bind`](#combining-data-from-multiple-files-with-row_bind)
      - [Combining Data with `join`](#combining-data-with-join)
      - [Importing Data from Excel](#importing-data-from-excel)
      - [Creating a `data.frame` from
        Scratch](#creating-a-data.frame-from-scratch)
  - [Tidying Data](#tidying-data)
      - [Interconverting Wide and Long Table
        Formats](#interconverting-wide-and-long-table-formats)
      - [Uniting and Splitting Columns](#uniting-and-splitting-columns)
  - [Hands-On Exercise](#hands-on-exercise)

-----

If you have not done so yet, please attach the `tidyverse`.

``` r
library(tidyverse)
```

## Importing Data from Delimited Files

A common standard to store tabulated data is as comma-separated values.
Typically such files end with the extension `.csv`. However, the
delimiters to represent the start of a new column could be any character
such as a semicolon or a white space (e.g. tab, blank).

### Importing Data from a Single File

The data of the plate mentioned in [this session’s
introduction](part_10-working_with_tables.html) was saved as a
comma-delimited file called ‘plate\_1.csv’. You can find it in the
sub-folder ‘part\_10-working\_with\_tables\_files’. We are going to
import this file using `readr::read_csv(file)`. This function will try
to guess the data type stored in each column.

> In the file path, `.` refers to the current working directory,
> `getwd()`, so that we can use relative paths.

``` r
plate_1 <- read_csv(file = "./part_10-working_with_tables_files/plate_1.csv")
```

    ## Parsed with column specification:
    ## cols(
    ##   A = col_double(),
    ##   B = col_double(),
    ##   C = col_double(),
    ##   D = col_double(),
    ##   E = col_double()
    ## )

``` r
plate_1
```

    ## # A tibble: 4 x 5
    ##        A     B      C      D      E
    ##    <dbl> <dbl>  <dbl>  <dbl>  <dbl>
    ## 1 -0.986 0.349 -0.377 -0.165 -0.402
    ## 2  0.48  0.23  -0.28   0.404  0.037
    ## 3  5.99  4.71   2.50   0.04  -0.211
    ## 4  6.52  3.57   1.76   1.48   0.081

The column names have been taken from the first line of the file (since
they were text). However, as we do know the actual names, we should use
them. They can be set either using `colnames(plate_1)<-` or during
import. In the latter case, we can (and should) skip the first line of
the file since it’s obsolete.

``` r
plate_1 <- read_csv(file = "./part_10-working_with_tables_files/plate_1.csv",
                    col_names = str_c("conc_", c(1:4, 0)), skip = 1)
```

    ## Parsed with column specification:
    ## cols(
    ##   conc_1 = col_double(),
    ##   conc_2 = col_double(),
    ##   conc_3 = col_double(),
    ##   conc_4 = col_double(),
    ##   conc_0 = col_double()
    ## )

The data we are typically dealing with is much too complex to rely on a
single row name only. To be useful, each parameter or variable should be
stored in a separate column.

So, let’s add one column specifying the sample type and one column
specifying the replicate number.

``` r
plate_1 <- plate_1 %>% 
  add_column(sample_id = rep(c("control", "treatment_A"), each = 2),
             replicate_id = rep(str_c("replicate_", c(1, 2)), times = 2),
             # insert before first column; default after last column
             .before = 1)
plate_1
```

    ## # A tibble: 4 x 7
    ##   sample_id   replicate_id conc_1 conc_2 conc_3 conc_4 conc_0
    ##   <chr>       <chr>         <dbl>  <dbl>  <dbl>  <dbl>  <dbl>
    ## 1 control     replicate_1  -0.986  0.349 -0.377 -0.165 -0.402
    ## 2 control     replicate_2   0.48   0.23  -0.28   0.404  0.037
    ## 3 treatment_A replicate_1   5.99   4.71   2.50   0.04  -0.211
    ## 4 treatment_A replicate_2   6.52   3.57   1.76   1.48   0.081

If you had (for some reason) rownames in a `data.frame` that you want to
keep in the `tidyverse`, use `dplyr::rownames_to_column()`.

### Combining Data from Multiple Files with `row_bind`

A nice feature of using a programmatic approach for data analysis is
that it allows retrieving (systematically) many files from multiple
locations on the harddrive (e.g. reflecting multiple experiments) and to
analyze them either in identical ways one by one, or to combine them for
analysis.

Certainly, we could import each file individually …

``` r
plate_2 <- read_csv(file = "./part_10-working_with_tables_files/plate_2.csv")
plate_3 <- read_csv(file = "./part_10-working_with_tables_files/plate_3.csv")
```

However, if we have many files to analyse, we get easily lost. A more
amenable approach is using `list.files(...)` to create a list
(`character` vector) of all files in a directory and apply regular
expressions to specify the files we would like to include.

``` r
plate_files.paths <- list.files(path = "./part_10-working_with_tables_files",
                                pattern = "plate_[^1]", full.names = TRUE)

plate_files <- sapply(plate_files.paths, read_csv, simplify = FALSE, USE.NAMES = TRUE)
```

    ## Parsed with column specification:
    ## cols(
    ##   sample_id = col_character(),
    ##   replicate_id = col_character(),
    ##   conc_1 = col_double(),
    ##   conc_2 = col_double(),
    ##   conc_3 = col_double(),
    ##   conc_4 = col_double(),
    ##   conc_0 = col_double()
    ## )

    ## Parsed with column specification:
    ## cols(
    ##   sample_id = col_character(),
    ##   replicate_id = col_character(),
    ##   conc_1 = col_double(),
    ##   conc_3 = col_double(),
    ##   conc_2 = col_double(),
    ##   conc_4 = col_double(),
    ##   conc_0 = col_double()
    ## )

``` r
plate_files
```

    ## $`./part_10-working_with_tables_files/plate_2.csv`
    ## # A tibble: 4 x 7
    ##   sample_id   replicate_id conc_1 conc_2 conc_3 conc_4 conc_0
    ##   <chr>       <chr>         <dbl>  <dbl>  <dbl>  <dbl>  <dbl>
    ## 1 treatment_B replicate_1   140.    83.2  41.3   12.8  -0.273
    ## 2 treatment_B replicate_2   118.    75.5  43.5   12.7  -0.118
    ## 3 treatment_C replicate_1    33.8   22.3   9.78   2.60  0.493
    ## 4 treatment_C replicate_2    29.5   17.7   8.02   3.09  0.753
    ## 
    ## $`./part_10-working_with_tables_files/plate_3.csv`
    ## # A tibble: 2 x 7
    ##   sample_id   replicate_id conc_1 conc_3 conc_2 conc_4 conc_0
    ##   <chr>       <chr>         <dbl>  <dbl>  <dbl>  <dbl>  <dbl>
    ## 1 treatment_D replicate_1  -0.219  0.79  -0.432  0.261  0.079
    ## 2 treatment_D replicate_2  -0.148  0.669  0.585 -0.027  0.601

The `plate_files` object is a `list` with two `tibble` objects
(`dplyr`’s way of `data.frame`). Apparently, two columns have been
swapped in `plate_3`, which could cause problems if we just pasted them
mindlessly below each other.

Luckily, R pays attention for us.

``` r
plate_files %>% bind_rows(.id = "file")
```

    ## # A tibble: 6 x 8
    ##   file           sample_id replicate_id  conc_1 conc_2 conc_3 conc_4 conc_0
    ##   <chr>          <chr>     <chr>          <dbl>  <dbl>  <dbl>  <dbl>  <dbl>
    ## 1 ./part_10-wor… treatmen… replicate_1  140.    83.2   41.3   12.8   -0.273
    ## 2 ./part_10-wor… treatmen… replicate_2  118.    75.5   43.5   12.7   -0.118
    ## 3 ./part_10-wor… treatmen… replicate_1   33.8   22.3    9.78   2.60   0.493
    ## 4 ./part_10-wor… treatmen… replicate_2   29.5   17.7    8.02   3.09   0.753
    ## 5 ./part_10-wor… treatmen… replicate_1   -0.219 -0.432  0.79   0.261  0.079
    ## 6 ./part_10-wor… treatmen… replicate_2   -0.148  0.585  0.669 -0.027  0.601

We have kept track of the file names (or even file paths) by specifying
an `.id` column called `"file"` from the names of the `tibble` `list`.
If need be, we could extract metadata stored in the file name or file
path by creating new columns. (Details in the next session.)

``` r
plate_files %>% bind_rows(.id = "file") %>% mutate(file = str_extract(file, "plate_[0-9]+"))
```

    ## # A tibble: 6 x 8
    ##   file    sample_id   replicate_id  conc_1 conc_2 conc_3 conc_4 conc_0
    ##   <chr>   <chr>       <chr>          <dbl>  <dbl>  <dbl>  <dbl>  <dbl>
    ## 1 plate_2 treatment_B replicate_1  140.    83.2   41.3   12.8   -0.273
    ## 2 plate_2 treatment_B replicate_2  118.    75.5   43.5   12.7   -0.118
    ## 3 plate_2 treatment_C replicate_1   33.8   22.3    9.78   2.60   0.493
    ## 4 plate_2 treatment_C replicate_2   29.5   17.7    8.02   3.09   0.753
    ## 5 plate_3 treatment_D replicate_1   -0.219 -0.432  0.79   0.261  0.079
    ## 6 plate_3 treatment_D replicate_2   -0.148  0.585  0.669 -0.027  0.601

For the moment, let’s combine the new data with `plate_1` without the
file paths and call the object `plate_data`.

``` r
plate_data <- bind_rows(plate_1, plate_files)

# remove objects no longer needed
rm(plate_files, plate_files.paths)
```

### Combining Data with `join`

We have seen that the command to combine rows is `dpylr::bind_rows(...)`
(and `rbind(...)` in base R). Likewise, there is `dplyr::bind_cols(...)`
(or `cbind(...)` in base R) to combine columns. However, we must be
absolutely sure that the rows align\!

``` r
cbind(
  # half plate with conc_1 measurments
  plate_data %>% select(sample_id, replicate_id, conc_1),
  # half plate with conc_2 measurements
  plate_data %>% select(sample_id, replicate_id, conc_2)
)
```

In the following case, this fails. (Don’t care too much about how to
create the randomized halfs.)

``` r
# half plate with conc_1 measurments
half_1 <- plate_data %>%
            select(sample_id, replicate_id, conc_1)
# half plate with conc_2 measurements
half_2 <- plate_data %>% sample_n(nrow(.)) %>% 
            select(sample_id, replicate_id, conc_2)

# this will be wrong
bind_cols(half_1, half_2)
```

    ## # A tibble: 10 x 6
    ##    sample_id   replicate_id  conc_1 sample_id1  replicate_id1 conc_2
    ##    <chr>       <chr>          <dbl> <chr>       <chr>          <dbl>
    ##  1 control     replicate_1   -0.986 treatment_B replicate_1   83.2  
    ##  2 control     replicate_2    0.48  treatment_D replicate_2    0.585
    ##  3 treatment_A replicate_1    5.99  control     replicate_2    0.23 
    ##  4 treatment_A replicate_2    6.52  treatment_A replicate_1    4.71 
    ##  5 treatment_B replicate_1  140.    treatment_B replicate_2   75.5  
    ##  6 treatment_B replicate_2  118.    treatment_D replicate_1   -0.432
    ##  7 treatment_C replicate_1   33.8   control     replicate_1    0.349
    ##  8 treatment_C replicate_2   29.5   treatment_C replicate_1   22.3  
    ##  9 treatment_D replicate_1   -0.219 treatment_C replicate_2   17.7  
    ## 10 treatment_D replicate_2   -0.148 treatment_A replicate_2    3.57

To properly merge `half_1` and `half_2`, we need to merge the tables
based on the colums `sample_id` and `replicate_id`. This is what
`dplyr::inner_join(x, y)` does.

``` r
# this will be correct
inner_join(half_1, half_2, by = c("sample_id", "replicate_id"))
```

    ## # A tibble: 10 x 4
    ##    sample_id   replicate_id  conc_1 conc_2
    ##    <chr>       <chr>          <dbl>  <dbl>
    ##  1 control     replicate_1   -0.986  0.349
    ##  2 control     replicate_2    0.48   0.23 
    ##  3 treatment_A replicate_1    5.99   4.71 
    ##  4 treatment_A replicate_2    6.52   3.57 
    ##  5 treatment_B replicate_1  140.    83.2  
    ##  6 treatment_B replicate_2  118.    75.5  
    ##  7 treatment_C replicate_1   33.8   22.3  
    ##  8 treatment_C replicate_2   29.5   17.7  
    ##  9 treatment_D replicate_1   -0.219 -0.432
    ## 10 treatment_D replicate_2   -0.148  0.585

If you want to keep rows that are in one table, but not in another,
there are also the functions `dpylr::left_join(x, y)` and
`dplyr::right_join(x, y)`. Having both options is useful when you think
of piping.

``` r
# remove objects no longer needed
rm(half_1, half_2)
```

### Importing Data from Excel

To read data from Excel, you can use `readxl::read_excel(file)`. The
`readxl` is a package of the `tidyverse`.

Here is an example with `plate_1.xlsx`, which contains the same data put
somewhere on the worksheet. By default, `readxl::read_excel(file)` uses
the smallest rectangle that contains the non-empty cells.

``` r
readxl::read_excel("./part_10-working_with_tables_files/plate_1.xlsx")
```

    ## New names:
    ## * `` -> ...1
    ## * `` -> ...2

    ## # A tibble: 4 x 7
    ##   ...1        ...2        conc_1 conc_2 conc_3 conc_4 conc_5
    ##   <chr>       <chr>        <dbl>  <dbl>  <dbl>  <dbl>  <dbl>
    ## 1 control     replicate_1 -0.986  0.349 -0.377 -0.165 -0.402
    ## 2 control     replicate_2  0.48   0.23  -0.28   0.404  0.037
    ## 3 treatment_A replicate_1  5.99   4.71   2.50   0.04  -0.211
    ## 4 treatment_A replicate_2  6.52   3.57   1.76   1.48   0.081

The blank column headers will get a new, unique name (e.g. `...1`,
`...2` etc.) and the content of merged cells will be assigned to the
top-most left cell of the area.

However, I would disencourage to rely on routine importing data from
Excel workbooks since `.xls` and `.xlsx` are neither open
(i.e. disclosed), nor standardized file formats. This means they can be
changed by Microsoft anytime and may consequently not be (immediately)
supported by other software.

Another problem you might be facing is that within Excel, it is tempting
to put multiple tables with different data side-by-side in a single
worksheet. To import such worksheets will require a lot of polishing
from your side after.

> If you plan to import from Excel, keep it simple and put one set of
> data per sheet. Ideally, export as `.csv`.

### Creating a `data.frame` from Scratch

Sometimes it’s necessary to quickly make a `data.frame` or related
objects.

This base R approach works *always*.

``` r
set.seed(181096)
```

``` r
# simulate some data
status <- factor(rep(c("smoker", "non-smoker"), each = 50))
risk_non_smoker <- rnorm(50)
risk_smoker <- 2 + risk_non_smoker

# create a data.frame by column
smoking_risk <- data.frame(status = status, risk = c(risk_smoker, risk_non_smoker))

head(smoking_risk)
```

    ##   status       risk
    ## 1 smoker  2.6992083
    ## 2 smoker -0.4244009
    ## 3 smoker  2.1020269
    ## 4 smoker  0.4823049
    ## 5 smoker  2.0833528
    ## 6 smoker  2.4069171

``` r
# create a quick plot
plot(risk ~ status, data = smoking_risk)
```

![](part_11-tidying_tables_files/figure-gfm/unnamed-chunk-18-1.png)<!-- -->

Remember that a `data.frame` is basically a `list` of vectors (of
identical length) that can contain a different type of data each. This
is why tables (if not imported) are created by column from vectors.

The `tidyverse` provides and uses a special case of the `data.frame`
class, which is the `tibble`. It has a couple of advantages (see
`?tibble::tibble`).

`tibble` objects are created like `data.frame` objects.

``` r
smoking_risk <- tibble(status = status, risk = c(risk_smoker, risk_non_smoker))

smoking_risk # no head() call required, prints only top rows
```

    ## # A tibble: 100 x 2
    ##    status   risk
    ##    <fct>   <dbl>
    ##  1 smoker  2.70 
    ##  2 smoker -0.424
    ##  3 smoker  2.10 
    ##  4 smoker  0.482
    ##  5 smoker  2.08 
    ##  6 smoker  2.41 
    ##  7 smoker  2.48 
    ##  8 smoker  2.11 
    ##  9 smoker  1.63 
    ## 10 smoker  1.34 
    ## # … with 90 more rows

If for some reason you want to create a `tibble` by rows, use
`tibble::tribble`.

``` r
tribble(
  ~colA, ~colB,
  "a",   1,
  "b",   2,
  "c",   3
)
```

    ## # A tibble: 3 x 2
    ##   colA   colB
    ##   <chr> <dbl>
    ## 1 a         1
    ## 2 b         2
    ## 3 c         3

## Tidying Data

Data can be tabulated in one of two ways: A tidy and a messy one.

> If each variable forms a column, each row represents an observation,
> and each cell a single value, we have **‘tidy data’**.

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

A tidier ‘wide format’ of the same data would look like that:

|    replicate\_id    | concentration  | control | treatment\_A |
| :-----------------: | :------------: | :-----: | :----------: |
| <b>replicate\_1</b> | <b>conc\_1</b> | \-0.986 |    5.992     |
| <b>replicate\_1</b> | <b>conc\_2</b> |  0.349  |    4.711     |
| <b>replicate\_1</b> | <b>conc\_3</b> | \-0.377 |    2.504     |
| <b>replicate\_1</b> | <b>conc\_4</b> | \-0.165 |    0.040     |
| <b>replicate\_1</b> | <b>conc\_0</b> | \-0.402 |   \-0.211    |
| <b>replicate\_2</b> | <b>conc\_1</b> |  0.480  |    6.521     |
| <b>replicate\_2</b> | <b>conc\_2</b> |  0.230  |    3.574     |

*(table abridged)*

The longer the table, the tidier the data. Here is the ‘long format’ of
`plate_1`:

|   sample\_id   |    replicate\_id    | concentration  | intensity |
| :------------: | :-----------------: | :------------: | :-------: |
| <b>control</b> | <b>replicate\_1</b> | <b>conc\_1</b> |  \-0.986  |
| <b>control</b> | <b>replicate\_1</b> | <b>conc\_2</b> |   0.349   |
| <b>control</b> | <b>replicate\_1</b> | <b>conc\_3</b> |  \-0.377  |
| <b>control</b> | <b>replicate\_1</b> | <b>conc\_4</b> |  \-0.165  |
| <b>control</b> | <b>replicate\_1</b> | <b>conc\_0</b> |  \-0.402  |
| <b>control</b> | <b>replicate\_2</b> | <b>conc\_1</b> |   0.480   |
| <b>control</b> | <b>replicate\_2</b> | <b>conc\_2</b> |   0.230   |
| <b>control</b> | <b>replicate\_2</b> | <b>conc\_3</b> |  \-0.280  |
| <b>control</b> | <b>replicate\_2</b> | <b>conc\_4</b> |   0.404   |

*(table abridged)*

The `tidyr` package (part of the `tidyverse`) provides functions to help
you tidy messy data.

### Interconverting Wide and Long Table Formats

The degree to which you want to make your table ‘longer’ than ‘wide’
depends on the manipulation you want to perform. Typically, the ‘longest
format’ serves as the linchpin to produce the ‘wider formats’.

For your convenience, this is an overview of the different functions
that are around to interconvert wide and long table formats. (Including
other packages, in case you encounter one of these or need help on
related topics.)

| action                           | `tidyr` ≥ v1.0 | `tidyr` \< v1.0 | `data.table` | `reshape`/`reshape2` |
| -------------------------------- | -------------- | --------------- | ------------ | -------------------- |
| make wide table long (‘melting’) | `pivot_longer` | `gather`        | `melt`       | `melt`               |
| make long table wide (‘casting’) | `pivot_wider`  | `spread`        | `dcast`      | `acast`, `dcast`     |

As many people don’t find the other functions intuitive, we will use
`tidyr::pivot_longer` and `tidyr::pivot_wider`.

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
    ## 1 control   replicate_1  conc_1           -0.986
    ## 2 control   replicate_1  conc_2            0.349
    ## 3 control   replicate_1  conc_3           -0.377

Instead of selecting the columns `conc_1:conc_0`, we could have
specified the columns which *not* to gather; these are somtimes called
the ‘identifying (id) columns’. The following commands evaluate to the
same result.

``` r
plate_1 %>% 
  # select explicitly negative
  pivot_longer(cols = -c("sample_id", "replicate_id"))

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
  # select using pattern
  pivot_longer(cols = starts_with("conc"))
```

We discuss selecting columns by name later.

For now, let’s continue with widening/spreading/casting the
long/gathered/molten representation of `plate_1`\! Maybe, we would like
to see the measured intensities for `control` and `treatment_A`
side-by-side given each concentration and the replicate.

``` r
plate_1 %>% 
  # make wide table long
  pivot_longer(conc_1:conc_0, names_to = "concentration", values_to = "intensity") %>% 
  # make long table wide
  pivot_wider(names_from  = sample_id, # column to use content as new column headers
              values_from = intensity  # column to use content to populate the new cells with
             ) %>% 
  # print only top rows
  head(3) 
```

    ## # A tibble: 3 x 4
    ##   replicate_id concentration control treatment_A
    ##   <chr>        <chr>           <dbl>       <dbl>
    ## 1 replicate_1  conc_1         -0.986        5.99
    ## 2 replicate_1  conc_2          0.349        4.71
    ## 3 replicate_1  conc_3         -0.377        2.50

This format of `plate_1` could be useful if we wanted to subtract the
control measurements from the treated samples. Here,
`dplyr::mutate(...)` creates a new column or modifies an exisiting
column using the specified operations. (More on this later, too.)

``` r
plate_1 %>% 
  # make wide table long
  pivot_longer(conc_1:conc_0, names_to = "concentration", values_to = "intensity") %>% 
  # make long table wide
  pivot_wider(names_from = sample_id, values_from = intensity) -> plate_1.wide

plate_1.wide %>% 
  # make new column with corrected treatment_A values
  mutate(tratment_A.corrected = treatment_A - control) %>% head(3)
```

    ## # A tibble: 3 x 5
    ##   replicate_id concentration control treatment_A tratment_A.corrected
    ##   <chr>        <chr>           <dbl>       <dbl>                <dbl>
    ## 1 replicate_1  conc_1         -0.986        5.99                 6.98
    ## 2 replicate_1  conc_2          0.349        4.71                 4.36
    ## 3 replicate_1  conc_3         -0.377        2.50                 2.88

``` r
plate_1.wide %>% 
  # operate in place; not always advisable
  mutate(treatment_A = treatment_A - control) %>% head(3)
```

    ## # A tibble: 3 x 4
    ##   replicate_id concentration control treatment_A
    ##   <chr>        <chr>           <dbl>       <dbl>
    ## 1 replicate_1  conc_1         -0.986        6.98
    ## 2 replicate_1  conc_2          0.349        4.36
    ## 3 replicate_1  conc_3         -0.377        2.88

You can check out more examples in the `tidyr` [vignette on
pivoting](https://tidyr.tidyverse.org/dev/articles/pivot.html).

### Uniting and Splitting Columns

`tidyr::unite(...)` takes the values form multiple columns and pastes
them together side-by-side.

``` r
plate_1 %>% 
  unite(col = "experiment",      # column to create
        sample_id, replicate_id, # columns to paste, can be unquoted
        sep = "/"                # separator, "_" is the default
  ) %>% 
  # print only top rows
  head(3)
```

    ## # A tibble: 3 x 6
    ##   experiment              conc_1 conc_2 conc_3 conc_4 conc_0
    ##   <chr>                    <dbl>  <dbl>  <dbl>  <dbl>  <dbl>
    ## 1 control/replicate_1     -0.986  0.349 -0.377 -0.165 -0.402
    ## 2 control/replicate_2      0.48   0.23  -0.28   0.404  0.037
    ## 3 treatment_A/replicate_1  5.99   4.71   2.50   0.04  -0.211

The `tidyr::separate(column)` function does the reverse. By default, any
non-alphanumeric character will be used to split the column, which can
result in missing pieces to be replaced with `NA`.

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

    ## # A tibble: 3 x 8
    ##   treatment specimen replicate_id conc_1 conc_2 conc_3 conc_4 conc_0
    ##   <chr>     <chr>    <chr>         <dbl>  <dbl>  <dbl>  <dbl>  <dbl>
    ## 1 control   <NA>     replicate_1  -0.986  0.349 -0.377 -0.165 -0.402
    ## 2 control   <NA>     replicate_2   0.48   0.23  -0.28   0.404  0.037
    ## 3 treatment A        replicate_1   5.99   4.71   2.50   0.04  -0.211

``` r
plate_1 %>% 
  unite("experiment", sample_id, replicate_id, sep = "/") %>% 
  separate(experiment, into = str_c("piece_", 1:5)) %>% 
  # print only top rows
  head(3)
```

    ## Warning: Expected 5 pieces. Missing pieces filled with `NA` in 4 rows [1,
    ## 2, 3, 4].

    ## # A tibble: 3 x 10
    ##   piece_1 piece_2 piece_3 piece_4 piece_5 conc_1 conc_2 conc_3 conc_4
    ##   <chr>   <chr>   <chr>   <chr>   <chr>    <dbl>  <dbl>  <dbl>  <dbl>
    ## 1 control replic… 1       <NA>    <NA>    -0.986  0.349 -0.377 -0.165
    ## 2 control replic… 2       <NA>    <NA>     0.48   0.23  -0.28   0.404
    ## 3 treatm… A       replic… 1       <NA>     5.99   4.71   2.50   0.04 
    ## # … with 1 more variable: conc_0 <dbl>

## Hands-On Exercise

The data for another plate has been saved in a file called
‘plates.RData’. When you load this file, you should see a data set
called `plate_4` and a named vector called `dose` in the ‘Global
Environment’.

``` r
load(file = "./part_10-working_with_tables_files/plates.RData")
```

1.  Combine `plate_4` with `plate_data`.

2.  Tidy `plate_data` and replace the concentration with actual values.
