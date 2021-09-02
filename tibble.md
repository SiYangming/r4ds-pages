# Tibbles

## Introduction

Throughout this book we work with "tibbles" instead of R's traditional `data.frame`.
Tibbles *are* data frames, but they tweak some older behaviours to make life a little easier.
R is an old language, and some things that were useful 10 or 20 years ago now get in your way.
It's difficult to change base R without breaking existing code, so most innovation occurs in packages.
Here we will describe the **tibble** package, which provides opinionated data frames that make working in the tidyverse a little easier.
In most places, I'll use the term tibble and data frame interchangeably; when I want to draw particular attention to R's built-in data frame, I'll call them `data.frame`s.

If this chapter leaves you wanting to learn more about tibbles, you might enjoy `vignette("tibble")`.

### Prerequisites

In this chapter we'll explore the **tibble** package, part of the core tidyverse.


```r
library(tidyverse)
```

## Creating tibbles

Almost all of the functions that you'll use in this book produce tibbles, as tibbles are one of the unifying features of the tidyverse.
Most other R packages use regular data frames, so you might want to coerce a data frame to a tibble.
You can do that with `as_tibble()`:


```r
as_tibble(mtcars)
#> [90m# A tibble: 32 x 11[39m
#>     [1mmpg[22m   [1mcyl[22m  [1mdisp[22m    [1mhp[22m  [1mdrat[22m    [1mwt[22m  [1mqsec[22m    [1mvs[22m    [1mam[22m  [1mgear[22m  [1mcarb[22m
#>   [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m
#> [90m1[39m  21       6   160   110  3.9   2.62  16.5     0     1     4     4
#> [90m2[39m  21       6   160   110  3.9   2.88  17.0     0     1     4     4
#> [90m3[39m  22.8     4   108    93  3.85  2.32  18.6     1     1     4     1
#> [90m4[39m  21.4     6   258   110  3.08  3.22  19.4     1     0     3     1
#> [90m5[39m  18.7     8   360   175  3.15  3.44  17.0     0     0     3     2
#> [90m6[39m  18.1     6   225   105  2.76  3.46  20.2     1     0     3     1
#> [90m# ... with 26 more rows[39m
```

You can create a new tibble from individual vectors with `tibble()`.
`tibble()` will automatically recycle inputs of length 1, and allows you to refer to variables that you just created, as shown below.


```r
tibble(
  x = 1:5, 
  y = 1, 
  z = x ^ 2 + y
)
#> [90m# A tibble: 5 x 3[39m
#>       [1mx[22m     [1my[22m     [1mz[22m
#>   [3m[90m<int>[39m[23m [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m
#> [90m1[39m     1     1     2
#> [90m2[39m     2     1     5
#> [90m3[39m     3     1    10
#> [90m4[39m     4     1    17
#> [90m5[39m     5     1    26
```

If you're already familiar with `data.frame()`, note that `tibble()` does much less: it never changes the type of the inputs (e.g. it never converts strings to factors!), it never changes the names of variables, and it never creates row names.

It's possible for a tibble to have column names that are not valid R variable names, aka **non-syntactic** names.
For example, they might not start with a letter, or they might contain unusual characters like a space.
To refer to these variables, you need to surround them with backticks, `` ` ``:


```r
tb <- tibble(
  `:)` = "smile", 
  ` ` = "space",
  `2000` = "number"
)
tb
#> [90m# A tibble: 1 x 3[39m
#>   [1m`:)`[22m  [1m` `[22m   [1m`2000`[22m
#>   [3m[90m<chr>[39m[23m [3m[90m<chr>[39m[23m [3m[90m<chr>[39m[23m 
#> [90m1[39m smile space number
```

You'll also need the backticks when working with these variables in other packages, like ggplot2, dplyr, and tidyr.

Another way to create a tibble is with `tribble()`, short for **tr**ansposed tibble.
`tribble()` is customised for data entry in code: column headings are defined by formulas (i.e. they start with `~`), and entries are separated by commas.
This makes it possible to lay out small amounts of data in easy to read form.


```r
tribble(
  ~x, ~y, ~z,
  #--|--|----
  "a", 2, 3.6,
  "b", 1, 8.5
)
#> [90m# A tibble: 2 x 3[39m
#>   [1mx[22m         [1my[22m     [1mz[22m
#>   [3m[90m<chr>[39m[23m [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m
#> [90m1[39m a         2   3.6
#> [90m2[39m b         1   8.5
```

I often add a comment (the line starting with `#`), to make it really clear where the header is.

## Tibbles vs. data.frame

There are two main differences in the usage of a tibble vs. a classic `data.frame`: printing and subsetting.

### Printing

Tibbles have a refined print method that shows only the first 10 rows, and all the columns that fit on screen.
This makes it much easier to work with large data.
In addition to its name, each column reports its type, a nice feature borrowed from `str()`:


