Working with Strings
================

This introduction will be on objects of type `character`. This is all
text which appears between quotation marks (`""` or `''`) and is not
executed by R. In informatics, such text objects are called ‘strings’.

Base R has a set of functions to combine and split strings, to find and
replace text within strings etc. For the sake of consistency and its
more powerful language, we will limit our discussion to the equivalent
functions of the `stringr` package for more complex string operations.

## Properties of Strings

Note how we get the “length” of a string in R, i.e. the number of
characters.

``` r
# number of characters
nchar("apple")
```

    ## [1] 5

``` r
# number of elements in the character vector
length("apple")
```

    ## [1] 1

``` r
fruits <- c("apple", "banana", "lemon")
# number of elements in the character vector
length(fruits)
```

    ## [1] 3

``` r
# number of characters in each element of the vector vs length of each element in the vector
sapply(fruits, nchar); sapply(fruits, length)
```

    ##  apple banana  lemon 
    ##      5      6      5

    ##  apple banana  lemon 
    ##      1      1      1

The `stringr` equivalent of `nchar("apple")` is
`stringr::str_length("apple")`.

## Combining Strings

The `paste(...)` contracts strings (e.g. two `character` objects) into a
single string (one `charachter` object).

``` r
paste("banana", "cake")
```

    ## [1] "banana cake"

If two vectors of different length are supplied, the shorter is repeated
over the length of the longer one.

``` r
paste(c("banana", "chocolate", "doughnut"), "cake")
```

    ## [1] "banana cake"    "chocolate cake" "doughnut cake"

Here is an illustration of the agruments `sep = ...` and `collapse =
...`.

``` r
x <- paste(c("A", "B", "C", "D"), c(1, 2), sep = "_")
x
```

    ## [1] "A_1" "B_2" "C_1" "D_2"

``` r
length(x) # a vector with four elements
```

    ## [1] 4

``` r
x <- paste(c("A", "B", "C", "D"), c(1, 2), sep = "_", collapse = " and ")
x
```

    ## [1] "A_1 and B_2 and C_1 and D_2"

``` r
length(x) # a vector with one element
```

    ## [1] 1

The default separator of `paste` is a space `sep = " "`. `paste0(...)`
is a shortcut for `paste(..., sep = "")`, which glues all arguments
without spaces.

The `stringr` equivalent of `paste("pipet", "boy")` is
`stringr::str_c("pipet", "boy")` with the default separator being no
space `sep = ""`. It has an `collapse = ...` argument too.

## Splitting Strings

Strings can be split after a specific character with base R’s `strsplit`
function.

``` r
strsplit(x = "one,2;three,4", split = ",")
```

    ## [[1]]
    ## [1] "one"     "2;three" "4"

In the `stringr` package, `stringr::str_split` is vectorized over either
arguments, `string` or
`pattern`.

``` r
stringr::str_split(string = c("one,2;three,4", "five,six,7"), pattern = ",")
```

    ## [[1]]
    ## [1] "one"     "2;three" "4"      
    ## 
    ## [[2]]
    ## [1] "five" "six"  "7"

``` r
stringr::str_split(string = "one,2;three,4", pattern = c(",", ";"))
```

    ## [[1]]
    ## [1] "one"     "2;three" "4"      
    ## 
    ## [[2]]
    ## [1] "one,2"   "three,4"

## Subsetting Strings

To extract a substring from a string, base R has a function called
`substr`.

Subsetting is done by position. Negative indices (in `stringr`; not
available in base R\!) count from the end.

``` r
library(stringr)

fruits <- c("apple", "banana", "lemon")

# the second letter
str_sub(fruits, start = 2, end = 2)
```

    ## [1] "p" "a" "e"

``` r
# the third to second to last letters
str_sub(fruits, start = 3, end = -2)
```

    ## [1] "pl"  "nan" "mo"

You can use `stringr::str_sub` to modify strings too.

``` r
str_sub(fruits, start = 3, end = -2) <- "x"
fruits
```

    ## [1] "apxe" "baxa" "lexn"

To remove whitespace from the ends of a string, there is `str_trim()`.

``` r
str_trim("   chitchat  ")
```

    ## [1] "chitchat"

## Finding and Replacing Strings in Strings

In its easiest implementation, finding and replacing strings works as
follows with
`stringr::str_replace`.

