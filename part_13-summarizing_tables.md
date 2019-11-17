Grouping and Summarizing Data
================

  - [Making Groups](#making-groups)
  - [Summarizing Groups](#summarizing-groups)
  - [Comparing Groups](#comparing-groups)
      - [Student’s *t* Test](#students-t-test)
      - [Student’s *t* Test with Multiple
        Comparisons](#students-t-test-with-multiple-comparisons)
      - [Linear Regression and Beyond](#linear-regression-and-beyond)

The important object you will learn in this section is how to summarize
(large) data sets. Typically, the data is first split into several
groups defined by a categorical variable (‘factor’) and then some sort
of summary statistics are applied to each group.

## Making Groups

`group_by(...)` specifies the columns containing the categorical
variables on which you want to split your data set. Note that it is
*not* necessary (but sometimes useful) to convert the column into a
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
    ##  1 control   replicate_1              0     0.007
    ##  2 control   replicate_1              1    -0.002
    ##  3 control   replicate_1             10    -0.307
    ##  4 control   replicate_1            100     0.941
    ##  5 control   replicate_1           1000     0.016
    ##  6 control   replicate_2              0    -0.704
    ##  7 control   replicate_2              1     0.15 
    ##  8 control   replicate_2             10     0.301
    ##  9 control   replicate_2            100    -0.063
    ## 10 control   replicate_2           1000     0.049
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

We could also use some ‘scoped’ variants such as `group_by_at(...)`,
`group_by_if(...)` or `group_by_all(...)` to select the columns on which
to group.

``` r
plate_data %>% group_by_at(vars(ends_with("id"))) # group by "sample_id" and "replicate_id"
```

By default, `group_by(...)` overrides existing grouping of the data set.
Use `group_by(..., add = TRUE)` to append.

To remove the grouping, use `ungroup()`.

## Summarizing Groups

A common summary of continuous data is to give their average value and
the spread. Given `plate_data`, let’s calculate the average intensity
for each `sample_id` and `concentration`.

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
    ##  1 control                 0 -0.146  0.399      4
    ##  2 control                 1  0.074  0.107      2
    ##  3 control                10 -0.003  0.430      2
    ##  4 control               100  0.0432 0.614      4
    ##  5 control              1000  0.0325 0.0233     2
    ##  6 treatment_A             0  0.0945 0.293      4
    ##  7 treatment_A             1  0.199  0.395      2
    ##  8 treatment_A            10  2.61   0.644      2
    ##  9 treatment_A           100  6.26   5.57       4
    ## 10 treatment_A          1000  6.12   0.511      2
    ## # … with 15 more rows

Suspiciously, the standard deviation of the samples with four replicates
(`replicate_1`, `replicate_2`, `replicate_3` and `replicate_4`) are
higher than the others. We shall keep this in mind as there might be
something ‘wrong’ with one of the replicates.

For categorical data, we are more likely to be interested in the number
of observations, `n()`, or the number of unique values a variable takes
within a group, `n_distinct()`.

There are many more summary functions. Have a look on this [cheat
sheet](https://github.com/rstudio/cheatsheets/raw/master/data-transformation.pdf).

## Comparing Groups

Statistics is a powerful way to quantify how different supposedly
different groups are from each other. To make uni- and multivariate
comparisons or to fit models through data based on groups needs just
some more tweaks in the `tidyverse`.

### Student’s *t* Test

Here, we will use a built-in data set in R, `reshape2::tips`, which
contains a record on the tips a waiter received over several months in
restaurant.

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

For example, we might be curious whether male or female guests give
higher tips. In base R, we could go with this one-liner.

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

However, if we want to perform this test by weekday, we are left with
subsetting the data (manually) by day and apply the following on each
subset.

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

We would certainly prefer a neat table with the parameters of the fit
organized as columns and the rows specifying the grouping variables such
as the weekday. In this sense, a vectorized approach.

Let’s look how to perform a Student’s *t* test within the `tidyverse`.

We will use `group_modify(...)`, a summary function for cases when
`summarize(...)` is too limited. It takes a function as argument, which
operates on and returns back a `data.frame`. In this case, the function
will be given the (four) different `data.frame` objects, one for each
group (day).

> Since R’s `t.test` does not return a `data.frame` by default, we wrap
> it with `~ broom::tidy(...)`. This function makes a `data.frame`
> containing the parameters (e.g. coefficients and slopes) of the model.

> Note: The `~` in front of `broom::tidy(...)` is mandatory for reasons
> beyond the scope of this tutorial at this point.

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
the additional comparisons. The function is a little bit misleadingly
named `pairwise.t.test` and can be used for paired and unpaired
comparisons between multiple groups.

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
`lm(...)` call.

``` r
plate_data %>% 
  group_by(sample_id) %>% 
  group_modify(~ broom::tidy(lm(intensity ~ log10(concentration + 1), data = .)))
```

    ## # A tibble: 10 x 6
    ## # Groups:   sample_id [5]
    ##    sample_id   term                   estimate std.error statistic  p.value
    ##    <chr>       <chr>                     <dbl>     <dbl>     <dbl>    <dbl>
    ##  1 control     (Intercept)             -0.0744    0.157     -0.475 6.43e- 1
    ##  2 control     log10(concentration +…   0.0501    0.0971     0.515 6.16e- 1
    ##  3 treatment_A (Intercept)              0.138     1.17       0.118 9.08e- 1
    ##  4 treatment_A log10(concentration +…   2.48      0.728      3.40  5.25e- 3
    ##  5 treatment_B (Intercept)              0.234     1.11       0.211 8.37e- 1
    ##  6 treatment_B log10(concentration +…  40.3       0.687     58.7   3.91e-16
    ##  7 treatment_C (Intercept)              0.121     1.10       0.109 9.15e- 1
    ##  8 treatment_C log10(concentration +…  10.5       0.684     15.3   3.10e- 9
    ##  9 treatment_D (Intercept)              0.279     1.20       0.232 8.21e- 1
    ## 10 treatment_D log10(concentration +…   0.507     0.731      0.694 5.04e- 1

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
    ##   sample_id r.squared adj.r.squared sigma statistic  p.value    df logLik
    ##   <chr>         <dbl>         <dbl> <dbl>     <dbl>    <dbl> <int>  <dbl>
    ## 1 control      0.0217       -0.0599 0.395     0.266 6.16e- 1     2  -5.77
    ## 2 treatmen…    0.491         0.448  2.96     11.6   5.25e- 3     2 -34.0 
    ## 3 treatmen…    0.997         0.996  2.79   3451.    3.91e-16     2 -33.1 
    ## 4 treatmen…    0.951         0.947  2.78    234.    3.10e- 9     2 -33.1 
    ## 5 treatmen…    0.0459       -0.0495 2.78      0.481 5.04e- 1     2 -28.2 
    ## # … with 4 more variables: AIC <dbl>, BIC <dbl>, deviance <dbl>,
    ## #   df.residual <int>

All `broom` functions apply equally well to the output of generalized
linear and non-linear models.
