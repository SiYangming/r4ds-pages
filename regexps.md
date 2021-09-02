# Regular expressions

::: {.rmdnote}
You are reading the work-in-progress second edition of R for Data Science. This chapter is currently undergoing heavy restructuring and may be confusing or incomplete. You can find the polished first edition at <https://r4ds.had.co.nz>.
:::

## Introduction

The focus of this chapter will be on regular expressions, or regexps for short.
Regular expressions are useful because strings usually contain unstructured or semi-structured data, and regexps are a concise language for describing patterns in strings.
When you first look at a regexp, you'll think a cat walked across your keyboard, but as your understanding improves they will soon start to make sense.

## Matching patterns with regular expressions

Regexps are a very terse language that allow you to describe patterns in strings.
They take a little while to get your head around, but once you understand them, you'll find them extremely useful.

To learn regular expressions, we'll use `str_view()` and `str_view_all()`.
These functions take a character vector and a regular expression, and show you how they match.
We'll start with very simple regular expressions and then gradually get more and more complicated.
Once you've mastered pattern matching, you'll learn how to apply those ideas with various stringr functions.

### Prerequisites

This chapter will focus on the **stringr** package for string manipulation, which is part of the core tidyverse.


```r
library(tidyverse)
```

## Basic matches

The simplest patterns match exact strings:


```r
x <- c("apple", "banana", "pear")
str_view(x, "an")
#> apple
#> b[36m<an>[39mana
#> pear
```

The next step up in complexity is `.`, which matches any character (except a newline):


```r
str_view(x, ".a.")
#> apple
#> [36m<ban>[39mana
#> p[36m<ear>[39m
```

But if "`.`" matches any character, how do you match the character "`.`"?
You need to use an "escape" to tell the regular expression you want to match it exactly, not use its special behaviour.
Like strings, regexps use the backslash, `\`, to escape special behaviour.
So to match an `.`, you need the regexp `\.`.
Unfortunately this creates a problem.
We use strings to represent regular expressions, and `\` is also used as an escape symbol in strings.
So to create the regular expression `\.` we need the string `"\\."`.


```r
# To create the regular expression, we need \\
dot <- "\\."

# But the expression itself only contains one:
writeLines(dot)
#> \.

# And this tells R to look for an explicit .
str_view(c("abc", "a.c", "bef"), "a\\.c")
#> abc
#> [36m<a.c>[39m
#> bef
```

If `\` is used as an escape character in regular expressions, how do you match a literal `\`?
Well you need to escape it, creating the regular expression `\\`.
To create that regular expression, you need to use a string, which also needs to escape `\`.
That means to match a literal `\` you need to write `"\\\\"` --- you need four backslashes to match one!


```r
x <- "a\\b"
writeLines(x)
#> a\b

str_view(x, "\\\\")
#> a[36m<\>[39mb
```

In this book, I'll write regular expression as `\.` and strings that represent the regular expression as `"\\."`.

### Exercises

1.  Explain why each of these strings don't match a `\`: `"\"`, `"\\"`, `"\\\"`.

