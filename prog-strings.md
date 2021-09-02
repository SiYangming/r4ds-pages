# Programming with strings

::: {.rmdnote}
You are reading the work-in-progress second edition of R for Data Science. This chapter is currently currently a dumping ground for ideas, and we don't recommend reading it. You can find the polished first edition at <https://r4ds.had.co.nz>.
:::


```r
library(stringr)
library(tidyr)
library(tibble)
```

### str_c

`NULL`s are silently dropped.
This is particularly useful in conjunction with `if`:


```r
name <- "Hadley"
time_of_day <- "morning"
birthday <- FALSE

str_c(
  "Good ", time_of_day, " ", name,
  if (birthday) " and HAPPY BIRTHDAY",
  "."
)
#> [1] "Good morning Hadley."
```

## Performance

`fixed()`: matches exactly the specified sequence of bytes.
It ignores all special regular expressions and operates at a very low level.
This allows you to avoid complex escaping and can be much faster than regular expressions.
The following microbenchmark shows that it's about 3x faster for a simple example.


```r
microbenchmark::microbenchmark(
  fixed = str_detect(sentences, fixed("the")),
  regex = str_detect(sentences, "the"),
  times = 20
)
#> Unit: microseconds
#>   expr   min     lq    mean  median     uq    max neval
#>  fixed 194.8 263.70 448.350  348.65  440.8 2459.5    20
#>  regex 527.8 761.45 946.625 1013.35 1106.4 1280.4    20
```

As you saw with `str_split()` you can use `boundary()` to match boundaries.
You can also use it with the other functions:


```r
x <- "This is a sentence."
str_view_all(x, boundary("word"))
#> [36m<This>[39m [36m<is>[39m [36m<a>[39m [36m<sentence>[39m.
str_extract_all(x, boundary("word"))
#> [[1]]
#> [1] "This"     "is"       "a"        "sentence"
```

### 

### Extract


```r
colours <- c("red", "orange", "yellow", "green", "blue", "purple")
colour_match <- str_c(colours, collapse = "|")
colour_match
#> [1] "red|orange|yellow|green|blue|purple"

more <- sentences[str_count(sentences, colour_match) > 1]
str_extract_all(more, colour_match)
#> [[1]]
#> [1] "blue" "red" 
#> 
#> [[2]]
#> [1] "green" "red"  
#> 
#> [[3]]
#> [1] "orange" "red"
```

If you use `simplify = TRUE`, `str_extract_all()` will return a matrix with short matches expanded to the same length as the longest:


```r

str_extract_all(more, colour_match, simplify = TRUE)
#>      [,1]     [,2] 
#> [1,] "blue"   "red"
#> [2,] "green"  "red"
#> [3,] "orange" "red"

x <- c("a", "a b", "a b c")
str_extract_all(x, "[a-z]", simplify = TRUE)
#>      [,1] [,2] [,3]
#> [1,] "a"  ""   ""  
#> [2,] "a"  "b"  ""  
#> [3,] "a"  "b"  "c"
```

We don't talk about matrices here, but they are useful elsewhere.

### Exercises

1.  From the Harvard sentences data, extract:

    1.  The first word from each sentence.
    2.  All words ending in `ing`.
    3.  All plurals.

## Grouped matches

Earlier in this chapter we talked about the use of parentheses for clarifying precedence and for backreferences when matching.
You can also use parentheses to extract parts of a complex match.
For example, imagine we want to extract nouns from the sentences.
As a heuristic, we'll look for any word that comes after "a" or "the".
Defining a "word" in a regular expression is a little tricky, so here I use a simple approximation: a sequence of at least one character that isn't a space.


```r
noun <- "(a|the) ([^ ]+)"

has_noun <- sentences %>%
  str_subset(noun) %>%
  head(10)
has_noun %>% 
  str_extract(noun)
#>  [1] "the smooth" "the sheet"  "the depth"  "a chicken"  "the parked"
#>  [6] "the sun"    "the huge"   "the ball"   "the woman"  "a helps"
```

`str_extract()` gives us the complete match; `str_match()` gives each individual component.
Instead of a character vector, it returns a matrix, with one column for the complete match followed by one column for each group:


```r
has_noun %>% 
  str_match(noun)
#>       [,1]         [,2]  [,3]     
#>  [1,] "the smooth" "the" "smooth" 
#>  [2,] "the sheet"  "the" "sheet"  
#>  [3,] "the depth"  "the" "depth"  
#>  [4,] "a chicken"  "a"   "chicken"
#>  [5,] "the parked" "the" "parked" 
#>  [6,] "the sun"    "the" "sun"    
#>  [7,] "the huge"   "the" "huge"   
#>  [8,] "the ball"   "the" "ball"   
#>  [9,] "the woman"  "the" "woman"  
#> [10,] "a helps"    "a"   "helps"
```

(Unsurprisingly, our heuristic for detecting nouns is poor, and also picks up adjectives like smooth and parked.)

## Spitting

Use `str_split()` to split a string up into pieces.
For example, we could split sentences into words:


