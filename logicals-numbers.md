# Logicals and numbers {#logicals-numbers}

::: {.rmdnote}
You are reading the work-in-progress second edition of R for Data Science. This chapter is currently currently a dumping ground for ideas, and we don't recommend reading it. You can find the polished first edition at <https://r4ds.had.co.nz>.
:::

## Introduction

`between()`


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
library(nycflights13)
```

## Logical operators

Multiple arguments to `filter()` are combined with "and": every expression must be true in order for a row to be included in the output.
For other types of combinations, you'll need to use Boolean operators yourself: `&` is "and", `|` is "or", and `!` is "not".
Figure \@ref(fig:bool-ops) shows the complete set of Boolean operations.

<div class="figure" style="text-align: center">
<img src="diagrams/transform-logical.png" alt="Six Venn diagrams, each explaining a given logical operator. The circles (sets) in each of the Venn diagrams represent x and y. 1. y &amp; !x is y but none of x, x &amp; y is the intersection of x and y, x &amp; !y is x but none of y, x is all of x none of y, xor(x, y) is everything except the intersection of x and y, y is all of y none of x, and x | y is everything." width="70%" />
<p class="caption">(\#fig:bool-ops)Complete set of boolean operations. `x` is the left-hand circle, `y` is the right-hand circle, and the shaded region show which parts each operator selects.</p>
</div>

The following code finds all flights that departed in November or December:


```r
filter(flights, month == 11 | month == 12)
```

The order of operations doesn't work like English.
You can't write `filter(flights, month == 11 | 12)`, which you might literally translate into "finds all flights that departed in November or December".
Instead it finds all months that equal `11 | 12`, an expression that evaluates to `TRUE`.
In a numeric context (like here), `TRUE` becomes `1`, so this finds all flights in January, not November or December.
This is quite confusing!

A useful short-hand for this problem is `x %in% y`.
This will select every row where `x` is one of the values in `y`.
We could use it to rewrite the code above:


```r
nov_dec <- filter(flights, month %in% c(11, 12))
```

Sometimes you can simplify complicated subsetting by remembering De Morgan's law: `!(x & y)` is the same as `!x | !y`, and `!(x | y)` is the same as `!x & !y`.
For example, if you wanted to find flights that weren't delayed (on arrival or departure) by more than two hours, you could use either of the following two filters:


```r
filter(flights, !(arr_delay > 120 | dep_delay > 120))
filter(flights, arr_delay <= 120, dep_delay <= 120)
```

As well as `&` and `|`, R also has `&&` and `||`.
Don't use them here!
You'll learn when you should use them in Section \@ref(conditional-execution) on conditional execution.

Whenever you start using complicated, multipart expressions in `filter()`, consider making them explicit variables instead.
That makes it much easier to check your work.
You'll learn how to create new variables shortly.

## Summaries

-   Counts and proportions of logical values: `sum(x > 10)`, `mean(y == 0)`.
    When used with numeric functions, `TRUE` is converted to 1 and `FALSE` to 0.
    This makes `sum()` and `mean()` very useful: `sum(x)` gives the number of `TRUE`s in `x`, and `mean(x)` gives the proportion.

    
    ```r
    not_cancelled <- flights %>% 
      filter(!is.na(dep_delay), !is.na(arr_delay))
    
    # How many flights left before 5am? (these usually indicate delayed
    # flights from the previous day)
    not_cancelled %>% 
      group_by(year, month, day) %>% 
      summarise(n_early = sum(dep_time < 500))
    #> `summarise()` has grouped output by 'year', 'month'. You can override using the `.groups` argument.
    #> [90m# A tibble: 365 x 4[39m
    #> [90m# Groups:   year, month [12][39m
    #>    [1myear[22m [1mmonth[22m   [1mday[22m [1mn_early[22m
    #>   [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m   [3m[90m<int>[39m[23m
    #> [90m1[39m  [4m2[24m013     1     1       0
    #> [90m2[39m  [4m2[24m013     1     2       3
    #> [90m3[39m  [4m2[24m013     1     3       4
    #> [90m4[39m  [4m2[24m013     1     4       3
    #> [90m5[39m  [4m2[24m013     1     5       3
    #> [90m6[39m  [4m2[24m013     1     6       2
    #> [90m# ... with 359 more rows[39m
    
    # What proportion of flights are delayed by more than an hour?
    not_cancelled %>% 
      group_by(year, month, day) %>% 
      summarise(hour_prop = mean(arr_delay > 60))
    #> `summarise()` has grouped output by 'year', 'month'. You can override using the `.groups` argument.
    #> [90m# A tibble: 365 x 4[39m
    #> [90m# Groups:   year, month [12][39m
    #>    [1myear[22m [1mmonth[22m   [1mday[22m [1mhour_prop[22m
    #>   [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m     [3m[90m<dbl>[39m[23m
    #> [90m1[39m  [4m2[24m013     1     1    0.072[4m2[24m
    #> [90m2[39m  [4m2[24m013     1     2    0.085[4m1[24m
    #> [90m3[39m  [4m2[24m013     1     3    0.056[4m7[24m
    #> [90m4[39m  [4m2[24m013     1     4    0.039[4m6[24m
    #> [90m5[39m  [4m2[24m013     1     5    0.034[4m9[24m
    #> [90m6[39m  [4m2[24m013     1     6    0.047[4m0[24m
    #> [90m# ... with 359 more rows[39m
    ```

`cumany()` `cumall()`

### Exercises

1.  For each plane, count the number of flights before the first delay of greater than 1 hour.

## Basic math

There are many functions for creating new variables that you can use with `mutate()`.
The key property is that the function must be vectorised: it must take a vector of values as input, return a vector with the same number of values as output.
There's no way to list every possible function that you might use, but here's a selection of functions that are frequently useful:

-   Arithmetic operators: `+`, `-`, `*`, `/`, `^`.
    These are all vectorised, using the so called "recycling rules".
    If one parameter is shorter than the other, it will be automatically extended to be the same length.
    This is most useful when one of the arguments is a single number: `air_time / 60`, `hours * 60 + minute`, etc.

    Arithmetic operators are also useful in conjunction with the aggregate functions you'll learn about later.
    For example, `x / sum(x)` calculates the proportion of a total, and `y - mean(y)` computes the difference from the mean.

-   Modular arithmetic: `%/%` (integer division) and `%%` (remainder), where `x == y * (x %/% y) + (x %% y)`.
    Modular arithmetic is a handy tool because it allows you to break integers up into pieces.
    For example, in the flights dataset, you can compute `hour` and `minute` from `dep_time` with:

    
    ```r
    transmute(flights,
      dep_time,
      hour = dep_time %/% 100,
      minute = dep_time %% 100
    )
    #> [90m# A tibble: 336,776 x 3[39m
    #>   [1mdep_time[22m  [1mhour[22m [1mminute[22m
    #>      [3m[90m<int>[39m[23m [3m[90m<dbl>[39m[23m  [3m[90m<dbl>[39m[23m
    #> [90m1[39m      517     5     17
    #> [90m2[39m      533     5     33
    #> [90m3[39m      542     5     42
    #> [90m4[39m      544     5     44
    #> [90m5[39m      554     5     54
    #> [90m6[39m      554     5     54
    #> [90m# ... with 336,770 more rows[39m
    ```

-   Logs: `log()`, `log2()`, `log10()`.
    Logarithms are an incredibly useful transformation for dealing with data that ranges across multiple orders of magnitude.
    They also convert multiplicative relationships to additive.

    All else being equal, I recommend using `log2()` because it's easy to interpret: a difference of 1 on the log scale corresponds to doubling on the original scale and a difference of -1 corresponds to halving.

-   Logical comparisons: `<`, `<=`, `>`, `>=`, `!=`, and `==`, which you learned about earlier.
    If you're doing a complex sequence of logical operations it's often a good idea to store the interim values in new variables so you can check that each step is working as expected.

-   Cumulative and rolling aggregates: R provides functions for running sums, products, mins and maxes: `cumsum()`, `cumprod()`, `cummin()`, `cummax()`; and dplyr provides `cummean()` for cumulative means.
    If you need rolling aggregates (i.e. a sum computed over a rolling window), try the RcppRoll package.

    
    ```r
    x <- 1:10
    cumsum(x)
    #>  [1]  1  3  6 10 15 21 28 36 45 55
    cummean(x)
    #>  [1] 1.0 1.5 2.0 2.5 3.0 3.5 4.0 4.5 5.0 5.5
    ```

### Recycling rules

Base R.

Tidyverse.

## Summaries

Just using means, counts, and sum can get you a long way, but R provides many other useful summary functions:

-   Measures of location: we've used `mean(x)`, but `median(x)` is also useful.
    The mean is the sum divided by the length; the median is a value where 50% of `x` is above it, and 50% is below it.

    
    ```r
    not_cancelled %>%
      group_by(month) %>%
      summarise(
        med_arr_delay = median(arr_delay),
        med_dep_delay = median(dep_delay)
        )
    #> [90m# A tibble: 12 x 3[39m
    #>   [1mmonth[22m [1mmed_arr_delay[22m [1mmed_dep_delay[22m
    #>   [3m[90m<int>[39m[23m         [3m[90m<dbl>[39m[23m         [3m[90m<dbl>[39m[23m
    #> [90m1[39m     1            -[31m3[39m            -[31m2[39m
    #> [90m2[39m     2            -[31m3[39m            -[31m2[39m
    #> [90m3[39m     3            -[31m6[39m            -[31m1[39m
    #> [90m4[39m     4            -[31m2[39m            -[31m2[39m
    #> [90m5[39m     5            -[31m8[39m            -[31m1[39m
    #> [90m6[39m     6            -[31m2[39m             0
    #> [90m# ... with 6 more rows[39m
    ```

    It's sometimes useful to combine aggregation with logical subsetting.
    We haven't talked about this sort of subsetting yet, but you'll learn more about it in Section \@ref(vector-subsetting).

    
    ```r
    not_cancelled %>% 
      group_by(year, month, day) %>% 
      summarise(
        avg_delay1 = mean(arr_delay),
        avg_delay2 = mean(arr_delay[arr_delay > 0]) # the average positive delay
      )
    #> `summarise()` has grouped output by 'year', 'month'. You can override using the `.groups` argument.
    #> [90m# A tibble: 365 x 5[39m
    #> [90m# Groups:   year, month [12][39m
    #>    [1myear[22m [1mmonth[22m   [1mday[22m [1mavg_delay1[22m [1mavg_delay2[22m
    #>   [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m      [3m[90m<dbl>[39m[23m      [3m[90m<dbl>[39m[23m
    #> [90m1[39m  [4m2[24m013     1     1      12.7        32.5
    #> [90m2[39m  [4m2[24m013     1     2      12.7        32.0
    #> [90m3[39m  [4m2[24m013     1     3       5.73       27.7
    #> [90m4[39m  [4m2[24m013     1     4      -[31m1[39m[31m.[39m[31m93[39m       28.3
    #> [90m5[39m  [4m2[24m013     1     5      -[31m1[39m[31m.[39m[31m53[39m       22.6
    #> [90m6[39m  [4m2[24m013     1     6       4.24       24.4
    #> [90m# ... with 359 more rows[39m
    ```

-   Measures of spread: `sd(x)`, `IQR(x)`, `mad(x)`.
    The root mean squared deviation, or standard deviation `sd(x)`, is the standard measure of spread.
    The interquartile range `IQR(x)` and median absolute deviation `mad(x)` are robust equivalents that may be more useful if you have outliers.

    
    ```r
    # Why is distance to some destinations more variable than to others?
    not_cancelled %>% 
      group_by(dest) %>% 
      summarise(distance_sd = sd(distance)) %>% 
      arrange(desc(distance_sd))
    #> [90m# A tibble: 104 x 2[39m
    #>   [1mdest[22m  [1mdistance_sd[22m
    #>   [3m[90m<chr>[39m[23m       [3m[90m<dbl>[39m[23m
    #> [90m1[39m EGE         10.5 
    #> [90m2[39m SAN         10.4 
    #> [90m3[39m SFO         10.2 
    #> [90m4[39m HNL         10.0 
    #> [90m5[39m SEA          9.98
    #> [90m6[39m LAS          9.91
    #> [90m# ... with 98 more rows[39m
    ```

-   Measures of rank: `min(x)`, `quantile(x, 0.25)`, `max(x)`.
    Quantiles are a generalisation of the median.
    For example, `quantile(x, 0.25)` will find a value of `x` that is greater than 25% of the values, and less than the remaining 75%.

    
    ```r
    # When do the first and last flights leave each day?
    not_cancelled %>% 
      group_by(year, month, day) %>% 
      summarise(
        first = min(dep_time),
        last = max(dep_time)
      )
    #> `summarise()` has grouped output by 'year', 'month'. You can override using the `.groups` argument.
    #> [90m# A tibble: 365 x 5[39m
    #> [90m# Groups:   year, month [12][39m
    #>    [1myear[22m [1mmonth[22m   [1mday[22m [1mfirst[22m  [1mlast[22m
    #>   [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m
    #> [90m1[39m  [4m2[24m013     1     1   517  [4m2[24m356
    #> [90m2[39m  [4m2[24m013     1     2    42  [4m2[24m354
    #> [90m3[39m  [4m2[24m013     1     3    32  [4m2[24m349
    #> [90m4[39m  [4m2[24m013     1     4    25  [4m2[24m358
    #> [90m5[39m  [4m2[24m013     1     5    14  [4m2[24m357
    #> [90m6[39m  [4m2[24m013     1     6    16  [4m2[24m355
    #> [90m# ... with 359 more rows[39m
    ```

### Exercises

1.  Brainstorm at least 5 different ways to assess the typical delay characteristics of a group of flights.
    Consider the following scenarios:

    -   A flight is 15 minutes early 50% of the time, and 15 minutes late 50% of the time.

    -   A flight is always 10 minutes late.

    -   A flight is 30 minutes early 50% of the time, and 30 minutes late 50% of the time.

    -   99% of the time a flight is on time.
        1% of the time it's 2 hours late.

    Which is more important: arrival delay or departure delay?

## Floating point

There's another common problem you might encounter when using `==`: floating point numbers.
These results might surprise you!


```r
(sqrt(2) ^ 2) == 2
#> [1] FALSE
(1 / 49 * 49) == 1
#> [1] FALSE
```

Computers use finite precision arithmetic (they obviously can't store an infinite number of digits!) so remember that every number you see is an approximation.
Instead of relying on `==`, use `near()`:


```r
near(sqrt(2) ^ 2,  2)
#> [1] TRUE
near(1 / 49 * 49, 1)
#> [1] TRUE
```

## Exercises

1.  What trigonometric functions does R provide?
2.  
