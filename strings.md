# Strings

::: {.rmdnote}
You are reading the work-in-progress second edition of R for Data Science. This chapter is currently undergoing heavy restructuring and may be confusing or incomplete. You can find the polished first edition at <https://r4ds.had.co.nz>.
:::

## Introduction

This chapter introduces you to strings.
You'll learn the basics of how strings work in R and how to create them "by hand".
Big topic so spread over three chapters: here we'll focus on the basic mechanics, in Chapter \@ref(regular-expressions) we'll dive into the details of regular expressions the sometimes cryptic language for describing patterns in strings, and we'll return to strings later in Chapter \@ref(programming-with-strings) when we think about them about from a programming perspective (rather than a data analysis perspective).
We'll finish up with a discussion of some of the new challenges that arise when working with non-English strings.

While base R contains functions that allow us to perform pretty much all of the operations described in this chapter, here we're going to use the **stringr** package.
stringr has been carefully designed to be as consistent as possible so that knowledge gained about one function can be more easily transferred to the next.
stringr functions all start with the same `str_` prefix.
This is particularly useful if you use RStudio, because typing `str_` will trigger autocomplete, allowing you to see all stringr's functions:

<img src="screenshots/stringr-autocomplete.png" width="70%" style="display: block; margin: auto;" />

### Prerequisites

This chapter will focus on the **stringr** package for string manipulation, which is part of the core tidyverse.
We'll also work with the babynames dataset.


```r
library(tidyverse)
library(babynames)
```

## Creating a string

To begin, let's discuss the mechanics of creating a string.
We've created strings in passing earlier in the book, but didn't discuss the details.
First, there are two basic ways to create a string: using either single quotes (`'`) or double quotes (`"`).
Unlike other languages, there is no difference in behaviour.
I recommend always using `"`, unless you want to create a string that contains multiple `"`.


```r
string1 <- "This is a string"
string2 <- 'If I want to include a "quote" inside a string, I use single quotes'
```

If you forget to close a quote, you'll see `+`, the continuation character:

    > "This is a string without a closing quote
    + 
    + 
    + HELP I'M STUCK

If this happen to you, press Escape and try again.

### Escapes

To include a literal single or double quote in a string you can use `\` to "escape" it:


```r
double_quote <- "\"" # or '"'
single_quote <- '\'' # or "'"
```

Which means if you want to include a literal backslash, you'll need to double it up: `"\\"`:


```r
backslash <- "\\"
```

Beware that the printed representation of a string is not the same as string itself, because the printed representation shows the escapes.
To see the raw contents of the string, use `str_view()`:


```r
x <- c(single_quote, double_quote, backslash)
x
#> [1] "'"  "\"" "\\"
str_view(x)
#> '
#> "
#> \
```

### Raw strings

Creating a string with multiple quotes or backslashes gets confusing quickly.
For example, lets create a string that contains the contents of the chunk where I define the `double_quote` and `single_quote` variables:


```r
tricky <- "double_quote <- \"\\\"\" # or '\"'
single_quote <- '\\'' # or \"'\""
str_view(tricky)
#> double_quote <- "\"" # or '"'
#> single_quote <- '\'' # or "'"
```

You can instead use a **raw string**[^strings-1] to reduce the amount of escaping:

[^strings-1]: Available in R 4.0.0 and above.


```r
tricky <- r"(double_quote <- "\"" # or '"'
single_quote <- '\'' # or "'"
)"
str_view(tricky)
#> double_quote <- "\"" # or '"'
#> single_quote <- '\'' # or "'"
```

A raw string starts with `r"(` and finishes with `)"`.
If your string contains `)"` you can instead use `r"[]"` or `r"{}"`, and if that's still not enough, you can insert any number of dashes to make the opening and closing pairs unique, e.g. `` `r"--()--" ``, `` `r"---()---" ``,etc.

### Other special characters

As well as `\"`, `\'`, and `\\` there are a handful of other special characters that may come in handy. The most common are `"\n"`, newline, and `"\t"`, tab, but you can see the complete list in `?'"'`.

You'll also sometimes see strings containing Unicode escapes that start with `\u` or `\U`.
This is a way of writing non-English characters that works on all systems:


```r
x <- c("\u00b5", "\U0001f604")
x
#> [1] "μ"  "<U+0001F604>"
str_view(x)
#> μ
#> <U+0001F604>
```

## Combining strings

Use `str_c()`[^strings-2] to join together multiple character vectors into a single vector:

[^strings-2]: `str_c()` is very similar to the base `paste0()`.
    There are two main reasons I use it here: it obeys the usual rules for handling `NA`, and it uses the tidyverse recycling rules.


