# Missing values {#missing-values}

::: {.rmdnote}
You are reading the work-in-progress second edition of R for Data Science. This chapter is currently currently a dumping ground for ideas, and we don't recommend reading it. You can find the polished first edition at <https://r4ds.had.co.nz>.
:::

## Introduction


```r
library(tidyverse)
#> -- [1mAttaching packages[22m --------------------------------------- tidyverse 1.3.1 --
#> [32mv[39m [34mggplot2[39m 3.3.5          [32mv[39m [34mpurrr  [39m 0.3.4     
#> [32mv[39m [34mtibble [39m 3.1.2          [32mv[39m [34mdplyr  [39m 1.0.7     
#> [32mv[39m [34mtidyr  [39m 1.1.3          [32mv[39m [34mstringr[39m 1.4.0.[31m9000[39m
#> [32mv[39m [34mreadr  [39m 2.0.1          [32mv[39m [34mforcats[39m 0.5.1
#> -- [1mConflicts[22m ------------------------------------------ tidyverse_conflicts() --
#> [31mx[39m [34mdplyr[39m::[32mfilter()[39m masks [34mstats[39m::filter()
#> [31mx[39m [34mdplyr[39m::[32mlag()[39m    masks [34mstats[39m::lag()
```

Missing topics:

-   Missing values generated from matching data frames (i.e. `left_join()` and `anti_join()`

-   Last observation carried forward and `tidy::fill()`

-   `coalesce()` and `na_if()`

## Basics

### Missing values {#missing-values-filter}

One important feature of R that can make comparison tricky is missing values, or `NA`s ("not availables").
`NA` represents an unknown value so missing values are "contagious": almost any operation involving an unknown value will also be unknown.


```r
NA > 5
#> [1] NA
10 == NA
#> [1] NA
NA + 10
#> [1] NA
NA / 2
#> [1] NA
```

The most confusing result is this one:


```r
NA == NA
#> [1] NA
```

It's easiest to understand why this is true with a bit more context:


```r
# Let x be Mary's age. We don't know how old she is.
x <- NA

# Let y be John's age. We don't know how old he is.
y <- NA

# Are John and Mary the same age?
x == y
#> [1] NA
# We don't know!
```

If you want to determine if a value is missing, use `is.na()`:


```r
is.na(x)
#> [1] TRUE
```

### Exercises

1.  How many flights have a missing `dep_time`?
    What other variables are missing?
    What might these rows represent?

2.  How could you use `arrange()` to sort all missing values to the start?
    (Hint: use `!is.na()`).

3.  Come up with another approach that will give you the same output as `not_cancelled %>% count(dest)` and `not_cancelled %>% count(tailnum, wt = distance)` (without using `count()`).

4.  Look at the number of cancelled flights per day.
    Is there a pattern?
    Is the proportion of cancelled flights related to the average delay?

## Explicit vs implicit missing values {#missing-values-tidy}

Changing the representation of a dataset brings up an important subtlety of missing values.
Surprisingly, a value can be missing in one of two possible ways:

-   **Explicitly**, i.e. flagged with `NA`.
-   **Implicitly**, i.e. simply not present in the data.

Let's illustrate this idea with a very simple data set:


```r
stocks <- tibble(
  year   = c(2015, 2015, 2015, 2015, 2016, 2016, 2016),
  qtr    = c(   1,    2,    3,    4,    2,    3,    4),
  return = c(1.88, 0.59, 0.35,   NA, 0.92, 0.17, 2.66)
)
```

There are two missing values in this dataset:

-   The return for the fourth quarter of 2015 is explicitly missing, because the cell where its value should be instead contains `NA`.

-   The return for the first quarter of 2016 is implicitly missing, because it simply does not appear in the dataset.

One way to think about the difference is with this Zen-like koan: An explicit missing value is the presence of an absence; an implicit missing value is the absence of a presence.

The way that a dataset is represented can make implicit values explicit.
For example, we can make the implicit missing value explicit by putting years in the columns:


```r
stocks %>%
  pivot_wider(names_from = year, values_from = return)
#> [90m# A tibble: 4 x 3[39m
#>     [1mqtr[22m [1m`2015`[22m [1m`2016`[22m
#>   [3m[90m<dbl>[39m[23m  [3m[90m<dbl>[39m[23m  [3m[90m<dbl>[39m[23m
#> [90m1[39m     1   1.88  [31mNA[39m   
#> [90m2[39m     2   0.59   0.92
#> [90m3[39m     3   0.35   0.17
#> [90m4[39m     4  [31mNA[39m      2.66
```

Because these explicit missing values may not be important in other representations of the data, you can set `values_drop_na = TRUE` in `pivot_longer()` to turn explicit missing values implicit:


```r
stocks %>%
  pivot_wider(names_from = year, values_from = return) %>%
  pivot_longer(
    cols = c(`2015`, `2016`),
    names_to = "year",
    values_to = "return",
    values_drop_na = TRUE
  )
#> [90m# A tibble: 6 x 3[39m
#>     [1mqtr[22m [1myear[22m  [1mreturn[22m
#>   [3m[90m<dbl>[39m[23m [3m[90m<chr>[39m[23m  [3m[90m<dbl>[39m[23m
#> [90m1[39m     1 2015    1.88
#> [90m2[39m     2 2015    0.59
#> [90m3[39m     2 2016    0.92
#> [90m4[39m     3 2015    0.35
#> [90m5[39m     3 2016    0.17
#> [90m6[39m     4 2016    2.66
```

Another important tool for making missing values explicit in tidy data is `complete()`:


```r
stocks %>%
  complete(year, qtr)
#> [90m# A tibble: 8 x 3[39m
#>    [1myear[22m   [1mqtr[22m [1mreturn[22m
#>   [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m  [3m[90m<dbl>[39m[23m
#> [90m1[39m  [4m2[24m015     1   1.88
#> [90m2[39m  [4m2[24m015     2   0.59
#> [90m3[39m  [4m2[24m015     3   0.35
#> [90m4[39m  [4m2[24m015     4  [31mNA[39m   
#> [90m5[39m  [4m2[24m016     1  [31mNA[39m   
#> [90m6[39m  [4m2[24m016     2   0.92
#> [90m# ... with 2 more rows[39m
```

`complete()` takes a set of columns, and finds all unique combinations.
It then ensures the original dataset contains all those values, filling in explicit `NA`s where necessary.

There's one other important tool that you should know for working with missing values.
Sometimes when a data source has primarily been used for data entry, missing values indicate that the previous value should be carried forward:


```r
treatment <- tribble(
  ~person,           ~treatment, ~response,
  "Derrick Whitmore", 1,         7,
  NA,                 2,         10,
  NA,                 3,         9,
  "Katherine Burke",  1,         4
)
```

You can fill in these missing values with `fill()`.
It takes a set of columns where you want missing values to be replaced by the most recent non-missing value (sometimes called last observation carried forward).


```r
treatment %>%
  fill(person)
#> [90m# A tibble: 4 x 3[39m
#>   [1mperson[22m           [1mtreatment[22m [1mresponse[22m
#>   [3m[90m<chr>[39m[23m                [3m[90m<dbl>[39m[23m    [3m[90m<dbl>[39m[23m
#> [90m1[39m Derrick Whitmore         1        7
#> [90m2[39m Derrick Whitmore         2       10
#> [90m3[39m Derrick Whitmore         3        9
#> [90m4[39m Katherine Burke          1        4
```

### Exercises

1.  Compare and contrast the `fill` arguments to `pivot_wider()` and `complete()`.

2.  What does the direction argument to `fill()` do?

## dplyr verbs

`filter()` only includes rows where the condition is `TRUE`; it excludes both `FALSE` and `NA` values.
If you want to preserve missing values, ask for them explicitly:


```r
df <- tibble(x = c(1, NA, 3))
filter(df, x > 1)
#> [90m# A tibble: 1 x 1[39m
#>       [1mx[22m
#>   [3m[90m<dbl>[39m[23m
#> [90m1[39m     3
filter(df, is.na(x) | x > 1)
#> [90m# A tibble: 2 x 1[39m
#>       [1mx[22m
#>   [3m[90m<dbl>[39m[23m
#> [90m1[39m    [31mNA[39m
#> [90m2[39m     3
```

Missing values are always sorted at the end:


```r
df <- tibble(x = c(5, 2, NA))
arrange(df, x)
#> [90m# A tibble: 3 x 1[39m
#>       [1mx[22m
#>   [3m[90m<dbl>[39m[23m
#> [90m1[39m     2
#> [90m2[39m     5
#> [90m3[39m    [31mNA[39m
arrange(df, desc(x))
#> [90m# A tibble: 3 x 1[39m
#>       [1mx[22m
#>   [3m[90m<dbl>[39m[23m
#> [90m1[39m     5
#> [90m2[39m     2
#> [90m3[39m    [31mNA[39m
```

## Exercises

1.  Why is `NA ^ 0` not missing? Why is `NA | TRUE` not missing? Why is `FALSE & NA` not missing? Can you figure out the general rule? (`NA * 0` is a tricky counterexample!)
