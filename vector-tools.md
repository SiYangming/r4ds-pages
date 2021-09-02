# Vector tools

::: {.rmdnote}
You are reading the work-in-progress second edition of R for Data Science. This chapter is currently currently a dumping ground for ideas, and we don't recommend reading it. You can find the polished first edition at <https://r4ds.had.co.nz>.
:::

## Introduction

`%in%`

`c()`


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

not_cancelled <- flights %>% 
  filter(!is.na(dep_delay), !is.na(arr_delay))
```

## Counts

-   Counts: You've seen `n()`, which takes no arguments, and returns the size of the current group.
    To count the number of non-missing values, use `sum(!is.na(x))`.
    To count the number of distinct (unique) values, use `n_distinct(x)`.

    
    ```r
    # Which destinations have the most carriers?
    not_cancelled %>% 
      group_by(dest) %>% 
      summarise(carriers = n_distinct(carrier)) %>% 
      arrange(desc(carriers))
    #> [90m# A tibble: 104 x 2[39m
    #>   [1mdest[22m  [1mcarriers[22m
    #>   [3m[90m<chr>[39m[23m    [3m[90m<int>[39m[23m
    #> [90m1[39m ATL          7
    #> [90m2[39m BOS          7
    #> [90m3[39m CLT          7
    #> [90m4[39m ORD          7
    #> [90m5[39m TPA          7
    #> [90m6[39m AUS          6
    #> [90m# ... with 98 more rows[39m
    ```

    Counts are so useful that dplyr provides a simple helper if all you want is a count:

    
    ```r
    not_cancelled %>% 
      count(dest)
    #> [90m# A tibble: 104 x 2[39m
    #>   [1mdest[22m      [1mn[22m
    #>   [3m[90m<chr>[39m[23m [3m[90m<int>[39m[23m
    #> [90m1[39m ABQ     254
    #> [90m2[39m ACK     264
    #> [90m3[39m ALB     418
    #> [90m4[39m ANC       8
    #> [90m5[39m ATL   [4m1[24m[4m6[24m837
    #> [90m6[39m AUS    [4m2[24m411
    #> [90m# ... with 98 more rows[39m
    ```

    Just like with `group_by()`, you can also provide multiple variables to `count()`.

    
    ```r
    not_cancelled %>% 
      count(carrier, dest)
    #> [90m# A tibble: 312 x 3[39m
    #>   [1mcarrier[22m [1mdest[22m      [1mn[22m
    #>   [3m[90m<chr>[39m[23m   [3m[90m<chr>[39m[23m [3m[90m<int>[39m[23m
    #> [90m1[39m 9E      ATL      56
    #> [90m2[39m 9E      AUS       2
    #> [90m3[39m 9E      AVL      10
    #> [90m4[39m 9E      BNA     452
    #> [90m5[39m 9E      BOS     853
    #> [90m6[39m 9E      BTV       2
    #> [90m# ... with 306 more rows[39m
    ```

    You can optionally provide a weight variable.
    For example, you could use this to "count" (sum) the total number of miles a plane flew:

    
    ```r
    not_cancelled %>% 
      count(tailnum, wt = distance)
    #> [90m# A tibble: 4,037 x 2[39m
    #>   [1mtailnum[22m      [1mn[22m
    #>   [3m[90m<chr>[39m[23m    [3m[90m<dbl>[39m[23m
    #> [90m1[39m D942DN    [4m3[24m418
    #> [90m2[39m N0EGMQ  [4m2[24m[4m3[24m[4m9[24m143
    #> [90m3[39m N10156  [4m1[24m[4m0[24m[4m9[24m664
    #> [90m4[39m N102UW   [4m2[24m[4m5[24m722
    #> [90m5[39m N103US   [4m2[24m[4m4[24m619
    #> [90m6[39m N104UW   [4m2[24m[4m4[24m616
    #> [90m# ... with 4,031 more rows[39m
    ```

## Window functions

-   Offsets: `lead()` and `lag()` allow you to refer to leading or lagging values.
    This allows you to compute running differences (e.g. `x - lag(x)`) or find when values change (`x != lag(x)`).
    They are most useful in conjunction with `group_by()`, which you'll learn about shortly.

    
    ```r
    (x <- 1:10)
    #>  [1]  1  2  3  4  5  6  7  8  9 10
    lag(x)
    #>  [1] NA  1  2  3  4  5  6  7  8  9
    lead(x)
    #>  [1]  2  3  4  5  6  7  8  9 10 NA
    ```

-   Ranking: there are a number of ranking functions, but you should start with `min_rank()`.
    It does the most usual type of ranking (e.g. 1st, 2nd, 2nd, 4th).
    The default gives smallest values the small ranks; use `desc(x)` to give the largest values the smallest ranks.

    
    ```r
    y <- c(1, 2, 2, NA, 3, 4)
    min_rank(y)
    #> [1]  1  2  2 NA  4  5
    min_rank(desc(y))
    #> [1]  5  3  3 NA  2  1
    ```

    If `min_rank()` doesn't do what you need, look at the variants `row_number()`, `dense_rank()`, `percent_rank()`, `cume_dist()`, `ntile()`.
    See their help pages for more details.

    
    ```r
    row_number(y)
    #> [1]  1  2  3 NA  4  5
    dense_rank(y)
    #> [1]  1  2  2 NA  3  4
    percent_rank(y)
    #> [1] 0.00 0.25 0.25   NA 0.75 1.00
    cume_dist(y)
    #> [1] 0.2 0.6 0.6  NA 0.8 1.0
    ```

-   Measures of position: `first(x)`, `nth(x, 2)`, `last(x)`.
    These work similarly to `x[1]`, `x[2]`, and `x[length(x)]` but let you set a default value if that position does not exist (i.e. you're trying to get the 3rd element from a group that only has two elements).
    For example, we can find the first and last departure for each day:

    
    ```r
    not_cancelled %>% 
      group_by(year, month, day) %>% 
      summarise(
        first_dep = first(dep_time), 
        last_dep = last(dep_time)
      )
    #> `summarise()` has grouped output by 'year', 'month'. You can override using the `.groups` argument.
    #> [90m# A tibble: 365 x 5[39m
    #> [90m# Groups:   year, month [12][39m
    #>    [1myear[22m [1mmonth[22m   [1mday[22m [1mfirst_dep[22m [1mlast_dep[22m
    #>   [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m     [3m[90m<int>[39m[23m    [3m[90m<int>[39m[23m
    #> [90m1[39m  [4m2[24m013     1     1       517     [4m2[24m356
    #> [90m2[39m  [4m2[24m013     1     2        42     [4m2[24m354
    #> [90m3[39m  [4m2[24m013     1     3        32     [4m2[24m349
    #> [90m4[39m  [4m2[24m013     1     4        25     [4m2[24m358
    #> [90m5[39m  [4m2[24m013     1     5        14     [4m2[24m357
    #> [90m6[39m  [4m2[24m013     1     6        16     [4m2[24m355
    #> [90m# ... with 359 more rows[39m
    ```

    These functions are complementary to filtering on ranks.
    Filtering gives you all variables, with each observation in a separate row:

    
    ```r
    not_cancelled %>% 
      group_by(year, month, day) %>% 
      mutate(r = min_rank(desc(dep_time))) %>% 
      filter(r %in% range(r))
    #> [90m# A tibble: 770 x 20[39m
    #> [90m# Groups:   year, month, day [365][39m
    #>    [1myear[22m [1mmonth[22m   [1mday[22m [1mdep_time[22m [1msched_dep_time[22m [1mdep_delay[22m [1marr_time[22m [1msched_arr_time[22m
    #>   [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m    [3m[90m<int>[39m[23m          [3m[90m<int>[39m[23m     [3m[90m<dbl>[39m[23m    [3m[90m<int>[39m[23m          [3m[90m<int>[39m[23m
    #> [90m1[39m  [4m2[24m013     1     1      517            515         2      830            819
    #> [90m2[39m  [4m2[24m013     1     1     [4m2[24m356           [4m2[24m359        -[31m3[39m      425            437
    #> [90m3[39m  [4m2[24m013     1     2       42           [4m2[24m359        43      518            442
    #> [90m4[39m  [4m2[24m013     1     2     [4m2[24m354           [4m2[24m359        -[31m5[39m      413            437
    #> [90m5[39m  [4m2[24m013     1     3       32           [4m2[24m359        33      504            442
    #> [90m6[39m  [4m2[24m013     1     3     [4m2[24m349           [4m2[24m359       -[31m10[39m      434            445
    #> [90m# ... with 764 more rows, and 12 more variables: [1marr_delay[22m <dbl>,[39m
    #> [90m#   [1mcarrier[22m <chr>, [1mflight[22m <int>, [1mtailnum[22m <chr>, [1morigin[22m <chr>, [1mdest[22m <chr>,[39m
    #> [90m#   [1mair_time[22m <dbl>, [1mdistance[22m <dbl>, [1mhour[22m <dbl>, [1mminute[22m <dbl>, [1mtime_hour[22m <dttm>,[39m
    #> [90m#   [1mr[22m <int>[39m
    ```

### dplyr


```r
flights_sml <- select(flights, 
  year:day, 
  ends_with("delay"), 
  distance, 
  air_time
)
```

-   Find the worst members of each group:

    
    ```r
    flights_sml %>% 
      group_by(year, month, day) %>%
      filter(rank(desc(arr_delay)) < 10)
    #> [90m# A tibble: 3,306 x 7[39m
    #> [90m# Groups:   year, month, day [365][39m
    #>    [1myear[22m [1mmonth[22m   [1mday[22m [1mdep_delay[22m [1marr_delay[22m [1mdistance[22m [1mair_time[22m
    #>   [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m     [3m[90m<dbl>[39m[23m     [3m[90m<dbl>[39m[23m    [3m[90m<dbl>[39m[23m    [3m[90m<dbl>[39m[23m
    #> [90m1[39m  [4m2[24m013     1     1       853       851      184       41
    #> [90m2[39m  [4m2[24m013     1     1       290       338     [4m1[24m134      213
    #> [90m3[39m  [4m2[24m013     1     1       260       263      266       46
    #> [90m4[39m  [4m2[24m013     1     1       157       174      213       60
    #> [90m5[39m  [4m2[24m013     1     1       216       222      708      121
    #> [90m6[39m  [4m2[24m013     1     1       255       250      589      115
    #> [90m# ... with 3,300 more rows[39m
    ```

-   Find all groups bigger than a threshold:

    
    ```r
    popular_dests <- flights %>% 
      group_by(dest) %>% 
      filter(n() > 365)
    popular_dests
    #> [90m# A tibble: 332,577 x 19[39m
    #> [90m# Groups:   dest [77][39m
    #>    [1myear[22m [1mmonth[22m   [1mday[22m [1mdep_time[22m [1msched_dep_time[22m [1mdep_delay[22m [1marr_time[22m [1msched_arr_time[22m
    #>   [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m    [3m[90m<int>[39m[23m          [3m[90m<int>[39m[23m     [3m[90m<dbl>[39m[23m    [3m[90m<int>[39m[23m          [3m[90m<int>[39m[23m
    #> [90m1[39m  [4m2[24m013     1     1      517            515         2      830            819
    #> [90m2[39m  [4m2[24m013     1     1      533            529         4      850            830
    #> [90m3[39m  [4m2[24m013     1     1      542            540         2      923            850
    #> [90m4[39m  [4m2[24m013     1     1      544            545        -[31m1[39m     [4m1[24m004           [4m1[24m022
    #> [90m5[39m  [4m2[24m013     1     1      554            600        -[31m6[39m      812            837
    #> [90m6[39m  [4m2[24m013     1     1      554            558        -[31m4[39m      740            728
    #> [90m# ... with 332,571 more rows, and 11 more variables: [1marr_delay[22m <dbl>,[39m
    #> [90m#   [1mcarrier[22m <chr>, [1mflight[22m <int>, [1mtailnum[22m <chr>, [1morigin[22m <chr>, [1mdest[22m <chr>,[39m
    #> [90m#   [1mair_time[22m <dbl>, [1mdistance[22m <dbl>, [1mhour[22m <dbl>, [1mminute[22m <dbl>, [1mtime_hour[22m <dttm>[39m
    ```

-   Standardise to compute per group metrics:

    
    ```r
    popular_dests %>% 
      filter(arr_delay > 0) %>% 
      mutate(prop_delay = arr_delay / sum(arr_delay)) %>% 
      select(year:day, dest, arr_delay, prop_delay)
    #> [90m# A tibble: 131,106 x 6[39m
    #> [90m# Groups:   dest [77][39m
    #>    [1myear[22m [1mmonth[22m   [1mday[22m [1mdest[22m  [1marr_delay[22m [1mprop_delay[22m
    #>   [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<chr>[39m[23m     [3m[90m<dbl>[39m[23m      [3m[90m<dbl>[39m[23m
    #> [90m1[39m  [4m2[24m013     1     1 IAH          11  0.000[4m1[24m[4m1[24m[4m1[24m 
    #> [90m2[39m  [4m2[24m013     1     1 IAH          20  0.000[4m2[24m[4m0[24m[4m1[24m 
    #> [90m3[39m  [4m2[24m013     1     1 MIA          33  0.000[4m2[24m[4m3[24m[4m5[24m 
    #> [90m4[39m  [4m2[24m013     1     1 ORD          12  0.000[4m0[24m[4m4[24m[4m2[24m4
    #> [90m5[39m  [4m2[24m013     1     1 FLL          19  0.000[4m0[24m[4m9[24m[4m3[24m8
    #> [90m6[39m  [4m2[24m013     1     1 ORD           8  0.000[4m0[24m[4m2[24m[4m8[24m3
    #> [90m# ... with 131,100 more rows[39m
    ```

A grouped filter is a grouped mutate followed by an ungrouped filter.
I generally avoid them except for quick and dirty manipulations: otherwise it's hard to check that you've done the manipulation correctly.

Functions that work most naturally in grouped mutates and filters are known as window functions (vs. the summary functions used for summaries).
You can learn more about useful window functions in the corresponding vignette: `vignette("window-functions")`.

### Exercises

1.  Find the 10 most delayed flights using a ranking function.
    How do you want to handle ties?
    Carefully read the documentation for `min_rank()`.

2.  Which plane (`tailnum`) has the worst on-time record?

3.  What time of day should you fly if you want to avoid delays as much as possible?

4.  For each destination, compute the total minutes of delay.
    For each flight, compute the proportion of the total delay for its destination.

5.  Delays are typically temporally correlated: even once the problem that caused the initial delay has been resolved, later flights are delayed to allow earlier flights to leave.
    Using `lag()`, explore how the delay of a flight is related to the delay of the immediately preceding flight.

6.  Look at each destination.
    Can you find flights that are suspiciously fast?
    (i.e. flights that represent a potential data entry error).
    Compute the air time of a flight relative to the shortest flight to that destination.
    Which flights were most delayed in the air?

7.  Find all destinations that are flown by at least two carriers.
    Use that information to rank the carriers.

8.  