```r
str_c("x", "y")
#> [1] "xy"
str_c("x", "y", "z")
#> [1] "xyz"
```

`str_c()` obeys the usual recycling rules:


```r
names <- c("Timothy", "Dewey", "Mable")
str_c("Hi ", names, "!")
#> [1] "Hi Timothy!" "Hi Dewey!"   "Hi Mable!"
```

And like most other functions in R, missing values are contagious.
You can use `coalesce()` to replace missing values with a value of your choosing:


```r
x <- c("abc", NA)
str_c("|-", x, "-|")
#> [1] "|-abc-|" NA
str_c("|-", coalesce(x, ""), "-|")
#> [1] "|-abc-|" "|--|"
```

Since `str_c()` creates a vector, you'll usually use it with a `mutate()`:


```r
starwars %>% 
  mutate(greeting = str_c("Hi! I'm ", name, "."), .after = name)
#> [90m# A tibble: 87 x 15[39m
#>   [1mname[22m   [1mgreeting[22m  [1mheight[22m  [1mmass[22m [1mhair_color[22m [1mskin_color[22m [1meye_color[22m [1mbirth_year[22m [1msex[22m  
#>   [3m[90m<chr>[39m[23m  [3m[90m<chr>[39m[23m      [3m[90m<int>[39m[23m [3m[90m<dbl>[39m[23m [3m[90m<chr>[39m[23m      [3m[90m<chr>[39m[23m      [3m[90m<chr>[39m[23m          [3m[90m<dbl>[39m[23m [3m[90m<chr>[39m[23m
#> [90m1[39m Luke ~ Hi! I'm ~    172    77 blond      fair       blue            19   male 
#> [90m2[39m C-3PO  Hi! I'm ~    167    75 [31mNA[39m         gold       yellow         112   none 
#> [90m3[39m R2-D2  Hi! I'm ~     96    32 [31mNA[39m         white, bl~ red             33   none 
#> [90m4[39m Darth~ Hi! I'm ~    202   136 none       white      yellow          41.9 male 
#> [90m5[39m Leia ~ Hi! I'm ~    150    49 brown      light      brown           19   fema~
#> [90m6[39m Owen ~ Hi! I'm ~    178   120 brown, gr~ light      blue            52   male 
#> [90m# ... with 81 more rows, and 6 more variables: [1mgender[22m <chr>, [1mhomeworld[22m <chr>,[39m
#> [90m#   [1mspecies[22m <chr>, [1mfilms[22m <list>, [1mvehicles[22m <list>, [1mstarships[22m <list>[39m
```

Another powerful way of combining strings is with the glue package.
You can either use `glue::glue()` directly or call it via the `str_glue()` wrapper that stringr provides for you.
Glue works a little differently to the other methods: you give it a single string then within the string use `{}` to indicate where existing variables should be evaluated:


```r
str_glue("|-{x}-|")
#> |-abc-|
#> |-NA-|
```

Like `str_c()`, `str_glue()` pairs well with `mutate()`:


```r
starwars %>% 
  mutate(
    intro = str_glue("Hi! My is {name} and I'm a {species} from {homeworld}"),
    .keep = "none"
  )
#> [90m# A tibble: 87 x 1[39m
#>   [1mintro[22m                                                 
#>   [3m[90m<glue>[39m[23m                                                
#> [90m1[39m Hi! My is Luke Skywalker and I'm a Human from Tatooine
#> [90m2[39m Hi! My is C-3PO and I'm a Droid from Tatooine         
#> [90m3[39m Hi! My is R2-D2 and I'm a Droid from Naboo            
#> [90m4[39m Hi! My is Darth Vader and I'm a Human from Tatooine   
#> [90m5[39m Hi! My is Leia Organa and I'm a Human from Alderaan   
#> [90m6[39m Hi! My is Owen Lars and I'm a Human from Tatooine     
#> [90m# ... with 81 more rows[39m
```

You can use any valid R code inside of `{}`, but it's a good idea to pull complex calculations out into their own variables so you can more easily check your work.

## Length and subsetting

It's natural to think about the letters that make up an individual string.
(But note that the idea of a "letter" isn't a natural fit to every language, we'll come back to that in Section \@ref(other-languages)).
For example, `str_length()` tells you the length of a string in characters:


```r
str_length(c("a", "R for data science", NA))
#> [1]  1 18 NA
```

You could use this with `count()` to find the distribution of lengths of US babynames, and then with `filter()` to look at the longest names:


