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
  - [Tidying Data](#tidying-data)
      - [Interconverting Wide and Long Table
        Formats](#interconverting-wide-and-long-table-formats)
      - [Uniting and Splitting Columns](#uniting-and-splitting-columns)
  - [Hands-On Excercise](#hands-on-excercise)

## Importing Data from Delimited Files

A common standard to represent tabulated data as the comma-separated
values, typically indicated by the extension `.csv`. However, the
delimiters to represent the begin of a new column can be anything such
as semicolons, white spaces (e.g. tab, blank).

### Importing Data from a Single File

The data of the plate mentioned in this session’s introduction was saved
in a comma-delimited file called ‘plate\_1.csv’. You can find it in the
sub-folder ‘part\_10-working\_with\_tables\_files’. We will import this
file using `readr::read_csv()`, which will try to guess the data type
stored in each column.

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
    ##       A      B      C      D      E
    ##   <dbl>  <dbl>  <dbl>  <dbl>  <dbl>
    ## 1 0.016  0.941 -0.307 -0.002  0.007
    ## 2 0.049 -0.063  0.301  0.15  -0.704
    ## 3 6.48   3.84   3.06   0.479  0.182
    ## 4 5.76   3.73   2.15  -0.08   0.204

The column names have been taken from the first line of the file (since
they were text). However, as we know the actual names, we should use
them. They can be set either using `colnames(plate_1)<-` or during
import. In the latter case, we can (and should) skip the first line of
the file.

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

Generally, it is disencouraged to rely on row names to work with complex
data sets since only one variable or parameter can be referenced. To be
useful, each parameter or variable should be stored in a separate
column. So, let’s add one column specifying the sample type and one
column specifying the replicate number.

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
    ## 1 control     replicate_1   0.016  0.941 -0.307 -0.002  0.007
    ## 2 control     replicate_2   0.049 -0.063  0.301  0.15  -0.704
    ## 3 treatment_A replicate_1   6.48   3.84   3.06   0.479  0.182
    ## 4 treatment_A replicate_2   5.76   3.73   2.15  -0.08   0.204

If you had (for some reason) rownames in your table that you want to
keep, use `dplyr::rownames_to_column()`.

### Combining Data from Multiple Files with `row_bind`

A nice feature of using a programmatic approach for data analysis is
that it allows retrieving (systematically) many files from multiple
locations on your harddrive (e.g. reflecting multiple experiments) and
to either analyze them in identical ways one by one, or to combine them
for analysis.

Certainly, we could import each file individually.

``` r
plate_2 <- read_csv(file = "./part_10-working_with_tables_files/plate_2.csv")
plate_3 <- read_csv(file = "./part_10-working_with_tables_files/plate_3.csv")
```

However, if we have many files to analyse, this approach can become
tedious. So, we use `list.files(...)` to create a list of all files that
we would like to include. These can even use regular expressions for
pattern matching.

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
    ## 1 treatment_B replicate_1   119.    79.9  42.1   11.4   0.618
    ## 2 treatment_B replicate_2   120.    79.5  41.8   12.4  -0.204
    ## 3 treatment_C replicate_1    30.3   19.9  10.4    2.68 -0.341
    ## 4 treatment_C replicate_2    30.1   19.6   9.76   2.70  1.05 
    ## 
    ## $`./part_10-working_with_tables_files/plate_3.csv`
    ## # A tibble: 2 x 7
    ##   sample_id   replicate_id conc_1 conc_3 conc_2 conc_4 conc_0
    ##   <chr>       <chr>         <dbl>  <dbl>  <dbl>  <dbl>  <dbl>
    ## 1 treatment_D replicate_1   0.203 -0.025  0.444 -0.129  0.535
    ## 2 treatment_D replicate_2   0.434 -0.269 -0.481  0.391  0.114

The `plate_files` object is a `list` with two `tibble` (`dplyr`’s way of
`data.frame`) objects. Apparently, two columns have been swapped in
`plate_3`, which could cause problems if we just pasted them. Luckily, R
pays attention for us.

``` r
plate_files %>% bind_rows(.id = "file")
```

    ## # A tibble: 6 x 8
    ##   file           sample_id replicate_id  conc_1 conc_2 conc_3 conc_4 conc_0
    ##   <chr>          <chr>     <chr>          <dbl>  <dbl>  <dbl>  <dbl>  <dbl>
    ## 1 ./part_10-wor… treatmen… replicate_1  119.    79.9   42.1   11.4    0.618
    ## 2 ./part_10-wor… treatmen… replicate_2  120.    79.5   41.8   12.4   -0.204
    ## 3 ./part_10-wor… treatmen… replicate_1   30.3   19.9   10.4    2.68  -0.341
    ## 4 ./part_10-wor… treatmen… replicate_2   30.1   19.6    9.76   2.70   1.05 
    ## 5 ./part_10-wor… treatmen… replicate_1    0.203  0.444 -0.025 -0.129  0.535
    ## 6 ./part_10-wor… treatmen… replicate_2    0.434 -0.481 -0.269  0.391  0.114

To be mentioned here, we also (could) keep track of the file names (or
paths) by specifying an `.id` column called `file`, which will take the
names of the `list`. If need be, we can then make use of metadata stored
in the file name or file path.

For the moment, let’s combine the data with `plate_1` and call the
object `plate_data`.

``` r
plate_data <- bind_rows(plate_1, plate_files)

# remove objects no longer needed
rm(plate_files, plate_files.paths)
```

### Combining Data with `join`

We have seen that the command to combine rows is `dpylr::bind_rows(...)`
in the `tidyverse` and `rbind(...)` in base R. To combine columns, we
could use `dplyr::bind_cols(...)` or `cbind(...)` respectively. However,
we must be absolutely sure that the rows align\!

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
    ##  1 control     replicate_1    0.016 treatment_A replicate_1    3.84 
    ##  2 control     replicate_2    0.049 treatment_B replicate_1   79.9  
    ##  3 treatment_A replicate_1    6.48  treatment_A replicate_2    3.73 
    ##  4 treatment_A replicate_2    5.76  treatment_D replicate_2   -0.481
    ##  5 treatment_B replicate_1  119.    treatment_C replicate_1   19.9  
    ##  6 treatment_B replicate_2  120.    control     replicate_2   -0.063
    ##  7 treatment_C replicate_1   30.3   treatment_C replicate_2   19.6  
    ##  8 treatment_C replicate_2   30.1   control     replicate_1    0.941
    ##  9 treatment_D replicate_1    0.203 treatment_B replicate_2   79.5  
    ## 10 treatment_D replicate_2    0.434 treatment_D replicate_1    0.444

To properly merge `half_1` and `half_2`, we need to use the colums named
`sample_id` and `replicate_id` as indices. This is what
`dplyr::inner_join(...)` does.

``` r
# this will be correct
inner_join(half_1, half_2, by = c("sample_id", "replicate_id"))
```

    ## # A tibble: 10 x 4
    ##    sample_id   replicate_id  conc_1 conc_2
    ##    <chr>       <chr>          <dbl>  <dbl>
    ##  1 control     replicate_1    0.016  0.941
    ##  2 control     replicate_2    0.049 -0.063
    ##  3 treatment_A replicate_1    6.48   3.84 
    ##  4 treatment_A replicate_2    5.76   3.73 
    ##  5 treatment_B replicate_1  119.    79.9  
    ##  6 treatment_B replicate_2  120.    79.5  
    ##  7 treatment_C replicate_1   30.3   19.9  
    ##  8 treatment_C replicate_2   30.1   19.6  
    ##  9 treatment_D replicate_1    0.203  0.444
    ## 10 treatment_D replicate_2    0.434 -0.481

If you want to keep rows that are in one table, but not in another,
there are also the functions `dpylr::left_join()` and
`dplyr::right_join()`. (Having both options is useful when you think of
piping.)

``` r
# remove objects no longer needed
rm(half_1, half_2)
```

### Importing Data from Excel

To read data from Excel, you can use `readxl::read_excel(...)` or
`XLConnect::readWorksheetFromFile(...)`. (Neither is part of the
`tidyverse` and probably not installed on your machine.)

However, I would disencourage to do so routinely, since `.xls` and
`.xlsx` are neither open (i.e. disclosed), nor standardized file
formats. This means they can be changed by Microsoft anytime and may
consequently not be (immediately) supported by these packages.

Another problem you might be facing is, that within Excel, it is
tempting to put multiple tables with different data side-by-side in a
single worksheet. To import such worksheets will require a lot of
polishing from your side after.

> If you plan to import from Excel, keep one set of data per sheet and
> export as `.csv`.

## Tidying Data

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
you tidy messy data.

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

For now, let’s next widen/spread/cast the long/gathered/molten
representation of `plate_1`\! Maybe, we would like to see the measured
intensities for `control` and `treatment_A` side-by-side given each
concentration and the replicate.

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

The format of `plate_1` from above can be useful if we wanted to
subtract the intensities measured in the control from the ones in the
treated samples. Here, `dplyr::mutate(...)` creates a new column or
modifies an exisiting column using the specified operations.

``` r
plate_1 %>% 
  # make wide table long
  pivot_longer(conc_1:conc_0, names_to = "concentration", values_to = "intensity") %>% 
  # make long table wide
  pivot_wider(names_from = "sample_id", values_from = "intensity") -> plate_1.wide

plate_1.wide %>% 
  # make new column with corrected treatment_A values
  mutate(tratment_A.corrected = treatment_A - control) %>% head(3)
```

    ## # A tibble: 3 x 5
    ##   replicate_id concentration control treatment_A tratment_A.corrected
    ##   <chr>        <chr>           <dbl>       <dbl>                <dbl>
    ## 1 replicate_1  conc_1          0.016        6.48                 6.46
    ## 2 replicate_1  conc_2          0.941        3.84                 2.90
    ## 3 replicate_1  conc_3         -0.307        3.06                 3.37

``` r
plate_1.wide %>% 
  # operate in place; not always advisable
  mutate(treatment_A = treatment_A - control) %>% head(3)
```

    ## # A tibble: 3 x 4
    ##   replicate_id concentration control treatment_A
    ##   <chr>        <chr>           <dbl>       <dbl>
    ## 1 replicate_1  conc_1          0.016        6.46
    ## 2 replicate_1  conc_2          0.941        2.90
    ## 3 replicate_1  conc_3         -0.307        3.37

You can check out more examples in the [`tidyr` vignette on
pivoting](https://tidyr.tidyverse.org/dev/articles/pivot.html).

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

    ## # A tibble: 3 x 6
    ##   experiment              conc_1 conc_2 conc_3 conc_4 conc_0
    ##   <chr>                    <dbl>  <dbl>  <dbl>  <dbl>  <dbl>
    ## 1 control/replicate_1      0.016  0.941 -0.307 -0.002  0.007
    ## 2 control/replicate_2      0.049 -0.063  0.301  0.15  -0.704
    ## 3 treatment_A/replicate_1  6.48   3.84   3.06   0.479  0.182

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

    ## # A tibble: 3 x 8
    ##   treatment specimen replicate_id conc_1 conc_2 conc_3 conc_4 conc_0
    ##   <chr>     <chr>    <chr>         <dbl>  <dbl>  <dbl>  <dbl>  <dbl>
    ## 1 control   <NA>     replicate_1   0.016  0.941 -0.307 -0.002  0.007
    ## 2 control   <NA>     replicate_2   0.049 -0.063  0.301  0.15  -0.704
    ## 3 treatment A        replicate_1   6.48   3.84   3.06   0.479  0.182

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
    ## 1 control replic… 1       <NA>    <NA>     0.016  0.941 -0.307 -0.002
    ## 2 control replic… 2       <NA>    <NA>     0.049 -0.063  0.301  0.15 
    ## 3 treatm… A       replic… 1       <NA>     6.48   3.84   3.06   0.479
    ## # … with 1 more variable: conc_0 <dbl>

## Hands-On Excercise

The data for another plate has been saved in a file called
‘plates.RData’. When you open this file, you should see an data set
called `plate_4` and a named vector called `dose` in the ‘Global
Environment’.

``` r
load(file = "./part_10-working_with_tables_files/plates.RData")
```

1.  Combine with `plate_data`, but be careful, the replicates (`.A` and
    `.B`) are actually `replicate_3` and `replicate_4`. You will
    probably want to change this. Hint: You can use `if_else()` to
    mutate values on a case-by-case basis.

2.  Tidy `plate_data` and replace concentration with actual values.
