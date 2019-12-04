Useful Statements
================

  - [Base R Statements](#base-r-statements)
      - [Repeating Vector Elements](#repeating-vector-elements)
      - [Sampling at Random](#sampling-at-random)
      - [Working with Dates](#working-with-dates)
  - [`tidyverse` Statements](#tidyverse-statements)
      - [Matching Nearby Ranges](#matching-nearby-ranges)

## Base R Statements

### Repeating Vector Elements

``` r
rep(c(1, 2), times = 2)
```

    ## [1] 1 2 1 2

``` r
rep(c(1, 2), each = 2)
```

    ## [1] 1 1 2 2

### Sampling at Random

Sampling random integers.

``` r
sample(1:10, size = 5, replace = TRUE)
```

    ## [1] 1 6 8 1 6

Sampling values from frequency distributions.

``` r
# from a uniform distribution
runif(5, min = 0, max = 1)
```

    ## [1] 0.4855393 0.5059194 0.4129937 0.8864714 0.5601306

``` r
# from a normal distribution
rnorm(5, mean = 0, sd = 1)
```

    ## [1]  1.799533  1.199312 -1.588946 -2.130766 -1.699861

``` r
# from a Poisson distribution
rpois(5, lambda = 2)
```

    ## [1] 3 2 4 2 4

``` r
# from a binomial distribution
rbinom(5, size = 10, prob = .5)
```

    ## [1] 3 3 3 4 7

### Working with Dates

When was the last time (and when will be the next time) in the 21st
century that 1st of December is First Advent Sunday?

``` r
year_range <- 2000:2100
year_range[which(weekdays(as.Date(paste0('0112', year_range), '%d%m%Y')) == "Sunday")]
```

    ##  [1] 2002 2013 2019 2024 2030 2041 2047 2052 2058 2069 2075 2080 2086 2097

## `tidyverse` Statements

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

### Matching Nearby Ranges

Floating point number representations are limited in their precision.
This is why `sqrt(2)^2 == 2` is `FALSE` (within the limits of your
computer’s accuracy, `.Machine$double.eps`). A a safe way of comparing
two vectors of floating point numbers is `dplyr::near`.

``` r
# within machine accuracy
near(sqrt(2)^2, 2)
```

    ## [1] TRUE

``` r
# within user-defined range
near(3, 2.6, tol = .5)
```

    ## [1] TRUE