```r
tibble(
  a = lubridate::now() + runif(1e3) * 86400,
  b = lubridate::today() + runif(1e3) * 30,
  c = 1:1e3,
  d = runif(1e3),
  e = sample(letters, 1e3, replace = TRUE)
)
#> [90m# A tibble: 1,000 x 5[39m
#>   [1ma[22m                   [1mb[22m              [1mc[22m     [1md[22m [1me[22m    
#>   [3m[90m<dttm>[39m[23m              [3m[90m<date>[39m[23m     [3m[90m<int>[39m[23m [3m[90m<dbl>[39m[23m [3m[90m<chr>[39m[23m
#> [90m1[39m 2021-09-03 [90m03:13:05[39m 2021-09-10     1 0.368 n    
#> [90m2[39m 2021-09-03 [90m21:18:15[39m 2021-09-15     2 0.612 l    
#> [90m3[39m 2021-09-03 [90m15:41:54[39m 2021-09-25     3 0.415 p    
#> [90m4[39m 2021-09-03 [90m05:03:11[39m 2021-09-24     4 0.212 m    
#> [90m5[39m 2021-09-03 [90m01:27:28[39m 2021-09-21     5 0.733 i    
#> [90m6[39m 2021-09-03 [90m12:28:25[39m 2021-09-17     6 0.460 n    
#> [90m# ... with 994 more rows[39m
```

Tibbles are designed so that you don't accidentally overwhelm your console when you print large data frames.
But sometimes you need more output than the default display.
There are a few options that can help.

First, you can explicitly `print()` the data frame and control the number of rows (`n`) and the `width` of the display.
`width = Inf` will display all columns:


```r
nycflights13::flights %>% 
  print(n = 10, width = Inf)
```

You can also control the default print behaviour by setting options:

-   `options(tibble.print_max = n, tibble.print_min = m)`: if more than `n` rows, print only `m` rows.
    Use `options(tibble.print_min = Inf)` to always show all rows.

-   Use `options(tibble.width = Inf)` to always print all columns, regardless of the width of the screen.

You can see a complete list of options by looking at the package help with `package?tibble`.

A final option is to use RStudio's built-in data viewer to get a scrollable view of the complete dataset.
This is also often useful at the end of a long chain of manipulations.


```r
nycflights13::flights %>% 
  View()
```

### Subsetting

So far all the tools you've learned have worked with complete data frames.
If you want to pull out a single variable, you need some new tools, `$` and `[[`.
`[[` can extract by name or position; `$` only extracts by name but is a little less typing.


```r
df <- tibble(
  id = LETTERS[1:5],
  x  = 1:5,
  y  = 6:10
)

# Extract by name
df$x
#> [1] 1 2 3 4 5
df[["x"]]
#> [1] 1 2 3 4 5

# Extract by position
df[[1]]
#> [1] "A" "B" "C" "D" "E"
```

To use these in a pipe, you'll need to use the special placeholder `.`:


```r
df %>% .$x
#> [1] 1 2 3 4 5
df %>% .[["x"]]
#> [1] 1 2 3 4 5
```

Alternatively, you can use the `pull()` function that is specifically designed to extract a variable from a data frame.
`pull()` also takes an optional `name` argument that specifies the column to be used as names for a named vector.


```r
df %>% pull(x)
#> [1] 1 2 3 4 5
df %>% pull(x, name = id)
#> A B C D E 
#> 1 2 3 4 5
```

Compared to a `data.frame`, tibbles are more strict: they never do partial matching, and they will generate a warning if the column you are trying to access does not exist.

## Interacting with older code

Some older functions don't work with tibbles.
If you encounter one of these functions, use `as.data.frame()` to turn a tibble back to a `data.frame`:


```r
class(as.data.frame(tb))
#> [1] "data.frame"
```

The main reason that some older functions don't work with tibble is the `[` function.
We don't use `[` much in this book because `dplyr::filter()` and `dplyr::select()` allow you to solve the same problems with clearer code (but you will learn a little about it in [vector subsetting](#vector-subsetting)).
With base R data frames, `[` sometimes returns a data frame, and sometimes returns a vector.
With tibbles, `[` always returns another tibble.

## Exercises

1.  How can you tell if an object is a tibble?
    (Hint: try printing `mtcars`, which is a regular data frame).

2.  Compare and contrast the following operations on a `data.frame` and equivalent tibble.
    What is different?
    Why might the default data frame behaviours cause you frustration?

    
    ```r
    df <- data.frame(abc = 1, xyz = "a")
    df$x
    df[, "xyz"]
    df[, c("abc", "xyz")]
    ```

3.  If you have the name of a variable stored in an object, e.g. `var <- "mpg"`, how can you extract the reference variable from a tibble?

4.  Practice referring to non-syntactic names in the following data frame by:

    a.  Extracting the variable called `1`.
    b.  Plotting a scatterplot of `1` vs `2`.
    c.  Creating a new column called `3` which is `2` divided by `1`.
    d.  Renaming the columns to `one`, `two` and `three`.

    
    ```r
    annoying <- tibble(
      `1` = 1:10,
      `2` = `1` * 2 + rnorm(length(`1`))
    )
    ```

5.  What does `tibble::enframe()` do?
    When might you use it?

6.  What option controls how many additional column names are printed at the footer of a tibble?
