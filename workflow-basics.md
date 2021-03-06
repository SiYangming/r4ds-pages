# Workflow: basics

You now have some experience running R code.
We didn't give you many details, but you've obviously figured out the basics, or you would've thrown this book away in frustration!
Frustration is natural when you start programming in R, because it is such a stickler for punctuation, and even one character out of place will cause it to complain.
But while you should expect to be a little frustrated, take comfort in that it's both typical and temporary: it happens to everyone, and the only way to get over it is to keep trying.

Before we go any further, let's make sure you've got a solid foundation in running R code, and that you know about some of the most helpful RStudio features.

## Coding basics

Let's review some basics we've so far omitted in the interests of getting you plotting as quickly as possible.
You can use R as a calculator:


```r
1 / 200 * 30
#> [1] 0.15
(59 + 73 + 2) / 3
#> [1] 44.66667
sin(pi / 2)
#> [1] 1
```

You can create new objects with `<-`:


```r
x <- 3 * 4
```

All R statements where you create objects, **assignment** statements, have the same form:


```r
object_name <- value
```

When reading that code say "object name gets value" in your head.

You will make lots of assignments and `<-` is a pain to type.
Don't be lazy and use `=`: it will work, but it will cause confusion later.
Instead, use RStudio's keyboard shortcut: Alt + - (the minus sign).
Notice that RStudio automagically surrounds `<-` with spaces, which is a good code formatting practice.
Code is miserable to read on a good day, so giveyoureyesabreak and use spaces.

## What's in a name?

Object names must start with a letter, and can only contain letters, numbers, `_` and `.`.
You want your object names to be descriptive, so you'll need a convention for multiple words.
We recommend **snake_case** where you separate lowercase words with `_`.


```r
i_use_snake_case
otherPeopleUseCamelCase
some.people.use.periods
And_aFew.People_RENOUNCEconvention
```

We'll come back to code style later, in Chapter \@ref(functions) on functions.
If you're interested in learning more about about best practices for code style, I also recommend The tidyverse style guide: [https://style.tidyverse.org](https://style.tidyverse.org/).

You can inspect an object by typing its name:


```r
x
#> [1] 12
```

Make another assignment:


```r
this_is_a_really_long_name <- 2.5
```

To inspect this object, try out RStudio's completion facility: type "this", press TAB, add characters until you have a unique prefix, then press return.

Ooops, you made a mistake!
`this_is_a_really_long_name` should have value 3.5 not 2.5.
Use another keyboard shortcut to help you fix it.
Type "this" then press Cmd/Ctrl + ???.
That will list all the commands you've typed that start those letters.
Use the arrow keys to navigate, then press enter to retype the command.
Change 2.5 to 3.5 and rerun.

Make yet another assignment:


```r
r_rocks <- 2 ^ 3
```

Let's try to inspect it:


```r
r_rock
#> Error: object 'r_rock' not found
R_rocks
#> Error: object 'R_rocks' not found
```

There's an implied contract between you and R: it will do the tedious computation for you, but in return, you must be completely precise in your instructions.
Typos matter.
Case matters.

## Calling functions

R has a large collection of built-in functions that are called like this:


```r
function_name(arg1 = val1, arg2 = val2, ...)
```

Let's try using `seq()` which makes regular **seq**uences of numbers and, while we're at it, learn more helpful features of RStudio.
Type `se` and hit TAB.
A popup shows you possible completions.
Specify `seq()` by typing more (a `q`) to disambiguate, or by using ???/??? arrows to select.
Notice the floating tooltip that pops up, reminding you of the function's arguments and purpose.
If you want more help, press F1 to get all the details in the help tab in the lower right pane.

Press TAB once more when you've selected the function you want.
RStudio will add matching opening (`(`) and closing (`)`) parentheses for you.
Type the arguments `1, 10` and hit return.


```r
seq(1, 10)
#>  [1]  1  2  3  4  5  6  7  8  9 10
```

Type this code and notice you get similar assistance with the paired quotation marks:


```r
x <- "hello world"
```

Quotation marks and parentheses must always come in a pair.
RStudio does its best to help you, but it's still possible to mess up and end up with a mismatch.
If this happens, R will show you the continuation character "+":

    > x <- "hello
    +

The `+` tells you that R is waiting for more input; it doesn't think you're done yet.
Usually that means you've forgotten either a `"` or a `)`. Either add the missing pair, or press ESCAPE to abort the expression and try again.

If you make an assignment, you don't get to see the value.
You're then tempted to immediately double-check the result:


```r
y <- seq(1, 10, length.out = 5)
y
#> [1]  1.00  3.25  5.50  7.75 10.00
```

This common action can be shortened by surrounding the assignment with parentheses, which causes assignment and "print to screen" to happen.


```r
(y <- seq(1, 10, length.out = 5))
#> [1]  1.00  3.25  5.50  7.75 10.00
```

Now look at your environment in the upper right pane:

<img src="screenshots/rstudio-env.png" width="597" style="display: block; margin: auto;" />

Here you can see all of the objects that you've created.

## Exercises

1.  Why does this code not work?

    
    ```r
    my_variable <- 10
    my_var??able
    #> Error: <text>:2:14: unexpected '>'
    #> 1: my_variable <- 10
    #> 2: my_var<U+0131>
    #>                 ^
    ```

    Look carefully!
    (This may seem like an exercise in pointlessness, but training your brain to notice even the tiniest difference will pay off when programming.)

2.  Tweak each of the following R commands so that they run correctly:

    
    ```r
    libary(tidyverse)
    
    ggplot(dota = mpg) + 
      geom_point(maping = aes(x = displ, y = hwy))
    ```

3.  Press Alt + Shift + K.
    What happens?
    How can you get to the same place using the menus?