```r
babynames %>%
  count(length = str_length(name), wt = n)
#> [90m# A tibble: 14 x 2[39m
#>   [1mlength[22m        [1mn[22m
#>    [3m[90m<int>[39m[23m    [3m[90m<int>[39m[23m
#> [90m1[39m      2   [4m3[24m[4m3[24m[4m8[24m150
#> [90m2[39m      3  8[4m5[24m[4m8[24m[4m9[24m596
#> [90m3[39m      4 48[4m5[24m[4m0[24m[4m6[24m739
#> [90m4[39m      5 87[4m0[24m[4m1[24m[4m1[24m607
#> [90m5[39m      6 90[4m7[24m[4m4[24m[4m9[24m404
#> [90m6[39m      7 72[4m1[24m[4m2[24m[4m0[24m767
#> [90m# ... with 8 more rows[39m

babynames %>% 
  filter(str_length(name) == 15) %>% 
  count(name, wt = n, sort = TRUE)
#> [90m# A tibble: 34 x 2[39m
#>   [1mname[22m                [1mn[22m
#>   [3m[90m<chr>[39m[23m           [3m[90m<int>[39m[23m
#> [90m1[39m Franciscojavier   123
#> [90m2[39m Christopherjohn   118
#> [90m3[39m Johnchristopher   118
#> [90m4[39m Christopherjame   108
#> [90m5[39m Christophermich    52
#> [90m6[39m Ryanchristopher    45
#> [90m# ... with 28 more rows[39m
```

You can extract parts of a string using `str_sub(string, start, end)`.
The `start` and `end` arguments are inclusive, so the length of the returned string will be `end - start + 1`:


```r
x <- c("Apple", "Banana", "Pear")
str_sub(x, 1, 3)
#> [1] "App" "Ban" "Pea"
```

You can use negative values to count back from the end of the string: -1 is the last character, -2 is the second to last character, etc.


```r
str_sub(x, -3, -1)
#> [1] "ple" "ana" "ear"
```

Note that `str_sub()` won't fail if the string is too short: it will just return as much as possible:


```r
str_sub("a", 1, 5)
#> [1] "a"
```

We could use `str_sub()` with `mutate()` to find the first and last letter of each name:


```r
babynames %>% 
  mutate(
    first = str_sub(name, 1, 1),
    last = str_sub(name, -1, -1)
  )
#> [90m# A tibble: 1,924,665 x 7[39m
#>    [1myear[22m [1msex[22m   [1mname[22m          [1mn[22m   [1mprop[22m [1mfirst[22m [1mlast[22m 
#>   [3m[90m<dbl>[39m[23m [3m[90m<chr>[39m[23m [3m[90m<chr>[39m[23m     [3m[90m<int>[39m[23m  [3m[90m<dbl>[39m[23m [3m[90m<chr>[39m[23m [3m[90m<chr>[39m[23m
#> [90m1[39m  [4m1[24m880 F     Mary       [4m7[24m065 0.072[4m4[24m M     y    
#> [90m2[39m  [4m1[24m880 F     Anna       [4m2[24m604 0.026[4m7[24m A     a    
#> [90m3[39m  [4m1[24m880 F     Emma       [4m2[24m003 0.020[4m5[24m E     a    
#> [90m4[39m  [4m1[24m880 F     Elizabeth  [4m1[24m939 0.019[4m9[24m E     h    
#> [90m5[39m  [4m1[24m880 F     Minnie     [4m1[24m746 0.017[4m9[24m M     e    
#> [90m6[39m  [4m1[24m880 F     Margaret   [4m1[24m578 0.016[4m2[24m M     t    
#> [90m# ... with 1,924,659 more rows[39m
```

Sometimes you'll get a column that's made up of individual fixed length strings that have been joined together:


```r
df <- tribble(
  ~ sex_year_age,
  "M200115",
  "F201503",
)
```

You can extract the columns using `str_sub()`:


```r
df %>% mutate(
  sex = str_sub(sex_year_age, 1, 1),
  year = str_sub(sex_year_age, 2, 5),
  age = str_sub(sex_year_age, 6, 7),
)
#> [90m# A tibble: 2 x 4[39m
#>   [1msex_year_age[22m [1msex[22m   [1myear[22m  [1mage[22m  
#>   [3m[90m<chr>[39m[23m        [3m[90m<chr>[39m[23m [3m[90m<chr>[39m[23m [3m[90m<chr>[39m[23m
#> [90m1[39m M200115      M     2001  15   
#> [90m2[39m F201503      F     2015  03
```

Or use the `separate()` helper function:


