Data Types and Data Structures
================

  - [Data Types in R](#data-types-in-r)
  - [Atomic Data Structures: Vectors](#atomic-data-structures-vectors)
  - [Advanced Data Structures:
    Classes](#advanced-data-structures-classes)
  - [Vectorization in R](#vectorization-in-r)
      - [Numerical Vectors and
        Operations](#numerical-vectors-and-operations)
      - [Logical Vectors and
        Operations](#logical-vectors-and-operations)
      - [Indexing Vectors](#indexing-vectors)
      - [Indexing Lists](#indexing-lists)
      - [Applying Functions to Vectors](#applying-functions-to-vectors)
  - [Quick Questions](#quick-questions)

As in many programming languages, understanding how data are stored and
manipulated is important. In this regard, the terms ‘data structure’ and
‘data type’ refer to how R treats your data rather than how you organize
your data.

This introduction might become rather technical, but don’t get upset. It
is less important to remember everything since we will recapitulate the
contents on a needs basis during the workshop.

In this introduction you will learn

  - that there are different data types,
  - that object classes are built from these ‘atomic’ data types,
  - the basic unit in R is a vector (of one single data type), and
  - vectorization of tasks is preferred over writing loops.

## Data Types in R

Any object in R can be of *one single* data type only. The most commonly
encountered data types are shown below.

| data type   | stores                         | example                       |
| ----------- | ------------------------------ | ----------------------------- |
| `double`    | floating point numbers         | `3.141, 1.8e-3`               |
| `integer`   | integers                       | `-5L, 1L, 2L, 3L`             |
| `complex`   | complex numbers                | `1+2i, 3-1i`                  |
| `character` | strings                        | `"banana", "apple", "melon"`  |
| `logical`   | boolean values                 | `TRUE, FALSE`                 |
| `list`      | sets with different data types | `list(3, "apple")`            |
| `closure`   | functions                      | `function(x) {return(x + 1)}` |

<!-- For the sake of completeness (but irrelevant for this workshop), everything that can be typed in the R console can be stored as an R object. Therefore, there are some more data types. -->

<!-- | data type | stores | example | -->

<!-- |-----------|--------|---------| -->

<!-- | `environment` | environments | `new.env()` | -->

<!-- | `language` | primitive expressions | `sapply(expression(x = 1 + 1, 2 + 3), typeof)` | -->

<!-- | `expression` | a set of (incoherent) expressions | `typeof(expression(x = 1 + 1, 2 + 3))` | -->

> By convention, if you type any number (including integers without an
> `L` suffix), R will store it as `double`. This allows to do accurate
> maths with it. `double` stands for ‘double-precision floating-point
> format’.

The data type can be queried using the `typeof(...)` function.

``` r
x <- pi
typeof(x)
```

    ## [1] "double"

There are some special values, R objects can take,

  - `NULL` for empty objects; their data type is eradicated (`NULL`)
    too,
  - `NA`, which is used to designate a missing value; their data type is
    preserved, and
  - `NaN` and `Inf` to designate non-numerical results of certain
    computations with `double` or `complex` objects.

## Atomic Data Structures: Vectors

In R, multiple objects of the same type can be *concatenated* (or
*combined*) to vectors of the same data type using the `c(...)`
function. Or, to put it the other way round, in R, every object is at
least a vector with length one.

``` r
x <- c(1, 2, 3)
typeof(x)
length(x)

x <- c(c(1, 2, 3), c(4, 5, 6))
typeof(x)
length(x)

x <- c("a", "b", "c")
typeof(x)
length(x)

x <- c(TRUE, FALSE, FALSE, TRUE)
typeof(x)
length(x)

x <- c(list("a", 1), list("b", 2), list("c", 3))
typeof(x)
length(x)
```

Elements in vectors can be *named*.

``` r
# names do not need to be quoted necessarily
noble_gases <- c(He = 2, "Ne" = 10, Ar = 18, Kr = 36, Xe = 54)
noble_gases
```

    ## He Ne Ar Kr Xe 
    ##  2 10 18 36 54

``` r
# get the names as character vector
names(noble_gases)
```

    ## [1] "He" "Ne" "Ar" "Kr" "Xe"

``` r
# set the names with another character vector
names(noble_gases) <- c("Helium", "Neon", "Argon", "Krypton", "Xenon")
noble_gases
```

    ##  Helium    Neon   Argon Krypton   Xenon 
    ##       2      10      18      36      54

We can add another entry by concatenation.

``` r
noble_gases <- c(noble_gases, "Radon" = 86)
```

When we try to concatenate *objects of different data types*, the
objects with the ‘least compatible’ data type will be *coerced* to an
object of the ‘most compatible’ data type. The ordering is roughly
`logical` \< `integer` \< `double` \< `complex` \< `character` \<
`list`.

``` r
# coercion to character
c(1, "a")
c(TRUE, "a")

# coercion to double
c(1L, 2.5)

# coercion to integer
x <- c(1L, TRUE, FALSE)
typeof(x)
```

Thus, to preserve the original data type of each element, we need to use
lists\!

We can also ‘force’ R to try this backwards.

``` r
library(magrittr)

as.double("1") %>% typeof # double
as.double("a") %>% typeof # NA of type double

as.logical("TRUE") %>% typeof # logical
```

## Advanced Data Structures: Classes

R allows to create more sophisticated data structures from the simpler
atomic data types. The structure of such objectes is called a ‘class’.
Besides its usefulness for an object-oriented programming style, having
a class associated to an object will cause functions such as
`print(...)`, `plot(...)` and `summary(...)` to respond in an
appropriate manner. Some functions operate only on objects of specific
classes.

> All objects in R can have additional “metadata” associated with them,
> so-called ‘attributes’. Classes define which ‘attributes’ an object
> must (at least) implement to belong to a class.

Here are two examples to illustrate the concept.

1.  *‘Tables’* (class `data.frame` or `data.table`) are objects of type
    `list`.
    
      - They combine columns (each a vector with a single data type), so
        that different data types can be stored side-by-side.
      - They have attributes such as column- and rownames to reference
        each entry in the table.

2.  *‘Factors’* (class `factor`) are object of type `integer` and hold
    categorical data.
    
    In statistics, many experiments involve the recording of categorical
    data, e.g. male and female, or the different cell lines, treatments
    etc. used in an experiment. As they would be stored as `character`,
    this would consume a lot of memory.
    
      - Each category (‘level’) is assigned a number. Instead of the
        `character` vector, a much smaller `integer` vector is stored.
      - The assignments are saved as an attribute to the `factor`
        object.
    
    If you look for an example, here you go.
    
    ``` r
    x <- c("m", "f", "m", "m", "f", "m", "d")
    x
    ```
    
        ## [1] "m" "f" "m" "m" "f" "m" "d"
    
    ``` r
    typeof(x)
    ```
    
        ## [1] "character"
    
    ``` r
    x <- factor(x)
    x
    ```
    
        ## [1] m f m m f m d
        ## Levels: d f m
    
    ``` r
    typeof(x)
    ```
    
        ## [1] "integer"
    
    ``` r
    # get the integer representation of the factor
    as.integer(x)
    ```
    
        ## [1] 3 2 3 3 2 3 1
    
    ``` r
    # get the assignments of the levels
    levels(x) # short-hand for objects of class ‘factor’
    ```
    
        ## [1] "d" "f" "m"

You can get (and modify) the object’s class with the `class(...)`
function and the attributes associated with it using the
`attributes(...)` function.

> Having said that `print(...)` is aware of an object’s class, you
> should note: The *good thing* is that when you print the object to the
> console (e.g. implicitly by typing the object’s name), you will see
> the important aspects of the object. The *bad thing* is that you can
> easily think that you are seeing the real object. In reality, you are
> just seeing the self-portrait of the object that it wants you to see.

## Vectorization in R

Recall that the basic unit in R is a vector. Sometimes there will be
dozens, sometimes millions of elements stored in a single vector.
Vectorization makes sure an operation treats the object as a whole
rather than each element in the vector separately.

### Numerical Vectors and Operations

There are some convenience expressions to create regular sequences of
numeric objects.

``` r
# create a vector of type integer from 1 to 10 ...
x <- 1:10
# ... and backwards
x <- 10:1

typeof(x)
```

    ## [1] "integer"

``` r
# create a vector of type double from 1 to 5 ...
y <- seq(1, 5)
# ... spaced by 0.5 ...
y <- seq(1, 5, 0.5)

typeof(y)
```

    ## [1] "double"

``` r
# concatenate both vectors
c(x, y)
```

    ##  [1] 10.0  9.0  8.0  7.0  6.0  5.0  4.0  3.0  2.0  1.0  1.0  1.5  2.0  2.5
    ## [15]  3.0  3.5  4.0  4.5  5.0

Note that the order of the elements in each vector is preserved upon
concatenation.

-----

Operations on a vector of length one are typically done
element-by-element.

``` r
1:10 + 2
```

    ##  [1]  3  4  5  6  7  8  9 10 11 12

If the operation invovles two vectors of the same length, the operation
is applied to each pair of elements.

``` r
1:10 + 1:10
```

    ##  [1]  2  4  6  8 10 12 14 16 18 20

If the vectors are of different lengths, but one length is a multiple of
the other, R resuses the shorter vector as needed.

``` r
1:2 + 1:10
```

    ##  [1]  2  4  4  6  6  8  8 10 10 12

If the lengths are not a multiple of eachother, you will get a warning.

``` r
1:10 + 1:3
```

    ## Warning in 1:10 + 1:3: longer object length is not a multiple of shorter
    ## object length

    ##  [1]  2  4  6  5  7  9  8 10 12 11

### Logical Vectors and Operations

The operators `<` (less than), `>` (greater than), `<=`, `>=`, `==`
(equal to), and `!=` (not equal to) can be used to create logical
vectors.

``` r
x = 1:4
x > 2
```

    ## [1] FALSE FALSE  TRUE  TRUE

We can assign the result of these expressions to a new variable.

``` r
y <- x > 2
z <- x %% 2 == 1
```

The `!` sign is used to *negate* logical vectors or boolean operations.

``` r
!y
```

    ## [1]  TRUE  TRUE FALSE FALSE

The boolean operators include `&` (and), `|` (or) and `xor(...)`. They
operate element-wise.

``` r
y; z
```

    ## [1] FALSE FALSE  TRUE  TRUE

    ## [1]  TRUE FALSE  TRUE FALSE

``` r
y & z
```

    ## [1] FALSE FALSE  TRUE FALSE

``` r
y | z
```

    ## [1]  TRUE FALSE  TRUE  TRUE

``` r
xor(y, z)
```

    ## [1]  TRUE FALSE FALSE  TRUE

To test if there is at least one logical `TRUE` in a vector, there is
the `any(...)` function. Its complement is the `all(...)` function.

### Indexing Vectors

An index is used to refer to a specific element in a vector (or any
other data structure). In R, square brackets are used to perfom
indexing.

The usual form of indexing is `[`, which can select *more than one*
element.

``` r
x <- 11:20
x[4] # fourth element in the vector
```

    ## [1] 14

Numeric vectors can be used as index vectors.

``` r
x[c(1, 3, 5)]
```

    ## [1] 11 13 15

``` r
LETTERS[x]      # LETTERS is a built-in vector with A, B, C, ..., X, Y, Z
```

    ##  [1] "K" "L" "M" "N" "O" "P" "Q" "R" "S" "T"

``` r
letters[x - 10] # letters is a built-in vector with a, b, c, ..., x, y, z
```

    ##  [1] "a" "b" "c" "d" "e" "f" "g" "h" "i" "j"

Negative values refer to indices to be excluded rather than included.

``` r
x[c(-1, -3, -5)]
```

    ## [1] 12 14 16 17 18 19 20

Logical vectors can be used as index vectors.

``` r
x[x < 15]
```

    ## [1] 11 12 13 14

``` r
x[!y]
```

    ## [1] 11 12 15 16 19 20

Character vectors can be used as index vectors for named vectors. The
advantage being that alphanumeric names are often easier to remember
than numeric indices.

``` r
noble_gases[c("Krypton", "Helium")]
```

    ## Krypton  Helium 
    ##      36       2

`[[` can only be used to select a single element, dropping names.

``` r
noble_gases[["Krypton"]]
```

    ## [1] 36

### Indexing Lists

*(This section is included for the sake of completeness. You may skip it.)*

In contrast to atomic vectors, lists can be recursively indexed with
`[[`. Compare.

``` r
z <- list(A = list(a = 1, b = 2), B = 3:5)
z
```

    ## $A
    ## $A$a
    ## [1] 1
    ## 
    ## $A$b
    ## [1] 2
    ## 
    ## 
    ## $B
    ## [1] 3 4 5

``` r
unlist(z) # can be used to flatten a list of lists
```

    ## A.a A.b  B1  B2  B3 
    ##   1   2   3   4   5

``` r
# returns a list of lists with one element that contains two lists, i.e. list z subsetted
z[1]
```

    ## $A
    ## $A$a
    ## [1] 1
    ## 
    ## $A$b
    ## [1] 2

``` r
# returns a list with two elements, i.e. the first element of list z
z[[1]] 
```

    ## $a
    ## [1] 1
    ## 
    ## $b
    ## [1] 2

``` r
# returns the second element of the first element of list z
z[[c(1, 2)]]
```

    ## [1] 2

For named lists, the `$` operator can be used to browse the different
levels in a list.

``` r
# returns the second element of the first element of list z
z$A$b
```

    ## [1] 2

``` r
# same as
z$"A"$"b"
```

    ## [1] 2

``` r
# same as
z[["A"]][["b"]]
```

    ## [1] 2

### Applying Functions to Vectors

The most powerful aspect of R is the vectorization of tasks. If you are
familiar with some other programming language, this is equivalent to,
but much faster than, writing a `for`-loop (which also exists in R).

Many functions in R are *vectorized*. If applied to a vector, the
function will be applied to each of the elements in the whole vector ‘at
once’. The result will be a vector.

``` r
# the command ...
sqrt(c(4, 9, 16, 25))
# ... has the same result as ...
c(sqrt(4), sqrt(9), sqrt(16), sqrt(25))
```

If we have more complex functions to apply on a vector or list, we can
quickly make use of an `apply(...)` statement. There are six different
kinds:

  - `apply` to go along in either direction of a matrix (not to be
    discussed in this workshop),
  - `lapply`, `vapply` and `sapply` to go along vectors and lists,
    i.e. one-dimensional matrices; each of the three functions comes
    with a different set of options,
  - `tapply` to split a vector or list based on a factor into groups and
    then apply a function to each group (useful, but not discussed,
    since we will have a different approach for `data.frame` objects),
    and
  - `mapply` to go along multiple vectors or lists simultaneously (not
    discussed in this workshop).

We will have a look on `lapply` only for the moment.

``` r
x <- list(a = 1:11, b = 21:31, c = 100:200)
# calculate the mean for each element in x
lapply(x, mean)
```

    ## $a
    ## [1] 6
    ## 
    ## $b
    ## [1] 26
    ## 
    ## $c
    ## [1] 150

``` r
# calculate the mean for each element i in x squared
lapply(x, function(i) mean(i**2))
```

    ## $a
    ## [1] 46
    ## 
    ## $b
    ## [1] 686
    ## 
    ## $c
    ## [1] 23350

Have a look how the one-liner from above would look as a `for`-loop.

``` r
for (i in x) {
  
  print(mean(i**2)) # no implicit printing from within flow control statetments
  
}
```

This becomes even more tedious if we wanted to save the results as well
in a vector.

``` r
y <- x[] # copy attributes of x if any; in this case x has names

for (i in seq_along(x)) {   # need to create indexing variable manually
  
  y[[i]] <- mean(x[[i]]**2) # need to access each element individually

}

y
```

    ## $a
    ## [1] 46
    ## 
    ## $b
    ## [1] 686
    ## 
    ## $c
    ## [1] 23350

So, using `lapply` saves you a lot of work\!

## Quick Questions

1.  Which data type is the vector `c("Tagesgericht" = 1.60, "Menü 1"
    = 2.00, "Menü 2" = 2.50, "Aktionsteller" = 3.50)`?
2.  Can you assign this vector to the variable `prizes_students`? How
    could you access the costs for “Menü 2”?
3.  Which menus cost more than two euros?
4.  Your colleagues order twice the “Tagesgericht”, three times “Menü
    1”, three times “Menü 2”, and once “Aktionsteller”. The boss pays.
    How much does he or she need to pay?
5.  The canteen wants to increase the prizes by 50 cents. Calculate the
    new prizes\!
6.  The prizes of the menus for employees are
    `c(3.10, 3.50, 4.00, 5.00)` respectively. Create a new vector called
    `prizes_employees` and assign the names of `prizes_students` to the
    names of this vector.
7.  Let’s make a list containing both vectors called `prizes`. If you
    want, you can include the prizes for guests, which are 50 cents
    higher than the costs for employees. Apply the rise in prices to all
    groups\!