```r
sentences %>%
  head(5) %>% 
  str_split(" ")
#> [[1]]
#> [1] "The"     "birch"   "canoe"   "slid"    "on"      "the"     "smooth" 
#> [8] "planks."
#> 
#> [[2]]
#> [1] "Glue"        "the"         "sheet"       "to"          "the"        
#> [6] "dark"        "blue"        "background."
#> 
#> [[3]]
#> [1] "It's"  "easy"  "to"    "tell"  "the"   "depth" "of"    "a"     "well."
#> 
#> [[4]]
#> [1] "These"   "days"    "a"       "chicken" "leg"     "is"      "a"      
#> [8] "rare"    "dish."  
#> 
#> [[5]]
#> [1] "Rice"   "is"     "often"  "served" "in"     "round"  "bowls."
```

Because each component might contain a different number of pieces, this returns a list.
If you're working with a length-1 vector, the easiest thing is to just extract the first element of the list:


```r
"a|b|c|d" %>% 
  str_split("\\|") %>% 
  .[[1]]
#> [1] "a" "b" "c" "d"
```

Otherwise, like the other stringr functions that return a list, you can use `simplify = TRUE` to return a matrix:


```r
sentences %>%
  head(5) %>% 
  str_split(" ", simplify = TRUE)
#>      [,1]    [,2]    [,3]    [,4]      [,5]  [,6]    [,7]     [,8]         
#> [1,] "The"   "birch" "canoe" "slid"    "on"  "the"   "smooth" "planks."    
#> [2,] "Glue"  "the"   "sheet" "to"      "the" "dark"  "blue"   "background."
#> [3,] "It's"  "easy"  "to"    "tell"    "the" "depth" "of"     "a"          
#> [4,] "These" "days"  "a"     "chicken" "leg" "is"    "a"      "rare"       
#> [5,] "Rice"  "is"    "often" "served"  "in"  "round" "bowls." ""           
#>      [,9]   
#> [1,] ""     
#> [2,] ""     
#> [3,] "well."
#> [4,] "dish."
#> [5,] ""
```

You can also request a maximum number of pieces:


```r
fields <- c("Name: Hadley", "Country: NZ", "Age: 35")
fields %>% str_split(": ", n = 2, simplify = TRUE)
#>      [,1]      [,2]    
#> [1,] "Name"    "Hadley"
#> [2,] "Country" "NZ"    
#> [3,] "Age"     "35"
```

Instead of splitting up strings by patterns, you can also split up by character, line, sentence and word `boundary()`s:


```r
x <- "This is a sentence.  This is another sentence."
str_view_all(x, boundary("word"))
#> [36m<This>[39m [36m<is>[39m [36m<a>[39m [36m<sentence>[39m.  [36m<This>[39m [36m<is>[39m [36m<another>[39m [36m<sentence>[39m.

str_split(x, " ")[[1]]
#> [1] "This"      "is"        "a"         "sentence." ""          "This"     
#> [7] "is"        "another"   "sentence."
str_split(x, boundary("word"))[[1]]
#> [1] "This"     "is"       "a"        "sentence" "This"     "is"       "another" 
#> [8] "sentence"
```

Show how `separate_rows()` is a special case of `str_split()` + `summarise()`.

## Replace with function

## Locations

`str_locate()` and `str_locate_all()` give you the starting and ending positions of each match.
These are particularly useful when none of the other functions does exactly what you want.
You can use `str_locate()` to find the matching pattern, `str_sub()` to extract and/or modify them.

## stringi

stringr is built on top of the **stringi** package.
stringr is useful when you're learning because it exposes a minimal set of functions, which have been carefully picked to handle the most common string manipulation functions.
stringi, on the other hand, is designed to be comprehensive.
It contains almost every function you might ever need: stringi has 256 functions to stringr's 53.

If you find yourself struggling to do something in stringr, it's worth taking a look at stringi.
The packages work very similarly, so you should be able to translate your stringr knowledge in a natural way.
The main difference is the prefix: `str_` vs. `stri_`.

### Exercises

1.  Find the stringi functions that:

    a.  Count the number of words.
    b.  Find duplicated strings.
    c.  Generate random text.

2.  How do you control the language that `stri_sort()` uses for sorting?

### Exercises

1.  What do the `extra` and `fill` arguments do in `separate()`?
    Experiment with the various options for the following two toy datasets.

    
    ```r
    tibble(x = c("a,b,c", "d,e,f,g", "h,i,j")) %>%
      separate(x, c("one", "two", "three"))
    
    tibble(x = c("a,b,c", "d,e", "f,g,i")) %>%
      separate(x, c("one", "two", "three"))
    ```

2.  Both `unite()` and `separate()` have a `remove` argument.
    What does it do?
    Why would you set it to `FALSE`?

3.  Compare and contrast `separate()` and `extract()`.
    Why are there three variations of separation (by position, by separator, and with groups), but only one unite?

4.  In the following example we're using `unite()` to create a `date` column from `month` and `day` columns.
    How would you achieve the same outcome using `mutate()` and `paste()` instead of unite?

    
    ```r
    events <- tribble(
      ~month, ~day,
      1     , 20,
      1     , 21,
      1     , 22
    )
    
    events %>%
      unite("date", month:day, sep = "-", remove = FALSE)
    ```

5.  Write a function that turns (e.g.) a vector `c("a", "b", "c")` into the string `a, b, and c`.
    Think carefully about what it should do if given a vector of length 0, 1, or 2.