``` r
str_replace(string  = "One small step for a man, one giant leap for mankind!",
            pattern = "man",
            replacement = "mouse")
```

    ## [1] "One small step for a mouse, one giant leap for mankind!"

Note that `stringr::str_replace` replaces only the *first* occurence of
a the ‘pattern’. If you want to replace *all* occurences, use
`stringr::str_replace_all`.

``` r
str_replace_all(string  = "One small step for a man, one giant leap for mankind!",
                pattern = "man",
                replacement = "mouse")
```

    ## [1] "One small step for a mouse, one giant leap for mousekind!"

## Regular Expressions

In many cases we would be glad to find and replace not only words
exactly matching our query, but also words that are constructed in a
specific manner (that have a specific ‘pattern’). The ‘language’ in
which such patterns are given is called ‘[regular
expression](https://www.regular-expressions.info/tutorial.html)’ and is
by far not limited to R, but applicable to many programming languages.

This is a brief overview with some examples modified from the `stringr`
[vignette on regular
expressions](https://stringr.tidyverse.org/articles/regular-expressions.html).

In almost all `stringr` functions, you can use regular expressions as
`pattern =
...`.

``` r
str_replace_all(string  = "One small step for a man, one giant leap for mankind!",
                pattern = "step|leap", # match either ‘step’ or ‘leap’
                replacement = "piece of cake")
```

    ## [1] "One small piece of cake for a man, one giant piece of cake for mankind!"

> In the next examples, we will use the `stringr::str_extract_all`
> function to return the substrings matching the regular expression.
> Since we provide a `character` vector with only a single element as
> `string = ...`, we immediatly index the returned list with `[[1]]` to
> save space while
printing.

``` r
str_extract_all(string  = "One small step for a man, one giant leap for mankind!", # one element only
                pattern = "step|leap")[[1]] # print it immediatly
```

    ## [1] "step" "leap"

``` r
str_extract_all(string  = c("One small step", "one giant leap"), # character vector with two elements
                pattern = "step|leap") # prints the entire list containing character vectors
```

    ## [[1]]
    ## [1] "step"
    ## 
    ## [[2]]
    ## [1] "leap"

### Placeholders

Regular expressions can have placeholder symbols. First, `.` will match
any character (except a
newline).

``` r
str_extract_all(string  = "I keep pressing the space bar, but I'm still on planet Earth.",
                pattern = ".ar")[[1]]
```

    ## [1] "bar" "Ear"

``` r
str_extract_all(string  = "I keep pressing the space bar, but I'm still on planet Earth.",
                pattern = ".ar.")[[1]]
```

    ## [1] "bar," "Eart"

> If you need to match a point, you need to use an ‘escape’ to tell the
> regular expression not to use the special meaning of “`.`”, but its
> literal meaning. The ‘escape’ sign for regular expressions is `\`, so
> you regex `\.`. However, `\` is also the escape sign for R. In order
> to *perserve* the escape sign for the evaluation of the regular
> expression, you need to escape the escape sign as well. (Sounds
> confusing.)

You will need `\\.` to match a fullstop
literally.

``` r
str_extract_all(string  = "If you obey all the rules, you miss all the fun.",
                pattern = "fun\\.")[[1]]
```

    ## [1] "fun."

### Quantifiers

To control how many times a pattern matches, you can use the repetition
operators.

| operator | repetitions |
| -------- | ----------- |
| `?`      | 0 or 1      |
| `+`      | 1 or more   |
| `*`      | 0 or more   |

``` r
x <- "1899 or MDCCCXCIX is the last year of the 1890s decade."
str_extract(x, "CC?")
```

    ## [1] "CC"

``` r
str_extract(x, "CC+")
```

    ## [1] "CCC"

``` r
str_extract_all(x, "CX*")[[1]]
```

    ## [1] "C"  "C"  "CX" "C"

You can also specify the number of matches using curly brackets.

| operator | repetitions         |
| -------- | ------------------- |
| `{n}`    | exactly *n* times   |
| `{n,}`   | *n* or more times   |
| `{n,m}`  | between *n* and *m* |

``` r
str_extract_all(x, "C{1,2}")[[1]]
```

    ## [1] "CC" "C"  "C"

By default, these matches are ‘greedy’. They will match the *longest*
string possible. To make them match the *shortest* string possible, put
a `?` after them.

| operator | repetitions                                     |
| -------- | ----------------------------------------------- |
| `??`     | 0 or 1, prefer 0                                |
| `+?`     | 1 or more, match as few as possible             |
| `*?`     | 0 or more, match as few as possible             |
| `{n,}?`  | *n* or more, match as few as possible           |
| `{n,m}?` | match at least *n*, but then as few as possible |

``` r
str_extract(x, c("C{2,3}", "C{2,3}?"))
```

    ## [1] "CCC" "CC"

### Special Characters

Escapes allow to specify individual chracters that are otherwise hard to
type.

  - `\s` matches any whitespace. This includes horizontal tabulation
    signs (`\t`), line feeds (`\n`), carriage returns (`\r`), as well as
    a variatey of space characters and other separators defined in
    Unicode.
    
    The complement `\S` matches any non-whitespace character.
    
    ``` r
    str_replace_all("If   there is\tno  space in 
    
                space,\nI will leave.  ", "\\s+", " ")
    ```
    
        ## [1] "If there is no space in space, I will leave. "

Escapes can also be useful to specify certain classes of characters.

  - `\d` matches any digit (0, 1, 2, …, 9). The complement, `\D`,
    matches any character that is not a decimal
    digit.
    
    ``` r
    str_extract_all("There are 214 moons in the solar system.", "\\d+")[[1]]
    ```
    
        ## [1] "214"
    
    ``` r
    str_extract_all(x, "\\d+")[[1]]
    ```
    
        ## [1] "1899" "1890"

  - `\w` matches any ‘word’ character. Words consist of alphabetic
    characters (including diacritics etc.) and decimal numbers.
    
    The complement `\W` matches any non-word character.
    
    ``` r
    str_extract_all("Don't shoot for the moon!", "\\w+")[[1]]
    ```
    
        ## [1] "Don"   "t"     "shoot" "for"   "the"   "moon"
    
    ``` r
    str_extract_all("Don't shoot for the moon!", "\\W+")[[1]]
    ```
    
        ## [1] "'" " " " " " " " " "!"

  - `\b` matches the transition between word and non-word characters.
    (Rarely needed; example below.)

You can specify your own *character classes* using `[]`:

  - `[abc]` matches a, b, or c,
  - `[a-z]` matches every character between a and z (case-sensitive),
  - `[^abc]` matches anything except a, b, or c.

There are a number of pre-built character classes in `stringr`:

  - `[:punct:]` punctuation,
  - `[:alpha:]` letters,
  - `[:lower:]` lowercase letters,
  - `[:upper:]` uppercase letters,
  - `[:digit:]` digits,
  - `[:alnum:]` letters and numbers,
  - `[:cntrl:]` control chacracters,
  - `[:graph:]` letters, numbers, and punctuation,
  - `[:print:]` letters, numbers, punctuation, and whitespace,
  - `[:space:]` space characters (same as `\s`),
  - `[:blank:]` space and tab.

### Anchors

In some cases, you want to match a pattern only if it occurs at the
start or the end of a line. There are

  - `^` to match the start of a line, and
  - `$` to match the end of the
line.

<!-- end list -->

``` r
str_extract_all("papaya or banana", "[yn]a")[[1]]    # all occurences of “ya” or “na”
```

    ## [1] "ya" "na" "na"

``` r
str_extract_all("papaya or banana", "[yn]a$")[[1]]   # matches at the end of a line only
```

    ## [1] "na"

``` r
str_extract_all("papaya or banana", "[yn]a\\b")[[1]] # matches at word boundaries only
```

    ## [1] "ya" "na"

To remember which one is which, [this
mnemonic](https://twitter.com/emisshula/status/323863393167613953) might
help you: ‘If you begin with power (`^`), you end up with money (`$`).’

## Learning Objectives

In this introduction you should have learned

  - how to combine and split strings,
  - how to subset strings based on character indices,
  - how to replace words in strings, and
  - that you can use regular expressions to formulate advanced string
    queries.

Quick questions.

1.  What is the result of `strsplit(paste0("AG", "CGT", "A", "TGCT"),
    "GC")`?
2.  Suppose you have the vector `c("ÁlM", "AnJ", "DaS", "DmS", "JaW",
    "ShP", "SuB")`. Using regular expressions, how do you identify all
    elements that contain
    1.  the lowercase ‘a’,
    2.  ‘a’ or ‘A’,
    3.  any non-Latin character,
    4.  more than two consonants in a row,
    5.  end (or start) with ‘S’?
