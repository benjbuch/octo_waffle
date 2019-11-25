Selecting and Manipulating Data
================

  - [Manipulating Rows (“Cases”)](#manipulating-rows-cases)
      - [Arranging Rows](#arranging-rows)
      - [Extracting Rows](#extracting-rows)
  - [Manipulating Columns
    (“Variables”)](#manipulating-columns-variables)
      - [Selecting Columns](#selecting-columns)
      - [Extracting and Arranging
        Columns](#extracting-and-arranging-columns)
      - [Making New Variables](#making-new-variables)
      - [New Variables from Non-Vectorized
        Functions](#new-variables-from-non-vectorized-functions)
  - [Hands-On Exercise](#hands-on-exercise)

-----

In this section, you will learn how to select specific entries (rows)
based on their content from a table. Also, you will learn how to
retrieve specific variables (columns) from the table and/or add new
variables based on exisiting values to the data.

If you have not done so yet, please attach the `tidyverse` and make sure
you have the `plate_data` object from the [last
section](part_11-tidying_tables.md)–exercises included. If not, here you
go.

``` r
library(tidyverse)
readRDS("part_11-plate_data.rds") -> plate_data
```

Having combined and tidied the plate data, you will have an object
`plate_data` containing

| sample\_id | replicate\_id | concentration | intensity |
| :--------: | :-----------: | :-----------: | :-------: |
|  control   |     rep.A     |       0       |  \-0.121  |
|  control   |     rep.A     |       1       |    NA     |
|  control   |     rep.A     |      10       |    NA     |
|  control   |     rep.A     |      100      |  \-0.340  |
|  control   |     rep.A     |     1000      |    NA     |
|  control   |     rep.B     |       0       |   0.232   |
|  control   |     rep.B     |       1       |    NA     |

*(table abridged)*

## Manipulating Rows (“Cases”)

We have already seen how to add cases using `add_row(...)`. Let’s see
how to arrange and extract rows based on their content.

### Arranging Rows

Tables can be sorted by specific columns. This is rather a convenience
for the user than a necessity to perform further manipulations.

``` r
plate_data %>% arrange(concentration, intensity)
plate_data %>% arrange(desc(concentration), intensity)
```

### Extracting Rows

These functions return a subset of rows as a new table.

``` r
# extract rows that meet criteria
plate_data %>% filter(sample_id == "treatment_B")
plate_data %>% filter(replicate_id == "rep.B" & concentration > 50)
plate_data %>% filter(intensity * concentration > 1e4)

plate_data %>% filter(intensity >= 1 & intensity <= 3)
plate_data %>% filter(between(intensity, 1, 3)) # same, shorter

plate_data %>% filter(near(intensity, 10, 0.5)) # safe for floating point numbers

# remove duplicate rows
plate_data %>% distinct(sample_id, concentration)
```

One can also select (randomly) a number of cases.

``` r
# randomly extract N rows without replacement
plate_data %>% sample_n(5, replace = FALSE)
# randomly extract % rows with replacement
plate_data %>% sample_frac(.1, replace = TRUE)

# select rows by position
plate_data %>% slice(4:6)

# order and select top N entries
plate_data %>% top_n(5, intensity)
```

## Manipulating Columns (“Variables”)

It might not be obvious at this point, but most power of the `tidyverse`
is taken from its ability to operate on existing columns and create new
columns.

### Selecting Columns

Many functions will require a proper selection of columns, e.g. during
data tidying. This is why we focus here on how to specify a column
selection before we move on to more illustrative examples.

> The `tidyverse` makes extensive use of ‘quasiquotation’. This means
> that you can refer e.g. to column names (which are usually of type
> `character`) without using quotation marks (`""` or `''`). But beware,
> some arguments must be `character` vectors. These must be quoted.

  - To select a column, you can always **type its name between quotation
    marks**. If you want to specify multiple columns, use a `character`
    vector with the column names.

  - Most functions will accept **unquoted column names**. You don’t need
    a vector or anything like that then.
    
    Typically, this is indicated with `...` in the ‘Usage’ section of
    the Documentation.

  - In addition, there are **helper functions** to select multiple
    columns based on their position or based on their name.
    
    They are documented in `?select_helpers`.
    
    | operator/function | selection                                        |
    | ----------------- | ------------------------------------------------ |
    | `everything`      | all columns                                      |
    | `last_col`        | last column                                      |
    | `one_of`          | these column names as character vector           |
    | `starts_with`     | column names starting with this prefix literally |
    | `ends_with`       | column names ending with this suffix literally   |
    | `contains`        | column names containing this string literally    |
    | `matches`         | column names matching the regular expression     |
    | `:`               | all columns between                              |
    | `-`               | all columns except                               |
    

    For ‘scoped’ summarizing (`summarize_at`) and mutating verbs
    (`mutate_at`), you will need to wrap the helper statement with
    `vars(...)`.
    
    For ‘scoped’ filtering verbs (`filter_all` or `filter_if`), there
    are additional variants called `all_vars(...)` and `any_vars(...)`.

### Extracting and Arranging Columns

Most frequently, you will need

  - `select` to (optionally rename and) extract columns as table, or
  - `pull` to extract column values of *one* column as a vector; which
    defaults to the last column is none is specified.

<!-- end list -->

``` r
# return (last) column as vector
plate_data %>% pull
# return selected columns as table
plate_data %>% select(intensity)   # with quasiquotation
plate_data %>% select("intensity") # with explicit names
plate_data %>% select(starts_with("int"))
plate_data %>% select(ends_with("id"))
plate_data %>% select(c("sample_id", "replicate_id"))
```

There are the ‘scoped’ variants,

  - `select_all` to rename all columns (e.g. make uppercase),
  - `select_at` to (optionally rename and) extract columns select with
    `vars(...)`, and
  - `select_if` to (optionally rename and) extract columns that meet a
    certain property, (e.g. contain numeric values).

<!-- end list -->

``` r
# select based e.g. on data type
plate_data %>% select_if(is.numeric)
# select columns and rename column names
plate_data %>% select_all(toupper)
plate_data %>% select_at(vars(ends_with("id")), ~ str_c(., "iot"))
```

To rename columns only, there are `rename` variants of the above.

``` r
plate_data %>% rename("conc" = "concentration") # with explicit names
plate_data %>% rename(conc = concentration)     # with quasiquotation
plate_data %>% rename_at(vars(ends_with("id")), ~ str_c(., c("ea", "iot")))
```

Less importantly, `select` can be used for re-arranging columns.

``` r
plate_data %>% select(concentration, ends_with("id"), intensity)
```

### Making New Variables

`mutate` and its ‘scoped’ variants will apply *vectorized* functions on
the selected columns and append a new column with the result of the
transformation.

`transmute` is a short-hand for `mutate` followed by `select` on the
result.

``` r
# create a new column ...
plate_data %>% mutate(log10(concentration))
# ... with another name ...
plate_data %>% mutate(log_conc = log10(concentration))
# ... or in place
plate_data %>% mutate(concentration = log10(concentration))
# with subsequent selection
plate_data %>% transmute(log_conc = log10(concentration))
# multiple columns can be specified at the same time
plate_data %>% transmute(log_conc = log10(concentration), 
                         sqr_conc = concentration ** 2)
```

There are many useful functions in `dplyr` to add cumulative aggregates
and ranks to the data. Have a look on this [cheat
sheet](https://github.com/rstudio/cheatsheets/raw/master/data-transformation.pdf).

``` r
# cumulative sum 
plate_data %>% filter(!is.na(intensity)) %>% mutate(cumsum(concentration))
# break the input vector into N buckets based on rank
plate_data %>% filter(!is.na(intensity)) %>% mutate(bucket_no = ntile(intensity, 10))
# break the input vector continuously on rank
plate_data %>% filter(!is.na(intensity)) %>% mutate(percent_rank(intensity))
```

An interesting application is to replace values or create variables
based on a combination of logical evaluations.

``` r
# replace an annoying value with NA
plate_data %>% mutate(concentration = na_if(concentration, 0))
# replace all missing values with a value
plate_data %>% mutate(replace_na(intensity, 0))

# general replacements
plate_data %>% 
  mutate(sample_id = recode(sample_id, control = "lemon", 
                            replicate_A = "banana",
                            replicate_B = "apple"))

# replace based on logical conditions
plate_data %>% filter(!is.na(intensity)) %>% 
  mutate(intensity = if_else(concentration == 100, intensity * 1e4, intensity))

# replace based on multiple conditions ...
plate_data %>% filter(!is.na(intensity)) %>% 
  mutate(intensity = case_when(
    concentration == 0   ~ intensity + 2e2,
    concentration == 1e2 ~ intensity * 1e4,
    # all other cases
    TRUE ~ intensity
  ))
```

``` r
# ... especially useful when you want to create a new variable based on a
# complex combination of existing variables
dplyr::starwars %>%
  select(name:mass, gender, species) %>%
  mutate(type = case_when(
    height > 200 | mass > 200 ~ "large",
    species == "Droid"        ~ "robot",
    TRUE                      ~ "other"
  )) -> my_starwars_data; my_starwars_data
```

    ## # A tibble: 87 x 6
    ##    name               height  mass gender species type 
    ##    <chr>               <int> <dbl> <chr>  <chr>   <chr>
    ##  1 Luke Skywalker        172    77 male   Human   other
    ##  2 C-3PO                 167    75 <NA>   Droid   robot
    ##  3 R2-D2                  96    32 <NA>   Droid   robot
    ##  4 Darth Vader           202   136 male   Human   large
    ##  5 Leia Organa           150    49 female Human   other
    ##  6 Owen Lars             178   120 male   Human   other
    ##  7 Beru Whitesun lars    165    75 female Human   other
    ##  8 R5-D4                  97    32 <NA>   Droid   robot
    ##  9 Biggs Darklighter     183    84 male   Human   other
    ## 10 Obi-Wan Kenobi        182    77 male   Human   other
    ## # … with 77 more rows

Let’s have a look at a different scenario: Creating a “message” from the
column contents. For example, you may wish to print a statment such as
“Luke Skywalker is 1.7m tall and of type ‘other’.”

``` r
# acceptable, but not great syntax ...
my_starwars_data %>% 
  transmute(message = str_c(name, " is ", round(height / 100, 1), "m tall and of type ‘", type, "’.")) %>% 
  head(4)
```

    ## # A tibble: 4 x 1
    ##   message                                         
    ##   <chr>                                           
    ## 1 Luke Skywalker is 1.7m tall and of type ‘other’.
    ## 2 C-3PO is 1.7m tall and of type ‘robot’.         
    ## 3 R2-D2 is 1m tall and of type ‘robot’.           
    ## 4 Darth Vader is 2m tall and of type ‘large’.

``` r
# ... more legible, but trailing zeros are dropped after rounding ...
my_starwars_data %>% 
  mutate(height_in_m = round(height / 100, 1)) %>% 
  transmute(message = str_glue("{name} is {height_in_m}m tall and of type ‘{type}’."))  %>% 
  head(4)
```

    ## # A tibble: 4 x 1
    ##   message                                         
    ##   <glue>                                          
    ## 1 Luke Skywalker is 1.7m tall and of type ‘other’.
    ## 2 C-3PO is 1.7m tall and of type ‘robot’.         
    ## 3 R2-D2 is 1m tall and of type ‘robot’.           
    ## 4 Darth Vader is 2m tall and of type ‘large’.

``` r
# ... better
my_starwars_data %>% 
  transmute(message = str_glue("{name} is {format(round(height / 100, 1), digits = 1)}m tall and of type ‘{type}’."))  %>% 
  head(4)
```

    ## # A tibble: 4 x 1
    ##   message                                         
    ##   <glue>                                          
    ## 1 Luke Skywalker is 1.7m tall and of type ‘other’.
    ## 2 C-3PO is 1.7m tall and of type ‘robot’.         
    ## 3 R2-D2 is 1.0m tall and of type ‘robot’.         
    ## 4 Darth Vader is 2.0m tall and of type ‘large’.

### New Variables from Non-Vectorized Functions

*(Slightly advanced material. This is adapted from [Kenta Yoshida’s
blog](http://yoshidk6.hatenablog.com/entry/2018/09/05/222248) and [Jenny
Bryan’s webinar](https://github.com/jennybc/row-oriented-workflows).)*

In most cases, the function you want to apply in `dplyr::mutate` is
vectorized and there is no need to use the following ‘tricks’. This
works because the output from vectorized functions have the same length
as the data (number of rows). However, you can easily bang your head
against a wall if you don’t know how to deal with non-vectorized
functions in the `dplyr::mutate` workflow.

The following will not work because `seq` is not vectorized.

``` r
ex1 <- tibble(A = c(1, 2), B = c(3, 6), C = c(8, 10))
ex1 %>% mutate(S = seq(A, B))
```

Instead, we must use `purrr::map` (one list as input) or `purrr::map2`
(two lists as input), which will take lists of equal length (i.e. the
columns) as input arguments and then apply the function of interest
using each pair of elements from the lists.

> The map functions always return a `list` of lists.

``` r
ex1 %>% mutate(S = map2(A, B, seq)) -> rs1
rs1
```

    ## # A tibble: 2 x 4
    ##       A     B     C S        
    ##   <dbl> <dbl> <dbl> <list>   
    ## 1     1     3     8 <int [3]>
    ## 2     2     6    10 <int [5]>

``` r
# this will print out the content of the lists in column D 
as.data.frame(rs1)
```

    ##   A B  C             S
    ## 1 1 3  8       1, 2, 3
    ## 2 2 6 10 2, 3, 4, 5, 6

If we want to explicitly specify arguments by name, we have to wrap the
mapping call as lambda with `~` and can use `..1` (or `.x`) and `..2`
(or `.y`) to refer to the first and respectivley second input list.

``` r
ex1 %>% mutate(S = map2(A, B, ~seq(from = ..1, to = 10, by = ..2))) %>% 
  as.data.frame
```

    ##   A B  C           S
    ## 1 1 3  8 1, 4, 7, 10
    ## 2 2 6 10        2, 8

With more and more input arguments, this strategy becomes increasingly
confusing, and we need to beware of the positional order of the input
arguments. Here are some foolproof alternatives.

If the entire table has *the exact same number and the exact same column
names* as the input arguments of the function, this simple `purrr::pmap`
syntax works, irrespective of the column order.

``` r
ex1 %>% 
  # you may quote any of these or none ...
  rename(from = "A", "to" = C, "by" = "B") %>% 
  mutate(S = pmap(., seq)) %>% 
  as.data.frame
```

    ##   from by to       S
    ## 1    1  3  8 1, 4, 7
    ## 2    2  6 10    2, 8

In most cases, you will have more columns than input arguments. Thus, R
will complain with a warning about unused arguments (for each row\!).

``` r
ex2 <- ex1 %>% bind_cols(D = c("apple", "banana"))
ex2 %>% 
  rename(from = A, to = C, by = B) %>% 
  mutate(S = pmap(., seq)) %>% 
  as.data.frame
```

    ## Warning: In seq.default(from = .l[[1]][[i]], by = .l[[2]][[i]], to = .l[[3]][[i]], 
    ##     D = .l[[4]][[i]], ...) :
    ##  extra argument 'D' will be disregarded
    
    ## Warning: In seq.default(from = .l[[1]][[i]], by = .l[[2]][[i]], to = .l[[3]][[i]], 
    ##     D = .l[[4]][[i]], ...) :
    ##  extra argument 'D' will be disregarded

    ##   from by to      D       S
    ## 1    1  3  8  apple 1, 4, 7
    ## 2    2  6 10 banana    2, 8

The easiest solution is to provide a named list as input vector to
`purrr::pmap`.

``` r
ex2 %>% 
  mutate(S = pmap(list(from = A, to = C, by = B), seq)) %>% 
  as.data.frame
```

    ##   A B  C      D       S
    ## 1 1 3  8  apple 1, 4, 7
    ## 2 2 6 10 banana    2, 8

What to do with this fancy new column containing lists? We could want to
apply another function on them.

``` r
ex2 %>% 
  mutate(S = pmap(list(from = A, to = C, by = B), seq)) %>% 
  # does not work since ‘S’ contains elements of type list
  mutate(S_mean = mean(S))

ex2 %>% 
  mutate(S = pmap(list(from = A, to = C, by = B), seq)) %>% 
  # wrong as the entire column is unlisted
  mutate(S_mean = mean(unlist(S)))
```

Indeed, we need `purrr::map` again.

``` r
ex2 %>% 
  mutate(S = pmap(list(from = A, to = C, by = B), seq)) %>% 
  mutate(S_mean = map(S, mean))
```

    ## # A tibble: 2 x 6
    ##       A     B     C D      S         S_mean   
    ##   <dbl> <dbl> <dbl> <chr>  <list>    <list>   
    ## 1     1     3     8 apple  <dbl [3]> <dbl [1]>
    ## 2     2     6    10 banana <dbl [2]> <dbl [1]>

If we already know the data type of the return value of the mapped
function, we can use an appropriate `purrr::pmap_*` derivate.

``` r
ex2 %>% 
  mutate(S = pmap(list(from = A, to = C, by = B), seq)) %>% 
  # coerce result to ‘double’ instead of ‘list’
  mutate(S_mean = map_dbl(S, mean))
```

    ## # A tibble: 2 x 6
    ##       A     B     C D      S         S_mean
    ##   <dbl> <dbl> <dbl> <chr>  <list>     <dbl>
    ## 1     1     3     8 apple  <dbl [3]>      4
    ## 2     2     6    10 banana <dbl [2]>      5

Another useful application is to calculate the row-wise sum, mean,
median, standard deviation etc. even on a subselection of columns. Since
we want to pass multiple columns as arguments, we need `purrr::pmap`.

``` r
ex3 <- tribble(
  ~ name, ~ t1, ~t2, ~t3,
  "Abby",    1,   4,   6,
  "Bess",    7,   2,   5,
  "Carl",    9,   8,   3
)

ex3 %>% mutate(t_sum = pmap_dbl(list(t1, t2, t3), sum))
```

    ## # A tibble: 3 x 5
    ##   name     t1    t2    t3 t_sum
    ##   <chr> <dbl> <dbl> <dbl> <dbl>
    ## 1 Abby      1     4     6    11
    ## 2 Bess      7     2     5    14
    ## 3 Carl      9     8     3    20

``` r
ex3 %>% mutate(t_sum = pmap_dbl(select(., starts_with("t")), sum))
```

    ## # A tibble: 3 x 5
    ##   name     t1    t2    t3 t_sum
    ##   <chr> <dbl> <dbl> <dbl> <dbl>
    ## 1 Abby      1     4     6    11
    ## 2 Bess      7     2     5    14
    ## 3 Carl      9     8     3    20

This works especially smoothly with `sum`, since it takes `...` as
primary argument. This is different for many other functions, which
expect *vectors* as their first argument.

``` r
   sum(..., na.rm = FALSE)
  mean(x, trim = 0, na.rm = FALSE, ...)
median(x, na.rm = FALSE, ...)
   var(x, y = NULL, na.rm = FALSE, use)
```

To deal with these, `purrr` has a family of `lift_*` functions that
convert functions between these different forms. For example,
`purrr::lift_vd` converts a function that takes a ‘vector’ as input into
one that accepts ‘dots’.

``` r
ex3 %>% mutate(t_avg = pmap_dbl(list(t1, t2, t3), lift_vd(mean)))
```

    ## # A tibble: 3 x 5
    ##   name     t1    t2    t3 t_avg
    ##   <chr> <dbl> <dbl> <dbl> <dbl>
    ## 1 Abby      1     4     6  3.67
    ## 2 Bess      7     2     5  4.67
    ## 3 Carl      9     8     3  6.67

Note that if you have a frequent need to compute these summary
statistics row-wise, on a subset of columns, it is highly suggestive
that your data is in the wrong shape, i.e. it’s not tidy. In the next
section, we explore approaches that are more transparent than using a
`purrr::lift_*` with `purrr::pmap` inside `dplyr::mutate(...)` and,
consequently, more verbose.

## Hands-On Exercise

1.  Change `"rep.A"` and `"rep.B"` into `"replicate_3"` and
    `"replicate_4"` respectively, arrange by `sample_id` and
    `replicate_id` (descending), then store the result as `plate_data`.

2.  Subtract `control` from all treatments (without modifying the data
    set).
