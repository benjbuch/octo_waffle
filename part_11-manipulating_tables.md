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

> In the file path, `.` refers to the current working directory
> (`getwd()`), so that we can use relative paths.

The column names have been taken from the first line of the file (since
they were text). However, as we know the actual names, we should use
them. They can be set either using `colnames(plate_1)<-` or during
import. In the latter case, we can (and should) skip the first line of
the file.

Generally, it is disencouraged to rely on row names to work with complex
data sets since only one variable or parameter can be referenced. To be
useful, each parameter or variable should be stored in a separate
column. So, let’s add one column specifying the sample type and one
column specifying the replicate number.

If you had (for some reason) rownames in your table that you want to
keep, use `dplyr::rownames_to_column()`.

### Combining Data from Multiple Files with `row_bind`

A nice feature of using a programmatic approach for data analysis is
that it allows retrieving (systematically) many files from multiple
locations on your harddrive (e.g. reflecting multiple experiments) and
to either analyze them in identical ways one by one, or to combine them
for analysis.

Certainly, we could import each file individually.

However, if we have many files to analyse, this approach can become
tedious. So, we use `list.files(...)` to create a list of all files that
we would like to include. These can even use regular expressions for
pattern matching.

The `plate_files` object is a `list` with two `tibble` (`dplyr`’s way of
`data.frame`) objects. Apparently, two columns have been swapped in
`plate_3`, which could cause problems if we just pasted them. Luckily, R
pays attention for us.

To be mentioned here, we also (could) keep track of the file names (or
paths) by specifying an `.id` column called `file`, which will take the
names of the `list`. If need be, we can then make use of metadata stored
in the file name or file path.

For the moment, let’s combine the data with `plate_1` and call the
object `plate_data`.

### Combining Data with `join`

We have seen that the command to combine rows is `dpylr::bind_rows(...)`
in the `tidyverse` and `rbind(...)` in base R. To combine columns, we
could use `dplyr::bind_cols(...)` or `cbind(...)` respectively. However,
we must be absolutely sure that the rows align\!

In the following case, this fails. (Don’t care too much about how to
create the randomized halfs.)

To properly merge `half_1` and `half_2`, we need to use the colums named
`sample_id` and `replicate_id` as indices. This is what
`dplyr::inner_join(...)` does.

If you want to keep rows that are in one table, but not in another,
there are also the functions `dpylr::left_join()` and
`dplyr::right_join()`. (Having both options is useful when you think of
piping.)

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

*(table abridged)*

The longer the table, the tidyer the data. Here is the ‘long format’ of
`plate_1`:

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

Instead of selecting the columns `conc_1:conc_0`, we could have
specified the columns which *not* to gather; these are somtimes called
the ‘identifying (id) columns’. The following commands evaluate to the
same result.

We discuss selecting columns by name later.

For now, let’s next widen/spread/cast the long/gathered/molten
representation of `plate_1`\! Maybe, we would like to see the measured
intensities for `control` and `treatment_A` side-by-side given each
concentration and the replicate.

The format of `plate_1` from above can be useful if we wanted to
subtract the intensities measured in the control from the ones in the
treated samples. Here, `dplyr::mutate(...)` creates a new column or
modifies an exisiting column using the specified operations.

You can check out more examples in the [`tidyr` vignette on
pivoting](https://tidyr.tidyverse.org/dev/articles/pivot.html).

### Uniting and Splitting Columns

The `tidyr::unite(...)` function takes multiple column names and pastes
the column contents together.

The `tidyr::separate(...)` function does the reverse. By default, any
non-alphanumeric character will be used to split the column. Note that
missing pieces will be replaced with `NA`.

## Hands-On Excercise

The data for another plate has been saved in a file called
‘plates.RData’. When you open this file, there should be one
additional object called `plate_4` and a named vector called `dose` in
the ‘Global Environment’.

1.  For illustrative purposes merge `plate_4` with `plate_1` by
    `sample_id` only. Use `inner_join`, `left_join`, and `right_join`.
    Don’t save the results.

2.  Combine `plate_4` into `plate_data`, but be careful, the replicates
    (`.1` and `.2`) are actually `replicate_3` and `replicate_4`. You
    will probably want to change this.
