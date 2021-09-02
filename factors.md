# Factors

## Introduction

In R, factors are used to work with categorical variables, variables that have a fixed and known set of possible values.
They are also useful when you want to display character vectors in a non-alphabetical order.

Historically, factors were much easier to work with than characters.
As a result, many of the functions in base R automatically convert characters to factors.
This means that factors often crop up in places where they're not actually helpful.
Fortunately, you don't need to worry about that in the tidyverse, and can focus on situations where factors are genuinely useful.

### Prerequisites

To work with factors, we'll use the **forcats** package, which is part of the core tidyverse.
It provides tools for dealing with **cat**egorical variables (and it's an anagram of factors!) using a wide range of helpers for working with factors.


```r
library(tidyverse)
```

### Learning more

If you want to learn more about factors, I recommend reading Amelia McNamara and Nicholas Horton's paper, [*Wrangling categorical data in R*](https://peerj.com/preprints/3163/).
This paper lays out some of the history discussed in [*stringsAsFactors: An unauthorized biography*](http://simplystatistics.org/2015/07/24/stringsasfactors-an-unauthorized-biography/) and [*stringsAsFactors = \<sigh\>*](http://notstatschat.tumblr.com/post/124987394001/stringsasfactors-sigh), and compares the tidy approaches to categorical data outlined in this book with base R methods.
An early version of the paper helped motivate and scope the forcats package; thanks Amelia & Nick!

## Creating factors

Imagine that you have a variable that records month:


```r
x1 <- c("Dec", "Apr", "Jan", "Mar")
```

Using a string to record this variable has two problems:

1.  There are only twelve possible months, and there's nothing saving you from typos:

    
    ```r
    x2 <- c("Dec", "Apr", "Jam", "Mar")
    ```

2.  It doesn't sort in a useful way:

    
    ```r
    sort(x1)
    #> [1] "Apr" "Dec" "Jan" "Mar"
    ```

You can fix both of these problems with a factor.
To create a factor you must start by creating a list of the valid **levels**:


```r
month_levels <- c(
  "Jan", "Feb", "Mar", "Apr", "May", "Jun", 
  "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"
)
```

Now you can create a factor:


```r
y1 <- factor(x1, levels = month_levels)
y1
#> [1] Dec Apr Jan Mar
#> Levels: Jan Feb Mar Apr May Jun Jul Aug Sep Oct Nov Dec
sort(y1)
#> [1] Jan Mar Apr Dec
#> Levels: Jan Feb Mar Apr May Jun Jul Aug Sep Oct Nov Dec
```

And any values not in the set will be silently converted to NA:


```r
y2 <- factor(x2, levels = month_levels)
y2
#> [1] Dec  Apr  <NA> Mar 
#> Levels: Jan Feb Mar Apr May Jun Jul Aug Sep Oct Nov Dec
```

If you want a warning, you can use `readr::parse_factor()`:


```r
y2 <- parse_factor(x2, levels = month_levels)
#> Warning: 1 parsing failure.
#> row col           expected actual
#>   3  -- value in level set    Jam
```

If you omit the levels, they'll be taken from the data in alphabetical order:


```r
factor(x1)
#> [1] Dec Apr Jan Mar
#> Levels: Apr Dec Jan Mar
```

Sometimes you'd prefer that the order of the levels match the order of the first appearance in the data.
You can do that when creating the factor by setting levels to `unique(x)`, or after the fact, with `fct_inorder()`:


```r
f1 <- factor(x1, levels = unique(x1))
f1
#> [1] Dec Apr Jan Mar
#> Levels: Dec Apr Jan Mar

f2 <- x1 %>% factor() %>% fct_inorder()
f2
#> [1] Dec Apr Jan Mar
#> Levels: Dec Apr Jan Mar
```

If you ever need to access the set of valid levels directly, you can do so with `levels()`:


```r
levels(f2)
#> [1] "Dec" "Apr" "Jan" "Mar"
```

## General Social Survey

For the rest of this chapter, we're going to focus on `forcats::gss_cat`.
It's a sample of data from the [General Social Survey](http://gss.norc.org), which is a long-running US survey conducted by the independent research organization NORC at the University of Chicago.
The survey has thousands of questions, so in `gss_cat` I've selected a handful that will illustrate some common challenges you'll encounter when working with factors.


```r
gss_cat
#> [90m# A tibble: 21,483 x 9[39m
#>    [1myear[22m [1mmarital[22m         [1mage[22m [1mrace[22m  [1mrincome[22m        [1mpartyid[22m   [1mrelig[22m  [1mdenom[22m  [1mtvhours[22m
#>   [3m[90m<int>[39m[23m [3m[90m<fct>[39m[23m         [3m[90m<int>[39m[23m [3m[90m<fct>[39m[23m [3m[90m<fct>[39m[23m          [3m[90m<fct>[39m[23m     [3m[90m<fct>[39m[23m  [3m[90m<fct>[39m[23m    [3m[90m<int>[39m[23m
#> [90m1[39m  [4m2[24m000 Never married    26 White $8000 to 9999  Ind,near~ Prote~ South~      12
#> [90m2[39m  [4m2[24m000 Divorced         48 White $8000 to 9999  Not str ~ Prote~ Bapti~      [31mNA[39m
#> [90m3[39m  [4m2[24m000 Widowed          67 White Not applicable Independ~ Prote~ No de~       2
#> [90m4[39m  [4m2[24m000 Never married    39 White Not applicable Ind,near~ Ortho~ Not a~       4
#> [90m5[39m  [4m2[24m000 Divorced         25 White Not applicable Not str ~ None   Not a~       1
#> [90m6[39m  [4m2[24m000 Married          25 White $20000 - 24999 Strong d~ Prote~ South~      [31mNA[39m
#> [90m# ... with 21,477 more rows[39m
```

(Remember, since this dataset is provided by a package, you can get more information about the variables with `?gss_cat`.)

When factors are stored in a tibble, you can't see their levels so easily.
One way to see them is with `count()`:


```r
gss_cat %>%
  count(race)
#> [90m# A tibble: 3 x 2[39m
#>   [1mrace[22m      [1mn[22m
#>   [3m[90m<fct>[39m[23m [3m[90m<int>[39m[23m
#> [90m1[39m Other  [4m1[24m959
#> [90m2[39m Black  [4m3[24m129
#> [90m3[39m White [4m1[24m[4m6[24m395
```

Or with a bar chart:


```r
ggplot(gss_cat, aes(race)) +
  geom_bar()
```

<img src="factors_files/figure-html/unnamed-chunk-13-1.png" width="70%" style="display: block; margin: auto;" />

By default, ggplot2 will drop levels that don't have any values.
You can force them to display with:


```r
ggplot(gss_cat, aes(race)) +
  geom_bar() +
  scale_x_discrete(drop = FALSE)
```

<img src="factors_files/figure-html/unnamed-chunk-14-1.png" width="70%" style="display: block; margin: auto;" />

These levels represent valid values that simply did not occur in this dataset.
In dplyr::count() set the `.drop` option to `FALSE`, to show these.


```r
gss_cat %>% 
  count(race, 
        .drop = FALSE)
#> [90m# A tibble: 4 x 2[39m
#>   [1mrace[22m               [1mn[22m
#>   [3m[90m<fct>[39m[23m          [3m[90m<int>[39m[23m
#> [90m1[39m Other           [4m1[24m959
#> [90m2[39m Black           [4m3[24m129
#> [90m3[39m White          [4m1[24m[4m6[24m395
#> [90m4[39m Not applicable     0
```

When working with factors, the two most common operations are changing the order of the levels, and changing the values of the levels.
Those operations are described in the sections below.

### Exercise

1.  Explore the distribution of `rincome` (reported income).
    What makes the default bar chart hard to understand?
    How could you improve the plot?

2.  What is the most common `relig` in this survey?
    What's the most common `partyid`?

3.  Which `relig` does `denom` (denomination) apply to?
    How can you find out with a table?
    How can you find out with a visualisation?

## Modifying factor order

It's often useful to change the order of the factor levels in a visualisation.
For example, imagine you want to explore the average number of hours spent watching TV per day across religions:


```r
relig_summary <- gss_cat %>%
  group_by(relig) %>%
  summarise(
    age = mean(age, na.rm = TRUE),
    tvhours = mean(tvhours, na.rm = TRUE),
    n = n()
  )

ggplot(relig_summary, aes(tvhours, relig)) + geom_point()
```

<img src="factors_files/figure-html/unnamed-chunk-16-1.png" width="70%" style="display: block; margin: auto;" />

It is difficult to interpret this plot because there's no overall pattern.
We can improve it by reordering the levels of `relig` using `fct_reorder()`.
`fct_reorder()` takes three arguments:

-   `f`, the factor whose levels you want to modify.
-   `x`, a numeric vector that you want to use to reorder the levels.
-   Optionally, `fun`, a function that's used if there are multiple values of `x` for each value of `f`. The default value is `median`.


```r
ggplot(relig_summary, aes(tvhours, fct_reorder(relig, tvhours))) +
  geom_point()
```

<img src="factors_files/figure-html/unnamed-chunk-17-1.png" width="70%" style="display: block; margin: auto;" />

Reordering religion makes it much easier to see that people in the "Don't know" category watch much more TV, and Hinduism & Other Eastern religions watch much less.

As you start making more complicated transformations, I'd recommend moving them out of `aes()` and into a separate `mutate()` step.
For example, you could rewrite the plot above as:


```r
relig_summary %>%
  mutate(relig = fct_reorder(relig, tvhours)) %>%
  ggplot(aes(tvhours, relig)) +
    geom_point()
```

What if we create a similar plot looking at how average age varies across reported income level?


```r
rincome_summary <- gss_cat %>%
  group_by(rincome) %>%
  summarise(
    age = mean(age, na.rm = TRUE),
    tvhours = mean(tvhours, na.rm = TRUE),
    n = n()
  )

ggplot(rincome_summary, aes(age, fct_reorder(rincome, age))) + geom_point()
```

<img src="factors_files/figure-html/unnamed-chunk-19-1.png" width="70%" style="display: block; margin: auto;" />

Here, arbitrarily reordering the levels isn't a good idea!
That's because `rincome` already has a principled order that we shouldn't mess with.
Reserve `fct_reorder()` for factors whose levels are arbitrarily ordered.

However, it does make sense to pull "Not applicable" to the front with the other special levels.
You can use `fct_relevel()`.
It takes a factor, `f`, and then any number of levels that you want to move to the front of the line.


```r
ggplot(rincome_summary, aes(age, fct_relevel(rincome, "Not applicable"))) +
  geom_point()
```

<img src="factors_files/figure-html/unnamed-chunk-20-1.png" width="70%" style="display: block; margin: auto;" />

Why do you think the average age for "Not applicable" is so high?

Another type of reordering is useful when you are colouring the lines on a plot.
`fct_reorder2()` reorders the factor by the `y` values associated with the largest `x` values.
This makes the plot easier to read because the line colours line up with the legend.


```r
by_age <- gss_cat %>%
  filter(!is.na(age)) %>%
  count(age, marital) %>%
  group_by(age) %>%
  mutate(prop = n / sum(n))

ggplot(by_age, aes(age, prop, colour = marital)) +
  geom_line(na.rm = TRUE)

ggplot(by_age, aes(age, prop, colour = fct_reorder2(marital, age, prop))) +
  geom_line() +
  labs(colour = "marital")
```

<img src="factors_files/figure-html/unnamed-chunk-21-1.png" width="50%" /><img src="factors_files/figure-html/unnamed-chunk-21-2.png" width="50%" />

Finally, for bar plots, you can use `fct_infreq()` to order levels in increasing frequency: this is the simplest type of reordering because it doesn't need any extra variables.
You may want to combine with `fct_rev()`.


```r
gss_cat %>%
  mutate(marital = marital %>% fct_infreq() %>% fct_rev()) %>%
  ggplot(aes(marital)) +
    geom_bar()
```

<img src="factors_files/figure-html/unnamed-chunk-22-1.png" width="70%" style="display: block; margin: auto;" />

### Exercises

1.  There are some suspiciously high numbers in `tvhours`.
    Is the mean a good summary?

2.  For each factor in `gss_cat` identify whether the order of the levels is arbitrary or principled.

3.  Why did moving "Not applicable" to the front of the levels move it to the bottom of the plot?

## Modifying factor levels

More powerful than changing the orders of the levels is changing their values.
This allows you to clarify labels for publication, and collapse levels for high-level displays.
The most general and powerful tool is `fct_recode()`.
It allows you to recode, or change, the value of each level.
For example, take the `gss_cat$partyid`:


```r
gss_cat %>% count(partyid)
#> [90m# A tibble: 10 x 2[39m
#>   [1mpartyid[22m                [1mn[22m
#>   [3m[90m<fct>[39m[23m              [3m[90m<int>[39m[23m
#> [90m1[39m No answer            154
#> [90m2[39m Don't know             1
#> [90m3[39m Other party          393
#> [90m4[39m Strong republican   [4m2[24m314
#> [90m5[39m Not str republican  [4m3[24m032
#> [90m6[39m Ind,near rep        [4m1[24m791
#> [90m# ... with 4 more rows[39m
```

The levels are terse and inconsistent.
Let's tweak them to be longer and use a parallel construction.


```r
gss_cat %>%
  mutate(partyid = fct_recode(partyid,
    "Republican, strong"    = "Strong republican",
    "Republican, weak"      = "Not str republican",
    "Independent, near rep" = "Ind,near rep",
    "Independent, near dem" = "Ind,near dem",
    "Democrat, weak"        = "Not str democrat",
    "Democrat, strong"      = "Strong democrat"
  )) %>%
  count(partyid)
#> [90m# A tibble: 10 x 2[39m
#>   [1mpartyid[22m                   [1mn[22m
#>   [3m[90m<fct>[39m[23m                 [3m[90m<int>[39m[23m
#> [90m1[39m No answer               154
#> [90m2[39m Don't know                1
#> [90m3[39m Other party             393
#> [90m4[39m Republican, strong     [4m2[24m314
#> [90m5[39m Republican, weak       [4m3[24m032
#> [90m6[39m Independent, near rep  [4m1[24m791
#> [90m# ... with 4 more rows[39m
```

`fct_recode()` will leave levels that aren't explicitly mentioned as is, and will warn you if you accidentally refer to a level that doesn't exist.

To combine groups, you can assign multiple old levels to the same new level:


```r
gss_cat %>%
  mutate(partyid = fct_recode(partyid,
    "Republican, strong"    = "Strong republican",
    "Republican, weak"      = "Not str republican",
    "Independent, near rep" = "Ind,near rep",
    "Independent, near dem" = "Ind,near dem",
    "Democrat, weak"        = "Not str democrat",
    "Democrat, strong"      = "Strong democrat",
    "Other"                 = "No answer",
    "Other"                 = "Don't know",
    "Other"                 = "Other party"
  )) %>%
  count(partyid)
#> [90m# A tibble: 8 x 2[39m
#>   [1mpartyid[22m                   [1mn[22m
#>   [3m[90m<fct>[39m[23m                 [3m[90m<int>[39m[23m
#> [90m1[39m Other                   548
#> [90m2[39m Republican, strong     [4m2[24m314
#> [90m3[39m Republican, weak       [4m3[24m032
#> [90m4[39m Independent, near rep  [4m1[24m791
#> [90m5[39m Independent            [4m4[24m119
#> [90m6[39m Independent, near dem  [4m2[24m499
#> [90m# ... with 2 more rows[39m
```

You must use this technique with care: if you group together categories that are truly different you will end up with misleading results.

If you want to collapse a lot of levels, `fct_collapse()` is a useful variant of `fct_recode()`.
For each new variable, you can provide a vector of old levels:


```r
gss_cat %>%
  mutate(partyid = fct_collapse(partyid,
    other = c("No answer", "Don't know", "Other party"),
    rep = c("Strong republican", "Not str republican"),
    ind = c("Ind,near rep", "Independent", "Ind,near dem"),
    dem = c("Not str democrat", "Strong democrat")
  )) %>%
  count(partyid)
#> [90m# A tibble: 4 x 2[39m
#>   [1mpartyid[22m     [1mn[22m
#>   [3m[90m<fct>[39m[23m   [3m[90m<int>[39m[23m
#> [90m1[39m other     548
#> [90m2[39m rep      [4m5[24m346
#> [90m3[39m ind      [4m8[24m409
#> [90m4[39m dem      [4m7[24m180
```

Sometimes you just want to lump together all the small groups to make a plot or table simpler.
That's the job of the `fct_lump_*()` family of functions.
`fct_lump_lowfreq()` is a simple starting point that progressively lumps the smallest groups categories into "Other", always keeping "Other" as the smallest category.


```r
gss_cat %>%
  mutate(relig = fct_lump_lowfreq(relig)) %>%
  count(relig)
#> [90m# A tibble: 2 x 2[39m
#>   [1mrelig[22m          [1mn[22m
#>   [3m[90m<fct>[39m[23m      [3m[90m<int>[39m[23m
#> [90m1[39m Protestant [4m1[24m[4m0[24m846
#> [90m2[39m Other      [4m1[24m[4m0[24m637
```

In this case it's not very helpful: it is true that the majority of Americans in this survey are Protestant, but we'd probably like to see some more details!
Instead, we can use the `fct_lump_n()` to specify that we want exactly 10 groups:


```r
gss_cat %>%
  mutate(relig = fct_lump_n(relig, n = 10)) %>%
  count(relig, sort = TRUE) %>%
  print(n = Inf)
#> [90m# A tibble: 10 x 2[39m
#>    [1mrelig[22m                       [1mn[22m
#>    [3m[90m<fct>[39m[23m                   [3m[90m<int>[39m[23m
#> [90m 1[39m Protestant              [4m1[24m[4m0[24m846
#> [90m 2[39m Catholic                 [4m5[24m124
#> [90m 3[39m None                     [4m3[24m523
#> [90m 4[39m Christian                 689
#> [90m 5[39m Other                     458
#> [90m 6[39m Jewish                    388
#> [90m 7[39m Buddhism                  147
#> [90m 8[39m Inter-nondenominational   109
#> [90m 9[39m Moslem/islam              104
#> [90m10[39m Orthodox-christian         95
```

### Exercises

1.  How have the proportions of people identifying as Democrat, Republican, and Independent changed over time?

2.  How could you collapse `rincome` into a small set of categories?

3.  Notice there are 9 groups (excluding other) in the `fct_lump` example above.
    Why not 10?
    (Hint: type `?fct_lump`, and find the default for the argument `other_level` is "Other".)