```r
df %>% 
  separate(sex_year_age, c("sex", "year", "age"), c(1, 5))
#> [90m# A tibble: 2 x 3[39m
#>   [1msex[22m   [1myear[22m  [1mage[22m  
#>   [3m[90m<chr>[39m[23m [3m[90m<chr>[39m[23m [3m[90m<chr>[39m[23m
#> [90m1[39m M     2001  15   
#> [90m2[39m F     2015  03
```

Note that you give `separate()` three columns but only two positions --- that's because you're telling `separate()` where to break up the string.

TODO: draw diagram to emphasise that it's the space between the characters.

Later on, we'll come back two related problems: the components have varying length and are a separated by a character, or they have an varying number of components and you want to split up into rows, rather than columns.

### Exercises

1.  Use `str_length()` and `str_sub()` to extract the middle letter from each baby name. What will you do if the string has an even number of characters?

## Long strings

Sometimes the reason you care about the length of a string is because you're trying to fit it into a label.
stringr provides two useful tools for cases where your string is too long:

-   `str_trunc(x, 20)` ensures that no string is longer than 20 characters, replacing any thing too long with `…`.

-   `str_wrap(x, 20)` wraps a string introducing new lines so that each line is at most 20 characters (it doesn't hyphenate, however, so any word longer than 20 characters will make a longer time)


```r
x <- "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat."

str_trunc(x, 30)
#> [1] "Lorem ipsum dolor sit amet,..."
str_view(str_wrap(x, 30))
#> Lorem ipsum dolor sit amet,
#> consectetur adipiscing
#> elit, sed do eiusmod tempor
#> incididunt ut labore et dolore
#> magna aliqua. Ut enim ad
#> minim veniam, quis nostrud
#> exercitation ullamco laboris
#> nisi ut aliquip ex ea commodo
#> consequat.
```

## String summaries

`str_c()` combines multiple character vectors into a single character vector; the output is the same length as the input.
An related function is `str_flatten()`: it takes a character vector and returns a single string:


```r
str_flatten(c("x", "y", "z"))
#> [1] "xyz"
str_flatten(c("x", "y", "z"), ", ")
#> [1] "x, y, z"
str_flatten(c("x", "y", "z"), ", ", ", and ")
#> [1] "x, y, and z"
```

Just like `sum()` and `mean()` take a vector of numbers and return a single number, `str_flatten()` takes a character vector and returns a single string.
This makes `str_flatten()` a summary function for strings, so you'll often pair it with `summarise()`:


```r
df <- tribble(
  ~ name, ~ fruit,
  "Carmen", "banana",
  "Carmen", "apple",
  "Marvin", "nectarine",
  "Terence", "cantaloupe",
  "Terence", "papaya",
  "Terence", "madarine"
)
df %>%
  group_by(name) %>% 
  summarise(fruits = str_flatten(fruit, ", "))
#> [90m# A tibble: 3 x 2[39m
#>   [1mname[22m    [1mfruits[22m                      
#>   [3m[90m<chr>[39m[23m   [3m[90m<chr>[39m[23m                       
#> [90m1[39m Carmen  banana, apple               
#> [90m2[39m Marvin  nectarine                   
#> [90m3[39m Terence cantaloupe, papaya, madarine
```

## Detect matches

To determine if a character vector matches a pattern, use `str_detect()`.
It returns a logical vector the same length as the input:


```r
x <- c("apple", "banana", "pear")
str_detect(x, "e")
#> [1]  TRUE FALSE  TRUE
```

This makes it a logical pairing with `filter()`.
The following example returns all names that contain a lower-case "x":


```r
babynames %>% filter(str_detect(name, "x"))
#> [90m# A tibble: 16,317 x 5[39m
#>    [1myear[22m [1msex[22m   [1mname[22m          [1mn[22m      [1mprop[22m
#>   [3m[90m<dbl>[39m[23m [3m[90m<chr>[39m[23m [3m[90m<chr>[39m[23m     [3m[90m<int>[39m[23m     [3m[90m<dbl>[39m[23m
#> [90m1[39m  [4m1[24m880 F     Roxie        62 0.000[4m6[24m[4m3[24m[4m5[24m 
#> [90m2[39m  [4m1[24m880 F     Dixie        15 0.000[4m1[24m[4m5[24m[4m4[24m 
#> [90m3[39m  [4m1[24m880 F     Roxanna       9 0.000[4m0[24m[4m9[24m[4m2[24m2
#> [90m4[39m  [4m1[24m880 F     Texas         5 0.000[4m0[24m[4m5[24m[4m1[24m2
#> [90m5[39m  [4m1[24m880 M     Alexander   211 0.001[4m7[24m[4m8[24m  
#> [90m6[39m  [4m1[24m880 M     Alex        147 0.001[4m2[24m[4m4[24m  
#> [90m# ... with 16,311 more rows[39m
```

Remember that when you use a logical vector in a numeric context, `FALSE` becomes 0 and `TRUE` becomes 1.
That means you can use `summarise()` with `sum()` or `mean()` and `str_detect()` if you want to answer questions about the prevalence of patterns.
For example, the following snippet, gives the proportion of names containing an "x" by year:


```r
babynames %>% 
  group_by(year) %>% 
  summarise(prop_x = mean(str_detect(name, "x"))) %>% 
  ggplot(aes(year, prop_x)) + 
  geom_line()
```

<img src="strings_files/figure-html/unnamed-chunk-30-1.png" title="A timeseries showing the proportion of baby names that contain the letter x. The proportion declines gradually from 8 per 1000 in 1880 to 4 per 1000 in 1980, then increases rapidly to 16 per 1000 in 2019." alt="A timeseries showing the proportion of baby names that contain the letter x. The proportion declines gradually from 8 per 1000 in 1880 to 4 per 1000 in 1980, then increases rapidly to 16 per 1000 in 2019." width="70%" style="display: block; margin: auto;" />

(Note that this gives us the proportion of names that contain an x; if you wanted the proportion of babies given a name containing an x, you'd need to perform a weighted mean).

A variation on `str_detect()` is `str_count()`: rather than a simple yes or no, it tells you how many matches there are in a string:


```r
str_count(x, "p")
#> [1] 2 0 1
```

It's natural to use `str_count()` with `mutate()`:


```r
babynames %>% 
  distinct(name) %>% 
  mutate(
    vowels = str_count(name, "[aeiou]"),
    consonants = str_count(name, "[^aeiou]")
  )
#> [90m# A tibble: 97,310 x 3[39m
#>   [1mname[22m      [1mvowels[22m [1mconsonants[22m
#>   [3m[90m<chr>[39m[23m      [3m[90m<int>[39m[23m      [3m[90m<int>[39m[23m
#> [90m1[39m Mary           1          3
#> [90m2[39m Anna           1          3
#> [90m3[39m Emma           1          3
#> [90m4[39m Elizabeth      3          6
#> [90m5[39m Minnie         3          3
#> [90m6[39m Margaret       3          5
#> [90m# ... with 97,304 more rows[39m
```

### Exercises

1.  What word has the highest number of vowels? What word has the highest proportion of vowels? (Hint: what is the denominator?)

## Introduction to regular expressions

Before we can continue on we need to discuss the second argument to `str_detect()` --- the pattern that you want to match.
Above, I used a simple string, but the pattern actually a much richer tool called a **regular expression**.
A regular expression uses special characters to match string patterns.
For example, `.` will match any character, so `"a."` will match any string that contains an a followed by another character:


```r
str_detect(c("a", "ab", "ae", "bd", "ea", "eab"), "a.")
#> [1] FALSE  TRUE  TRUE FALSE FALSE  TRUE
```

`str_view()` shows you regular expressions to help understand what's happening:


```r
str_view(c("a", "ab", "ae", "bd", "ea", "eab"), "a.")
#> a
#> [36m<ab>[39m
#> [36m<ae>[39m
#> bd
#> ea
#> e[36m<ab>[39m
```

Regular expressions are a powerful and flexible language which we'll come back to in Chapter \@ref(regular-expressions).
Here we'll use only the most important components of the syntax as you learn the other stringr tools for working with patterns.

There are three useful **quantifiers** that can be applied to other pattern: `?` makes a pattern option (i.e. it matches 0 or 1 times), `+` lets a pattern repeat (ie. it matches at least once), and `*` lets a pattern be optional or repeat (i.e. it matches any number of times, including 0).

-   `ab?` match an "a", optionally followed by a b

-   `ab+` matches an "a", followed by at least one b

-   `ab*` matches an "a", followed by any number of bs

You can use `()` to control precedence:

-   `(ab)?` optionally matches "ab"

-   `(ab)+` matches one or more "ab" repeats


```r
str_view(c("aba", "ababab", "abbbbbb"), "ab+")
#> [36m<ab>[39ma
#> [36m<ab>[39mabab
#> [36m<abbbbbb>[39m
str_view(c("aba", "ababab", "abbbbbb"), "(ab)+")
#> [36m<ab>[39ma
#> [36m<ababab>[39m
#> [36m<ab>[39mbbbbb
```

There are various alternatives to `.` that match a restricted set of characters.
One useful operator is the **character class:** `[abcd]` match "a", "b", "c", or "d"; `[^abcd]` matches anything **except** "a", "b", "c", or "d".

You can opt-out of the regular expression rules by using `fixed`:


```r
str_view(c("", "a", "."), fixed("."))
#> 
#> a
#> [36m<.>[39m
```

Note that both fixed strings and regular expressions are case sensitive by default.
You can opt out by setting `ignore_case = TRUE`.


```r
str_view_all("x  X  xy", "X")
#> x  [36m<X>[39m  xy
str_view_all("x  X  xy", fixed("X", ignore_case = TRUE))
#> [36m<x>[39m  [36m<X>[39m  [36m<x>[39my
str_view_all("x  X  xy", regex(".Y", ignore_case = TRUE))
#> x  X  [36m<xy>[39m
```

We'll come back to case later, because it's not trivial for many languages.

### Exercises

1.  For each of the following challenges, try solving it by using both a single regular expression, and a combination of multiple `str_detect()` calls.

    a.  Find all words that start or end with `x`.
    b.  Find all words that start with a vowel and end with a consonant.
    c.  Are there any words that contain at least one of each different vowel?

## Replacing matches

`str_replace_all()` allow you to replace matches with new strings.
The simplest use is to replace a pattern with a fixed string:


```r
x <- c("apple", "pear", "banana")
str_replace_all(x, "[aeiou]", "-")
#> [1] "-ppl-"  "p--r"   "b-n-n-"
```

With `str_replace_all()` you can perform multiple replacements by supplying a named vector.
The name gives a regular expression to match, and the value gives the replacement.


```r
x <- c("1 house", "1 person has 2 cars", "3 people")
str_replace_all(x, c("1" = "one", "2" = "two", "3" = "three"))
#> [1] "one house"               "one person has two cars"
#> [3] "three people"
```

`str_remove_all()` is a short cut for `str_replace_all(x, pattern, "")` --- it removes matching patterns from a string.

Use in `mutate()`

Using pipe inside mutate.
Recommendation to make a function, and think about testing it --- don't need formal tests, but useful to build up a set of positive and negative test cases as you.

#### Exercises

1.  Replace all forward slashes in a string with backslashes.

2.  Implement a simple version of `str_to_lower()` using `str_replace_all()`.

3.  Switch the first and last letters in `words`.
    Which of those strings are still words?

## Extract full matches

If your data is in a tibble, it's often easier to use `tidyr::extract()`.
It works like `str_match()` but requires you to name the matches, which are then placed in new columns:


```r
tibble(sentence = sentences) %>% 
  tidyr::extract(
    sentence, c("article", "noun"), "(a|the) ([^ ]+)", 
    remove = FALSE
  )
#> [90m# A tibble: 720 x 3[39m
#>   [1msentence[22m                                    [1marticle[22m [1mnoun[22m   
#>   [3m[90m<chr>[39m[23m                                       [3m[90m<chr>[39m[23m   [3m[90m<chr>[39m[23m  
#> [90m1[39m The birch canoe slid on the smooth planks.  the     smooth 
#> [90m2[39m Glue the sheet to the dark blue background. the     sheet  
#> [90m3[39m It's easy to tell the depth of a well.      the     depth  
#> [90m4[39m These days a chicken leg is a rare dish.    a       chicken
#> [90m5[39m Rice is often served in round bowls.        [31mNA[39m      [31mNA[39m     
#> [90m6[39m The juice of lemons makes fine punch.       [31mNA[39m      [31mNA[39m     
#> [90m# ... with 714 more rows[39m
```

### Exercises

1.  In the previous example, you might have noticed that the regular expression matched "flickered", which is not a colour. Modify the regex to fix the problem.
2.  Find all words that come after a "number" like "one", "two", "three" etc. Pull out both the number and the word.
3.  Find all contractions. Separate out the pieces before and after the apostrophe.

## Strings -\> Columns

## Separate

`separate()` pulls apart one column into multiple columns, by splitting wherever a separator character appears.
Take `table3`:


```r
table3
#> [90m# A tibble: 6 x 3[39m
#>   [1mcountry[22m      [1myear[22m [1mrate[22m             
#> [90m*[39m [3m[90m<chr>[39m[23m       [3m[90m<int>[39m[23m [3m[90m<chr>[39m[23m            
#> [90m1[39m Afghanistan  [4m1[24m999 745/19987071     
#> [90m2[39m Afghanistan  [4m2[24m000 2666/20595360    
#> [90m3[39m Brazil       [4m1[24m999 37737/172006362  
#> [90m4[39m Brazil       [4m2[24m000 80488/174504898  
#> [90m5[39m China        [4m1[24m999 212258/1272915272
#> [90m6[39m China        [4m2[24m000 213766/1280428583
```

The `rate` column contains both `cases` and `population` variables, and we need to split it into two variables.
`separate()` takes the name of the column to separate, and the names of the columns to separate into, as shown in Figure \@ref(fig:tidy-separate) and the code below.


```r
table3 %>%
  separate(rate, into = c("cases", "population"))
#> [90m# A tibble: 6 x 4[39m
#>   [1mcountry[22m      [1myear[22m [1mcases[22m  [1mpopulation[22m
#>   [3m[90m<chr>[39m[23m       [3m[90m<int>[39m[23m [3m[90m<chr>[39m[23m  [3m[90m<chr>[39m[23m     
#> [90m1[39m Afghanistan  [4m1[24m999 745    19987071  
#> [90m2[39m Afghanistan  [4m2[24m000 2666   20595360  
#> [90m3[39m Brazil       [4m1[24m999 37737  172006362 
#> [90m4[39m Brazil       [4m2[24m000 80488  174504898 
#> [90m5[39m China        [4m1[24m999 212258 1272915272
#> [90m6[39m China        [4m2[24m000 213766 1280428583
```

<div class="figure" style="text-align: center">
<img src="images/tidy-17.png" alt="Two panels, one with a data frame with three columns (country, year, and rate) and the other with a data frame with four columns (country, year, cases, and population). Arrows show how the rate variable is separated into two variables: cases and population." width="75%" />
<p class="caption">(\#fig:tidy-separate)Separating `rate` into `cases` and `population` to make `table3` tidy</p>
</div>

By default, `separate()` will split values wherever it sees a non-alphanumeric character (i.e. a character that isn't a number or letter).
For example, in the code above, `separate()` split the values of `rate` at the forward slash characters.
If you wish to use a specific character to separate a column, you can pass the character to the `sep` argument of `separate()`.
For example, we could rewrite the code above as:


```r
table3 %>%
  separate(rate, into = c("cases", "population"), sep = "/")
```

`separate_rows()`

## Strings -\> Rows


```r
starwars %>% 
  select(name, eye_color) %>% 
  filter(str_detect(eye_color, ", ")) %>% 
  separate_rows(eye_color)
#> [90m# A tibble: 4 x 2[39m
#>   [1mname[22m     [1meye_color[22m
#>   [3m[90m<chr>[39m[23m    [3m[90m<chr>[39m[23m    
#> [90m1[39m R4-P17   red      
#> [90m2[39m R4-P17   blue     
#> [90m3[39m Grievous green    
#> [90m4[39m Grievous yellow
```

### Exercises

1.  Split up a string like `"apples, pears, and bananas"` into individual components.

2.  Why is it better to split up by `boundary("word")` than `" "`?

3.  What does splitting with an empty string (`""`) do?
    Experiment, and then read the documentation.

## Other writing systems {#other-languages}

Unicode is a system for representing the many writing systems used around the world.
Fundamental unit is a **code point**.
This usually represents something like a letter or symbol, but might also be formatting like a diacritic mark or a (e.g.) the skin tone of an emoji.
Character vs grapheme cluster.

Include some examples from <https://gankra.github.io/blah/text-hates-you/>.

All stringr functions default to the English locale.
This ensures that your code works the same way on every system, avoiding subtle bugs.

Maybe things you think are true, but aren't list?

### Encoding

You will not generally find the base R `Encoding()` to be useful because it only supports three different encodings (and interpreting what they mean is non-trivial) and it only tells you the encoding that R thinks it is, not what it really is.
And typically the problem is that the declaring encoding is wrong.

The tidyverse follows best practices[^strings-3] of using UTF-8 everywhere, so any string you create with the tidyverse will use UTF-8.
It's still possible to have problems, but they'll typically arise during data import.
Once you've diagnosed you have an encoding problem, you should fix it in data import (i.e. by using the `encoding` argument to `readr::locale()`).

[^strings-3]: <http://utf8everywhere.org>

### Length and subsetting

This seems like a straightforward computation if you're only familiar with English, but things get complex quick when working with other languages.

Four most common are Latin, Chinese, Arabic, and Devangari, which represent three different systems of writing systems:

-   Latin uses an alphabet, where each consonant and vowel gets its own letter.

-   Chinese.
    Logograms.
    Half width vs full width.
    English letters are roughly twice as high as they are wide.
    Chinese characters are roughly square.

-   Arabic is an abjad, only consonants are written and vowels are optionally as diacritics.
    Additionally, it's written from right-to-left, so the first letter is the letter on the far right.

-   Devangari is an abugida where each symbol represents a consonant-vowel pair, , vowel notation secondary.

> For instance, 'ch' is two letters in English and Latin, but considered to be one letter in Czech and Slovak.
> --- <http://utf8everywhere.org>


```r
# But
str_split("check", boundary("character", locale = "cs_CZ"))
#> [[1]]
#> [1] "c" "h" "e" "c" "k"
```

This is a problem even with Latin alphabets because many languages use **diacritics**, glyphs added to the basic alphabet.
This is a problem because Unicode provides two ways of representing characters with accents: many common characters have a special codepoint, but others can be built up from individual components.


```r
x <- c("á", "x́")
str_length(x)
#> [1] 1 9
# str_width(x)
str_sub(x, 1, 1)
#> [1] "á" "x"

# stri_width(c("全形", "ab"))
# 0, 1, or 2
# but this assumes no font substitution
```


```r
cyrillic_a <- "А"
latin_a <- "A"
cyrillic_a == latin_a
#> [1] FALSE
stringi::stri_escape_unicode(cyrillic_a)
#> [1] "\\u0410"
stringi::stri_escape_unicode(latin_a)
#> [1] "A"
```

### Collation rules

`coll()`: compare strings using standard **coll**ation rules.
This is useful for doing case insensitive matching.
Note that `coll()` takes a `locale` parameter that controls which rules are used for comparing characters.
Unfortunately different parts of the world use different rules!B
oth `fixed()` and `regex()` have `ignore_case` arguments, but they do not allow you to pick the locale: they always use the default locale.
You can see what that is with the following code; more on stringi later.


```r
a1 <- "\u00e1"
a2 <- "a\u0301"
c(a1, a2)
#> [1] "á" "a<U+0301>"
a1 == a2
#> [1] FALSE

str_detect(a1, fixed(a2))
#> [1] FALSE
str_detect(a1, coll(a2))
#> [1] TRUE
```

The downside of `coll()` is speed; because the rules for recognising which characters are the same are complicated, `coll()` is relatively slow compared to `regex()` and `fixed()`.

### Upper and lower case

Relatively few writing systems have upper and lower case: Latin, Greek, and Cyrillic, plus a handful of lessor known languages.

Above I used `str_to_lower()` to change the text to lower case.
You can also use `str_to_upper()` or `str_to_title()`.
However, changing case is more complicated than it might at first appear because different languages have different rules for changing case.
You can pick which set of rules to use by specifying a locale:


```r
# Turkish has two i's: with and without a dot, and it
# has a different rule for capitalising them:
str_to_upper(c("i", "ı"))
#> [1] "I"        "<U+0131>"
str_to_upper(c("i", "ı"), locale = "tr")
#> [1] "<U+0130>" "<U+0131>"
```

The locale is specified as a ISO 639 language code, which is a two or three letter abbreviation.
If you don't already know the code for your language, [Wikipedia](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) has a good list, and you can see which are supported with `stringi::stri_locale_list()`.
If you leave the locale blank, it will use English.

The locale also affects case-insensitive matching, which `coll(ignore_case =  TRUE)` which you can control with `coll()`:


```r
i <- c("Iİiı")

str_view_all(i, coll("i", ignore_case = TRUE))
#> [36m<I>[39m<U+0130>[36m<i>[39m<U+0131>
str_view_all(i, coll("i", ignore_case = TRUE, locale = "tr"))
#> I<U+0130>[36m<i>[39m<U+0131>
```

You can also do case insensitive matching this `fixed(ignore_case = TRUE)`, but this uses a simple approximation which will not work in all cases.

### Sorting

Unicode collation algorithm: <https://unicode.org/reports/tr10/>

Another important operation that's affected by the locale is sorting.
The base R `order()` and `sort()` functions sort strings using the current locale.
If you want robust behaviour across different computers, you may want to use `str_sort()` and `str_order()` which take an additional `locale` argument.

Can also control the "strength", which determines how accents are sorted.


```r
str_sort(c("a", "ch", "c", "h"))
#> [1] "a"  "c"  "ch" "h"
str_sort(c("a", "ch", "c", "h"), locale = "cs_CZ")
#> [1] "a"  "c"  "h"  "ch"
```

TODO: add connection to `arrange()`
