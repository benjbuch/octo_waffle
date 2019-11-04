Basic Interactions
================

The only meaningful way of interacting with R is by typing code into the
R console. R’s command prompt is a ‘\>’ sign. When present, R is waiting
for the next line of code.

The prompt looks like this:

    > _

In the absence of the prompt, R is busy with calculations. Anything you
type in meanwhile, will be evaluated *after* the current command has
completed.

In case a line is (syntactically) incomplete after you hit the return
key, R will continue the next line with a ‘+’ waiting for you to
complete the command.

> If at any point you want to abort the execution of a running command,
> or if you cannot figure out how to correctly complete the current line
> of input, hit Ctrl + C.

## Code and Comments

In the following, you will see pieces of R code (‘chunks’) in grey
boxes. You can copy and paste this code to your R console and execute it
by yourself. For the sake of reproducibility, some of the chunks are
directly followed by the output you should obtain on your machine. Here,
these lines are prefixed with a ‘\#\#’ mark.

In R, the ‘\#’ sign serves as comment character. Anything typed behind
this sign will be invisible to R.

## Expressions and Assignments

Code to interact with R will fall into one of two categories.

*Expressions:*

``` r
1 + sin(pi/2)
```

    ## [1] 2

``` r
nchar("math")
```

    ## [1] 4

*Assignments:*

``` r
x = 1  # right-hand side (RHS) is assigned to left-hand side (LHS)

y <- 2 # RHS is assigned to LHS
3 -> z # LHS is assigned to RHS
```

### Expressions

When expressions are typed, R will evaluate them and return the result.

Expressions can be (but are not limited to) mathematical expressions in
a colloquial sense. Thus, R can be used as a very fancy calculator. Some
of the arithmetic operators implemented in R include:

| operator    | effect             |
| ----------- | ------------------ |
| `+`         | addition           |
| `-`         | subtraction        |
| `*`         | multiplication     |
| `/`         | division           |
| `%/%`       | integer division   |
| `%%`        | modulo (remainder) |
| `^` or `**` | exponention        |

``` r
13 %/% 4
13 %% 4

# check out the result of these ‘tricky’ cases
1/0
1/0 + 1/0
(-8)^(1/3)
```

There are many (a lot of\!) built-in functions for other mathemtatical
operations including `abs(...)`, `floor(...)`, `round(...)`,
`sqrt(...)`, `exp(...)`, `log(...)`, `sin(...)`, `cos(...)`, etc. R is
designed to work out boolean and matrix algebra, and offers a large
statistical toolbox.

R follows the standard order of operations, groupings based on
parentheses.

``` r
6 + 9 / 3
(6 + 9) / 3
```