Grouping and Summarizing Data
================

  - [Making Groups](#making-groups)
  - [Summarizing Groups](#summarizing-groups)
  - [Comparing Groups](#comparing-groups)
      - [Student’s *t* Test](#students-t-test)
      - [Student’s *t* Test with Multiple
        Comparisons](#students-t-test-with-multiple-comparisons)
      - [Linear Regression and Beyond](#linear-regression-and-beyond)
  - [Hands-On Exercise](#hands-on-exercise)

-----

You will learn in this section is how to summarize (large) data sets.
Typically, the data is first split into several groups defined by a
categorical variable (‘factor’) and then some sort of summary statistics
are applied to each group.

If you have not done so yet, please attach the `tidyverse` and make sure
you have the `plate_data` object from the [last
section](part_12-manipulating_tables.html)–exercises included. If not,
here you go.

``` r
library(tidyverse)
readRDS("part_12-plate_data.rds") -> plate_data
```

## Making Groups

`group_by(...)` specifies the columns containing the categorical
variables on which you want to split your data set. It is *not*
necessary (although sometimes useful) to convert the column into a
`factor`.

Grouping does not change how the data looks like, apart from listing how
it’s grouped.

``` r
plate_data %>% group_by(sample_id)
```

    ## # A tibble: 100 x 4
    ## # Groups:   sample_id [5]
    ##    sample_id replicate_id concentration intensity
    ##    <chr>     <chr>                <dbl>     <dbl>
    ##  1 control   replicate_4              0     0.290
    ##  2 control   replicate_4              1    NA    
    ##  3 control   replicate_4             10    NA    
    ##  4 control   replicate_4            100    -0.456
    ##  5 control   replicate_4           1000    NA    
    ##  6 control   replicate_3              0    -0.152
    ##  7 control   replicate_3              1    NA    
    ##  8 control   replicate_3             10    NA    
    ##  9 control   replicate_3            100    -0.426
    ## 10 control   replicate_3           1000    NA    
    ## # … with 90 more rows

To check how many entries there are per group, there is `group_data()`.

``` r
plate_data %>% group_by(sample_id, replicate_id) %>% group_data()
```

    ## # A tibble: 20 x 3
    ##    sample_id   replicate_id .rows    
    ##    <chr>       <chr>        <list>   
    ##  1 control     replicate_1  <int [5]>
    ##  2 control     replicate_2  <int [5]>
    ##  3 control     replicate_3  <int [5]>
    ##  4 control     replicate_4  <int [5]>
    ##  5 treatment_A replicate_1  <int [5]>
    ##  6 treatment_A replicate_2  <int [5]>
    ##  7 treatment_A replicate_3  <int [5]>
    ##  8 treatment_A replicate_4  <int [5]>
    ##  9 treatment_B replicate_1  <int [5]>
    ## 10 treatment_B replicate_2  <int [5]>
    ## 11 treatment_B replicate_3  <int [5]>
    ## 12 treatment_B replicate_4  <int [5]>
    ## 13 treatment_C replicate_1  <int [5]>
    ## 14 treatment_C replicate_2  <int [5]>
    ## 15 treatment_C replicate_3  <int [5]>
    ## 16 treatment_C replicate_4  <int [5]>
    ## 17 treatment_D replicate_1  <int [5]>
    ## 18 treatment_D replicate_2  <int [5]>
    ## 19 treatment_D replicate_3  <int [5]>
    ## 20 treatment_D replicate_4  <int [5]>

We could also use ‘scoped’ variants such as `group_by_at(...)`,
`group_by_if(...)` or `group_by_all(...)` to select the columns on which
to group.

``` r
plate_data %>% group_by_at(vars(ends_with("id"))) # group by "sample_id" and "replicate_id"
```

By default, `group_by(...)` overrides existing grouping of the data set.
Use `group_by(..., add = TRUE)` to append.

To remove the grouping, use `ungroup()`.

## Summarizing Groups

A common summary of continuous data is to give the average and a measure
of the spread e.g. using the standard deviation `sd(x)`, the median
absolute deviation `mad(x)`, or the interquartile range `IQR(x)`.

Let’s have a look at a simple case first to identify the differences.

``` r
df <- data.frame(g = rep(c("a", "b"), each = 2), x = seq(1, 4)) 

df %>% mutate(y = mean(x))
```

    ##   g x   y
    ## 1 a 1 2.5
    ## 2 a 2 2.5
    ## 3 b 3 2.5
    ## 4 b 4 2.5

``` r
df %>% group_by(g) %>% mutate(y = mean(x))
```

    ## # A tibble: 4 x 3
    ## # Groups:   g [2]
    ##   g         x     y
    ##   <fct> <int> <dbl>
    ## 1 a         1   1.5
    ## 2 a         2   1.5
    ## 3 b         3   3.5
    ## 4 b         4   3.5

``` r
df %>% group_by(g) %>% summarize(y = mean(x))
```

    ## # A tibble: 2 x 2
    ##   g         y
    ##   <fct> <dbl>
    ## 1 a       1.5
    ## 2 b       3.5

Given `plate_data`, let’s calculate the average intensity for each
`sample_id` and `concentration`.

``` r
plate_data %>% 
  group_by(sample_id, concentration) %>% 
  # calculate mean and sd
  summarize(mean = mean(intensity, na.rm = TRUE), 
            sd = sd(intensity, na.rm = TRUE), 
            # count the rows that are not NA
            N  = sum(!is.na(intensity)))
```

    ## # A tibble: 25 x 5
    ## # Groups:   sample_id [5]
    ##    sample_id   concentration    mean     sd     N
    ##    <chr>               <dbl>   <dbl>  <dbl> <int>
    ##  1 control                 0 -0.0568 0.293      4
    ##  2 control                 1  0.120  0.402      2
    ##  3 control                10 -0.328  0.0686     2
    ##  4 control               100 -0.0758 0.425      4
    ##  5 control              1000 -0.253  1.04       2
    ##  6 treatment_A             0  0.152  0.700      4
    ##  7 treatment_A             1  0.761  1.02       2
    ##  8 treatment_A            10  2.13   0.528      2
    ##  9 treatment_A           100  3.26   1.13       4
    ## 10 treatment_A          1000  6.26   0.374      2
    ## # … with 15 more rows

Suspiciously, the standard deviation of some samples with four
replicates are higher than the samples with just two replicates. Might
there be something ‘wrong’ with one of `replicate_3` or `replicate_4`?

For categorical data, we are more likely to be interested in the number
of observations, `n()`, or the number of unique values a variable takes
within a group, `n_distinct(...)`.

A routine task is for example to calculate the proportional composition.

``` r
mtcars %>%
    group_by(am, gear) %>%
    summarise(n = n()) %>%
    mutate(freq = n / sum(n))
```

    ## # A tibble: 4 x 4
    ## # Groups:   am [2]
    ##      am  gear     n  freq
    ##   <dbl> <dbl> <int> <dbl>
    ## 1     0     3    15 0.789
    ## 2     0     4     4 0.211
    ## 3     1     4     8 0.615
    ## 4     1     5     5 0.385

``` r
mtcars %>%
    group_by(am, gear) %>%
    summarise(n = n()) %>%
    mutate(freq = prop.table(n))
```

    ## # A tibble: 4 x 4
    ## # Groups:   am [2]
    ##      am  gear     n  freq
    ##   <dbl> <dbl> <int> <dbl>
    ## 1     0     3    15 0.789
    ## 2     0     4     4 0.211
    ## 3     1     4     8 0.615
    ## 4     1     5     5 0.385

There are many more summary functions. Have a look on this [cheat
sheet](https://github.com/rstudio/cheatsheets/raw/master/data-transformation.pdf).

## Comparing Groups

Statistics is a powerful way to quantify how different supposedly
different groups are from each other. Let’s see how to make uni- and
multivariate comparisons or to fit models through data based on grouped
`data.frame` objects in the `tidyverse`.

### Student’s *t* Test

In this section, we will use a built-in data set in R, `reshape2::tips`,
which contains a record on the tips a waiter received over several
months in a restaurant.

``` r
reshape2::tips %>% as_tibble()
```

    ## # A tibble: 244 x 7
    ##    total_bill   tip sex    smoker day   time    size
    ##         <dbl> <dbl> <fct>  <fct>  <fct> <fct>  <int>
    ##  1      17.0   1.01 Female No     Sun   Dinner     2
    ##  2      10.3   1.66 Male   No     Sun   Dinner     3
    ##  3      21.0   3.5  Male   No     Sun   Dinner     3
    ##  4      23.7   3.31 Male   No     Sun   Dinner     2
    ##  5      24.6   3.61 Female No     Sun   Dinner     4
    ##  6      25.3   4.71 Male   No     Sun   Dinner     4
    ##  7       8.77  2    Male   No     Sun   Dinner     2
    ##  8      26.9   3.12 Male   No     Sun   Dinner     4
    ##  9      15.0   1.96 Male   No     Sun   Dinner     2
    ## 10      14.8   3.23 Male   No     Sun   Dinner     2
    ## # … with 234 more rows

For example, we could be curious whether male or female guests give
higher tips. In base R, we would go with this one-liner.

``` r
t.test(tip ~ sex, data = reshape2::tips)
```

    ## 
    ##  Welch Two Sample t-test
    ## 
    ## data:  tip by sex
    ## t = -1.4895, df = 215.71, p-value = 0.1378
    ## alternative hypothesis: true difference in means is not equal to 0
    ## 95 percent confidence interval:
    ##  -0.5951448  0.0828057
    ## sample estimates:
    ## mean in group Female   mean in group Male 
    ##             2.833448             3.089618

However, if we wanted to perform this test by weekday, we were left with
subsetting the data by day and analyze each subset.

``` r
reshape2::tips[which(reshape2::tips$day == "Fri"), ] -> tips.Fri
t.test(tip ~ sex, data = tips.Fri)
```

    ## 
    ##  Welch Two Sample t-test
    ## 
    ## data:  tip by sex
    ## t = 0.1849, df = 16.895, p-value = 0.8555
    ## alternative hypothesis: true difference in means is not equal to 0
    ## 95 percent confidence interval:
    ##  -0.9177677  1.0939899
    ## sample estimates:
    ## mean in group Female   mean in group Male 
    ##             2.781111             2.693000

We would certainly prefer a neat table with all the results organized by
the grouping variable. In a sense, a vectorized approach.

Let’s look how to perform a Student’s *t* test within the `tidyverse`.

We will use `group_modify(...)` to create a summary for cases when
`summarize(...)` is too limited. `group_modify(...)` takes a function as
argument, which operates on and returns back a `data.frame`. In this
case, the function will be given the different `data.frame` objects, one
for each group (here: Thu, Fri, Sat, Sun).

> Since R’s `t.test` does not return a `data.frame` by default, we wrap
> it with `~ broom::tidy(...)`. This function makes a `data.frame`
> containing the parameters (e.g. coefficients and slopes) of the model.

> The `~` in front of `broom::tidy(...)` is mandatory for reasons beyond
> the scope of this tutorial at this point.

Within `group_modify`, we can refer to the current subset with `.` (or
`.x`) and the value of the grouping variable with `.y`.

``` r
reshape2::tips %>% 
  group_by(day) %>%
  group_modify(~ broom::tidy(t.test(tip ~ sex, data = .)))
```

    ## # A tibble: 4 x 11
    ## # Groups:   day [4]
    ##   day   estimate estimate1 estimate2 statistic p.value parameter conf.low
    ##   <fct>    <dbl>     <dbl>     <dbl>     <dbl>   <dbl>     <dbl>    <dbl>
    ## 1 Fri     0.0881      2.78      2.69     0.185   0.856      16.9   -0.918
    ## 2 Sat    -0.282       2.80      3.08    -0.855   0.395      73.7   -0.939
    ## 3 Sun     0.147       3.37      3.22     0.465   0.645      31.3   -0.497
    ## 4 Thur   -0.405       2.58      2.98    -1.28    0.205      56.2   -1.04 
    ## # … with 3 more variables: conf.high <dbl>, method <chr>,
    ## #   alternative <chr>

A little down-side of this nicely general approach is that column names
may become rather enigmatic. This is because they need to apply to very
different cases in which `broom::tidy(...)` is used.

Here, the columns named `estimate`, `estimate1` and `estimate2` refer to
the mean difference between both groups, the mean of the first group
(here: ‘Female’) and the mean of the second group (here: ‘Male’)
respectively.

### Student’s *t* Test with Multiple Comparisons

If more than two groups are compared, we should adjust the p-values for
the additional comparisons. In R, the function is a little bit
misleadingly named `pairwise.t.test(x, g)`, but can be used for paired
and unpaired comparisons between multiple groups.

First, the same as above.

``` r
reshape2::tips %>% 
  group_by(day) %>% 
  group_modify(~ broom::tidy(pairwise.t.test(
    x = .$tip, # the column with the values to compare
    g = .$sex, # the column from which the different groups are taken
    p.adjust.method = "none", pool.sd = FALSE)))
```

    ## # A tibble: 4 x 4
    ## # Groups:   day [4]
    ##   day   group1 group2 p.value
    ##   <fct> <chr>  <chr>    <dbl>
    ## 1 Fri   Male   Female   0.856
    ## 2 Sat   Male   Female   0.395
    ## 3 Sun   Male   Female   0.645
    ## 4 Thur  Male   Female   0.205

Let’s group by sex and compare the tips for each weekday without
applying a correction.

``` r
reshape2::tips %>% 
  group_by(sex) %>% 
  group_modify(~ broom::tidy(pairwise.t.test(
    x = .$tip, g = .$day, p.adjust.method = "none", pool.sd = FALSE)))
```

    ## # A tibble: 12 x 4
    ## # Groups:   sex [2]
    ##    sex    group1 group2 p.value
    ##    <fct>  <chr>  <chr>    <dbl>
    ##  1 Female Sat    Fri     0.958 
    ##  2 Female Sun    Fri     0.171 
    ##  3 Female Thur   Fri     0.586 
    ##  4 Female Sun    Sat     0.120 
    ##  5 Female Thur   Sat     0.461 
    ##  6 Female Thur   Sun     0.0227
    ##  7 Male   Sat    Fri     0.374 
    ##  8 Male   Sun    Fri     0.206 
    ##  9 Male   Thur   Fri     0.518 
    ## 10 Male   Sun    Sat     0.635 
    ## 11 Male   Thur   Sat     0.761 
    ## 12 Male   Thur   Sun     0.424

However, when properly done, the ‘significant’ difference for female
payers between Thursdays and Sundays vanishes.

``` r
reshape2::tips %>% 
  group_by(sex) %>% 
  group_modify(~ broom::tidy(pairwise.t.test(
    x = .$tip, g = .$day, p.adjust.method = "hochberg", pool.sd = FALSE)))
```

This approach can also be applied to other hypothesis testing functions
such as `cor.test`, `wilcox.test`, `chisq.test` etc.

### Linear Regression and Beyond

Another example on the same lines is a linear regression by group. Let’s
try to describe the `intensity` observed in `plate_data` as a function
of the `concentration`. Such a model is fit in R with the linear model
`lm` call.

``` r
plate_data %>% 
  group_by(sample_id) %>% 
  group_modify(~ broom::tidy(lm(intensity ~ log10(concentration + 1), data = .)))
```

    ## # A tibble: 10 x 6
    ## # Groups:   sample_id [5]
    ##    sample_id   term                  estimate std.error statistic   p.value
    ##    <chr>       <chr>                    <dbl>     <dbl>     <dbl>     <dbl>
    ##  1 control     (Intercept)            -0.0313     0.171   -0.183    8.58e-1
    ##  2 control     log10(concentration …  -0.0608     0.106   -0.572    5.78e-1
    ##  3 treatment_A (Intercept)             0.0784     0.339    0.231    8.21e-1
    ##  4 treatment_A log10(concentration …   1.85       0.210    8.79     1.41e-6
    ##  5 treatment_B (Intercept)             1.21      17.3      0.0699   9.45e-1
    ##  6 treatment_B log10(concentration …  49.3       10.7      4.59     6.22e-4
    ##  7 treatment_C (Intercept)             0.100      2.07     0.0484   9.62e-1
    ##  8 treatment_C log10(concentration …  11.1        1.28     8.64     1.70e-6
    ##  9 treatment_D (Intercept)             0.407      0.304    1.34     2.10e-1
    ## 10 treatment_D log10(concentration …  -0.279      0.185   -1.51     1.63e-1

So far, we have used `broom::tidy` to get the parameters of the model.
If we are interested in the statistical summary of the overall model, we
use `broom::glance`.

``` r
plate_data %>% 
  group_by(sample_id) %>% 
  group_modify(~ broom::glance(lm(intensity ~ log10(concentration + 1), data = .)))
```

    ## # A tibble: 5 x 12
    ## # Groups:   sample_id [5]
    ##   sample_id r.squared adj.r.squared  sigma statistic p.value    df logLik
    ##   <chr>         <dbl>         <dbl>  <dbl>     <dbl>   <dbl> <int>  <dbl>
    ## 1 control      0.0266       -0.0546  0.432     0.327 5.78e-1     2  -7.02
    ## 2 treatmen…    0.866         0.854   0.853    77.3   1.41e-6     2 -16.6 
    ## 3 treatmen…    0.637         0.607  43.6      21.1   6.22e-4     2 -71.6 
    ## 4 treatmen…    0.861         0.850   5.21     74.6   1.70e-6     2 -41.9 
    ## 5 treatmen…    0.185         0.104   0.702     2.27  1.63e-1     2 -11.7 
    ## # … with 4 more variables: AIC <dbl>, BIC <dbl>, deviance <dbl>,
    ## #   df.residual <int>

All `broom` functions apply equally well to the output of generalized
linear and non-linear models.

## Hands-On Exercise

Unfortunately, some of the fits on `plate_data` are not of satisfactory
quality. Maybe we can figure out if this is because of a bad replicate
in our data.

1.  We will group `plate_data` by `sample_id` and `replicate_id`. One of
    the groups has no values recorded at all, which will cause troubles
    in later steps. Figure out which one it is and exlude it by
    filtering the rows accordingly.

2.  Now, perform the fit with `lm` and have a look only at the slopes.
    How much do they differ between replicates?