2.  How would you match the sequence `"'\`?

3.  What patterns will the regular expression `\..\..\..` match?
    How would you represent it as a string?

## Anchors

By default, regular expressions will match any part of a string.
It's often useful to *anchor* the regular expression so that it matches from the start or end of the string.
You can use:

-   `^` to match the start of the string.
-   `$` to match the end of the string.


```r
x <- c("apple", "banana", "pear")
str_view(x, "^a")
#> [36m<a>[39mpple
#> banana
#> pear
str_view(x, "a$")
#> apple
#> banan[36m<a>[39m
#> pear
```

To remember which is which, try this mnemonic which I learned from [Evan Misshula](https://twitter.com/emisshula/status/323863393167613953): if you begin with power (`^`), you end up with money (`$`).

To force a regular expression to only match a complete string, anchor it with both `^` and `$`:


```r
x <- c("apple pie", "apple", "apple cake")
str_view(x, "apple")
#> [36m<apple>[39m pie
#> [36m<apple>[39m
#> [36m<apple>[39m cake
str_view(x, "^apple$")
#> apple pie
#> [36m<apple>[39m
#> apple cake
```

You can also match the boundary between words with `\b`.
I don't often use this in R, but I will sometimes use it when I'm doing a search in RStudio when I want to find the name of a function that's a component of other functions.
For example, I'll search for `\bsum\b` to avoid matching `summarise`, `summary`, `rowsum` and so on.

### Exercises

1.  How would you match the literal string `"$^$"`?

2.  Given the corpus of common words in `stringr::words`, create regular expressions that find all words that:

    a.  Start with "y".
    b.  End with "x"
    c.  Are exactly three letters long. (Don't cheat by using `str_length()`!)
    d.  Have seven letters or more.

    Since this list is long, you might want to use the `match` argument to `str_view()` to show only the matching or non-matching words.

## Overlapping and zero-width patterns

Note that matches never overlap.
For example, in `"abababa"`, how many times will the pattern `"aba"` match?
Regular expressions say two, not three:


```r
str_count("abababa", "aba")
#> [1] 2
str_view_all("abababa", "aba")
#> [36m<aba>[39mb[36m<aba>[39m
```

## Character classes and alternatives

There are a number of special patterns that match more than one character.
You've already seen `.`, which matches any character apart from a newline.
There are four other useful tools:

-   `\d`: matches any digit.
-   `\s`: matches any whitespace (e.g. space, tab, newline).
-   `[abc]`: matches a, b, or c.
-   `[^abc]`: matches anything except a, b, or c.

Remember, to create a regular expression containing `\d` or `\s`, you'll need to escape the `\` for the string, so you'll type `"\\d"` or `"\\s"`.

A character class containing a single character is a nice alternative to backslash escapes when you want to include a single metacharacter in a regex.
Many people find this more readable.


```r
# Look for a literal character that normally has special meaning in a regex
str_view(c("abc", "a.c", "a*c", "a c"), "a[.]c")
#> abc
#> [36m<a.c>[39m
#> a*c
#> a c
str_view(c("abc", "a.c", "a*c", "a c"), ".[*]c")
#> abc
#> a.c
#> [36m<a*c>[39m
#> a c
str_view(c("abc", "a.c", "a*c", "a c"), "a[ ]")
#> abc
#> a.c
#> a*c
#> [36m<a >[39mc
```

This works for most (but not all) regex metacharacters: `$` `.` `|` `?` `*` `+` `(` `)` `[` `{`.
Unfortunately, a few characters have special meaning even inside a character class and must be handled with backslash escapes: `]` `\` `^` and `-`.

You can use *alternation* to pick between one or more alternative patterns.
For example, `abc|d..f` will match either '"abc"', or `"deaf"`.
Note that the precedence for `|` is low, so that `abc|xyz` matches `abc` or `xyz` not `abcyz` or `abxyz`.
Like with mathematical expressions, if precedence ever gets confusing, use parentheses to make it clear what you want:


```r
str_view(c("grey", "gray"), "gr(e|a)y")
#> [36m<grey>[39m
#> [36m<gray>[39m
```

When you have complex logical conditions (e.g. match a or b but not c unless d) it's often easier to combine multiple `str_detect()` calls with logical operators, rather than trying to create a single regular expression.
For example, here are two ways to find all words that don't contain any vowels:


```r
# Find all words containing at least one vowel, and negate
no_vowels_1 <- !str_detect(words, "[aeiou]")
# Find all words consisting only of consonants (non-vowels)
no_vowels_2 <- str_detect(words, "^[^aeiou]+$")
identical(no_vowels_1, no_vowels_2)
#> [1] TRUE
```

The results are identical, but I think the first approach is significantly easier to understand.
If your regular expression gets overly complicated, try breaking it up into smaller pieces, giving each piece a name, and then combining the pieces with logical operations.

### Exercises

1.  Create regular expressions to find all words that:

    a.  Start with a vowel.
    b.  That only contain consonants. (Hint: thinking about matching "not"-vowels.)
    c.  End with `ed`, but not with `eed`.
    d.  End with `ing` or `ise`.

2.  Empirically verify the rule "i before e except after c".

3.  Is "q" always followed by a "u"?

4.  Write a regular expression that matches a word if it's probably written in British English, not American English.

5.  Create a regular expression that will match telephone numbers as commonly written in your country.

## Repetition / Quantifiers

The next step up in power involves controlling how many times a pattern matches:

-   `?`: 0 or 1
-   `+`: 1 or more
-   `*`: 0 or more


```r
x <- "1888 is the longest year in Roman numerals: MDCCCLXXXVIII"
str_view(x, "CC?")
#> 1888 is the longest year in Roman numerals: MD[36m<CC>[39mCLXXXVIII
str_view(x, "CC+")
#> 1888 is the longest year in Roman numerals: MD[36m<CCC>[39mLXXXVIII
str_view(x, 'C[LX]+')
#> 1888 is the longest year in Roman numerals: MDCC[36m<CLXXX>[39mVIII
```

Note that the precedence of these operators is high, so you can write: `colou?r` to match either American or British spellings.
That means most uses will need parentheses, like `bana(na)+`.

You can also specify the number of matches precisely:

-   `{n}`: exactly n
-   `{n,}`: n or more
-   `{1,m}`: at most m
-   `{n,m}`: between n and m


```r
str_view(x, "C{2}")
#> 1888 is the longest year in Roman numerals: MD[36m<CC>[39mCLXXXVIII
str_view(x, "C{2,}")
#> 1888 is the longest year in Roman numerals: MD[36m<CCC>[39mLXXXVIII
str_view(x, "C{1,3}")
#> 1888 is the longest year in Roman numerals: MD[36m<CCC>[39mLXXXVIII
str_view(x, "C{2,3}")
#> 1888 is the longest year in Roman numerals: MD[36m<CCC>[39mLXXXVIII
```

By default these matches are "greedy": they will match the longest string possible.
You can make them "lazy", matching the shortest string possible by putting a `?` after them.
This is an advanced feature of regular expressions, but it's useful to know that it exists:


```r
str_view(x, 'C{2,3}?')
#> 1888 is the longest year in Roman numerals: MD[36m<CC>[39mCLXXXVIII
str_view(x, 'C[LX]+?')
#> 1888 is the longest year in Roman numerals: MDCC[36m<CL>[39mXXXVIII
```

Collectively, these operators are called **quantifiers** because they quantify how many times a match can occur.

### Exercises

1.  Describe the equivalents of `?`, `+`, `*` in `{m,n}` form.

2.  Describe in words what these regular expressions match: (read carefully to see if I'm using a regular expression or a string that defines a regular expression.)

    a.  `^.*$`
    b.  `"\\{.+\\}"`
    c.  `\d{4}-\d{2}-\d{2}`
    d.  `"\\\\{4}"`

3.  Create regular expressions to find all words that:

    a.  Start with three consonants.
    b.  Have three or more vowels in a row.
    c.  Have two or more vowel-consonant pairs in a row.

4.  Solve the beginner regexp crosswords at [\<https://regexcrossword.com/challenges/beginner\>](https://regexcrossword.com/challenges/beginner){.uri}.

## Grouping and backreferences

Earlier, you learned about parentheses as a way to disambiguate complex expressions.
Parentheses also create a *numbered* capturing group (number 1, 2 etc.).
A capturing group stores *the part of the string* matched by the part of the regular expression inside the parentheses.
You can refer to the same text as previously matched by a capturing group with *backreferences*, like `\1`, `\2` etc.
For example, the following regular expression finds all fruits that have a repeated pair of letters.


```r
str_view(fruit, "(..)\\1", match = TRUE)
#> b[36m<anan>[39ma
#> [36m<coco>[39mnut
#> [36m<cucu>[39mmber
#> [36m<juju>[39mbe
#> [36m<papa>[39mya
#> s[36m<alal>[39m berry
```

(Shortly, you'll also see how they're useful in conjunction with `str_match()`.)

Also use for replacement:


```r
sentences %>% 
  str_replace("([^ ]+) ([^ ]+) ([^ ]+)", "\\1 \\3 \\2") %>% 
  head(5)
#> [1] "The canoe birch slid on the smooth planks." 
#> [2] "Glue sheet the to the dark blue background."
#> [3] "It's to easy tell the depth of a well."     
#> [4] "These a days chicken leg is a rare dish."   
#> [5] "Rice often is served in round bowls."
```

Names that start and end with the same letter.
Implement with `str_sub()` instead.

### Exercises

1.  Describe, in words, what these expressions will match:

    a.  `(.)\1\1`
    b.  `"(.)(.)\\2\\1"`
    c.  `(..)\1`
    d.  `"(.).\\1.\\1"`
    e.  `"(.)(.)(.).*\\3\\2\\1"`

2.  Construct regular expressions to match words that:

    a.  Start and end with the same character.
    b.  Contain a repeated pair of letters (e.g. "church" contains "ch" repeated twice.)
    c.  Contain one letter repeated in at least three places (e.g. "eleven" contains three "e"s.)

## Other uses of regular expressions

There are two useful function in base R that also use regular expressions:

-   `apropos()` searches all objects available from the global environment.
    This is useful if you can't quite remember the name of the function.

    
    ```r
    apropos("replace")
    #> [1] "%+replace%"       "replace"          "replace_na"       "setReplaceMethod"
    #> [5] "str_replace"      "str_replace_all"  "str_replace_na"   "theme_replace"
    ```

-   `dir()` lists all the files in a directory.
    The `pattern` argument takes a regular expression and only returns file names that match the pattern.
    For example, you can find all the R Markdown files in the current directory with:

    
    ```r
    head(dir(pattern = "\\.Rmd$"))
    #> [1] "column-wise.Rmd"       "communicate-plots.Rmd" "communicate.Rmd"      
    #> [4] "contribute.Rmd"        "data-import.Rmd"       "data-tidy.Rmd"
    ```

    (If you're more comfortable with "globs" like `*.Rmd`, you can convert them to regular expressions with `glob2rx()`):

## Options

When you use a pattern that's a string, it's automatically wrapped into a call to `regex()`:


```r
# The regular call:
str_view(fruit, "nana")
# Is shorthand for
str_view(fruit, regex("nana"))
```

You can use the other arguments of `regex()` to control details of the match:

-   `ignore_case = TRUE` allows characters to match either their uppercase or lowercase forms.
    This always uses the current locale.

    
    ```r
    bananas <- c("banana", "Banana", "BANANA")
    str_view(bananas, "banana")
    #> [36m<banana>[39m
    #> Banana
    #> BANANA
    str_view(bananas, regex("banana", ignore_case = TRUE))
    #> [36m<banana>[39m
    #> [36m<Banana>[39m
    #> [36m<BANANA>[39m
    ```

-   `multiline = TRUE` allows `^` and `$` to match the start and end of each line rather than the start and end of the complete string.

    
    ```r
    x <- "Line 1\nLine 2\nLine 3"
    str_extract_all(x, "^Line")[[1]]
    #> [1] "Line"
    str_extract_all(x, regex("^Line", multiline = TRUE))[[1]]
    #> [1] "Line" "Line" "Line"
    ```

-   `comments = TRUE` allows you to use comments and white space to make complex regular expressions more understandable.
    Spaces are ignored, as is everything after `#`.
    To match a literal space, you'll need to escape it: `"\\ "`.

    
    ```r
    phone <- regex("
      \\(?     # optional opening parens
      (\\d{3}) # area code
      [) -]?   # optional closing parens, space, or dash
      (\\d{3}) # another three numbers
      [ -]?    # optional space or dash
      (\\d{3}) # three more numbers
      ", comments = TRUE)
    
    str_match("514-791-8141", phone)
    #>      [,1]          [,2]  [,3]  [,4] 
    #> [1,] "514-791-814" "514" "791" "814"
    ```

-   `dotall = TRUE` allows `.` to match everything, including `\n`.

## A caution

A word of caution before we continue: because regular expressions are so powerful, it's easy to try and solve every problem with a single regular expression.
In the words of Jamie Zawinski:

> Some people, when confronted with a problem, think "I know, I'll use regular expressions." Now they have two problems.

As a cautionary tale, check out this regular expression that checks if a email address is valid:

    (?:(?:\r\n)?[ \t])*(?:(?:(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t]
    )+|\Z|(?=[\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:
    \r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(
    ?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ 
    \t]))*"(?:(?:\r\n)?[ \t])*))*@(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\0
    31]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\
    ](?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+
    (?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:
    (?:\r\n)?[ \t])*))*|(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z
    |(?=[\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)
    ?[ \t])*)*\<(?:(?:\r\n)?[ \t])*(?:@(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\
    r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[
     \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)
    ?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t]
    )*))*(?:,@(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[
     \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*
    )(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t]
    )+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*))*)
    *:(?:(?:\r\n)?[ \t])*)?(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+
    |\Z|(?=[\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r
    \n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:
    \r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t
    ]))*"(?:(?:\r\n)?[ \t])*))*@(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031
    ]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](
    ?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?
    :(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?
    :\r\n)?[ \t])*))*\>(?:(?:\r\n)?[ \t])*)|(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?
    :(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?
    [ \t]))*"(?:(?:\r\n)?[ \t])*)*:(?:(?:\r\n)?[ \t])*(?:(?:(?:[^()<>@,;:\\".\[\] 
    \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|
    \\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>
    @,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|"
    (?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t])*))*@(?:(?:\r\n)?[ \t]
    )*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\
    ".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?
    :[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[
    \]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*))*|(?:[^()<>@,;:\\".\[\] \000-
    \031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\.|(
    ?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t])*)*\<(?:(?:\r\n)?[ \t])*(?:@(?:[^()<>@,;
    :\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([
    ^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\"
    .\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\
    ]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*))*(?:,@(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\
    [\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\
    r\\]|\\.)*\](?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] 
    \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]
    |\\.)*\](?:(?:\r\n)?[ \t])*))*)*:(?:(?:\r\n)?[ \t])*)?(?:[^()<>@,;:\\".\[\] \0
    00-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\
    .|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,
    ;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|"(?
    :[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t])*))*@(?:(?:\r\n)?[ \t])*
    (?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".
    \[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t])*(?:[
    ^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\]
    ]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*))*\>(?:(?:\r\n)?[ \t])*)(?:,\s*(
    ?:(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\
    ".\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t])*)(?:\.(?:(
    ?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[
    \["()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t
    ])*))*@(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t
    ])+|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*)(?
    :\.(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|
    \Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*))*|(?:
    [^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".\[\
    ]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t])*)*\<(?:(?:\r\n)
    ?[ \t])*(?:@(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["
    ()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)
    ?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>
    @,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*))*(?:,@(?:(?:\r\n)?[
     \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,
    ;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*)(?:\.(?:(?:\r\n)?[ \t]
    )*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\
    ".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*))*)*:(?:(?:\r\n)?[ \t])*)?
    (?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\["()<>@,;:\\".
    \[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t])*)(?:\.(?:(?:
    \r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z|(?=[\[
    "()<>@,;:\\".\[\]]))|"(?:[^\"\r\\]|\\.|(?:(?:\r\n)?[ \t]))*"(?:(?:\r\n)?[ \t])
    *))*@(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])
    +|\Z|(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*)(?:\
    .(?:(?:\r\n)?[ \t])*(?:[^()<>@,;:\\".\[\] \000-\031]+(?:(?:(?:\r\n)?[ \t])+|\Z
    |(?=[\["()<>@,;:\\".\[\]]))|\[([^\[\]\r\\]|\\.)*\](?:(?:\r\n)?[ \t])*))*\>(?:(
    ?:\r\n)?[ \t])*))*)?;\s*)

This is a somewhat pathological example (because email addresses are actually surprisingly complex), but is used in real code.
See the Stack Overflow discussion at <http://stackoverflow.com/a/201378> for more details.

Don't forget that you're in a programming language and you have other tools at your disposal.
Instead of creating one complex regular expression, it's often easier to write a series of simpler regexps.
If you get stuck trying to create a single regexp that solves your problem, take a step back and think if you could break the problem down into smaller pieces, solving each challenge before moving onto the next one.
