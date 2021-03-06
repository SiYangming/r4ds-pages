# Introduction

Data science is an exciting discipline that allows you to turn raw data into understanding, insight, and knowledge.
The goal of "R for Data Science" is to help you learn the most important tools in R that will allow you to do data science.
After reading this book, you'll have the tools to tackle a wide variety of data science challenges, using the best parts of R.

## What you will learn

Data science is a huge field, and there's no way you can master it by reading a single book.
The goal of this book is to give you a solid foundation in the most important tools.
Our model of the tools needed in a typical data science project looks something like this:

<img src="diagrams/data-science.png" width="75%" style="display: block; margin: auto;" />

First you must **import** your data into R.
This typically means that you take data stored in a file, database, or web application programming interface (API), and load it into a data frame in R.
If you can't get your data into R, you can't do data science on it!

Once you've imported your data, it is a good idea to **tidy** it.
Tidying your data means storing it in a consistent form that matches the semantics of the dataset with the way it is stored.
In brief, when your data is tidy, each column is a variable, and each row is an observation.
Tidy data is important because the consistent structure lets you focus your struggle on questions about the data, not fighting to get the data into the right form for different functions.

Once you have tidy data, a common first step is to **transform** it.
Transformation includes narrowing in on observations of interest (like all people in one city, or all data from the last year), creating new variables that are functions of existing variables (like computing speed from distance and time), and calculating a set of summary statistics (like counts or means).
Together, tidying and transforming are called **wrangling**, because getting your data in a form that's natural to work with often feels like a fight!

Once you have tidy data with the variables you need, there are two main engines of knowledge generation: visualisation and modelling.
These have complementary strengths and weaknesses so any real analysis will iterate between them many times.

**Visualisation** is a fundamentally human activity.
A good visualisation will show you things that you did not expect, or raise new questions about the data.
A good visualisation might also hint that you're asking the wrong question, or you need to collect different data.
Visualisations can surprise you, but don't scale particularly well because they require a human to interpret them.

**Models** are complementary tools to visualisation.
Once you have made your questions sufficiently precise, you can use a model to answer them.
Models are a fundamentally mathematical or computational tool, so they generally scale well.
Even when they don't, it's usually cheaper to buy more computers than it is to buy more brains!
But every model makes assumptions, and by its very nature a model cannot question its own assumptions.
That means a model cannot fundamentally surprise you.

The last step of data science is **communication**, an absolutely critical part of any data analysis project.
It doesn't matter how well your models and visualisation have led you to understand the data unless you can also communicate your results to others.

Surrounding all these tools is **programming**.
Programming is a cross-cutting tool that you use in every part of the project.
You don't need to be an expert programmer to be a data scientist, but learning more about programming pays off because becoming a better programmer allows you to automate common tasks, and solve new problems with greater ease.

You'll use these tools in every data science project, but for most projects they're not enough.
There's a rough 80-20 rule at play; you can tackle about 80% of every project using the tools that you'll learn in this book, but you'll need other tools to tackle the remaining 20%.
Throughout this book we'll point you to resources where you can learn more.

## How this book is organised

The previous description of the tools of data science is organised roughly according to the order in which you use them in an analysis (although of course you'll iterate through them multiple times).
In our experience, however, this is not the best way to learn them:

-   Starting with data ingest and tidying is sub-optimal because 80% of the time it's routine and boring, and the other 20% of the time it's weird and frustrating.
    That's a bad place to start learning a new subject!
    Instead, we'll start with visualisation and transformation of data that's already been imported and tidied.
    That way, when you ingest and tidy your own data, your motivation will stay high because you know the pain is worth it.

-   Some topics are best explained with other tools.
    For example, we believe that it's easier to understand how models work if you already know about visualisation, tidy data, and programming.

-   Programming tools are not necessarily interesting in their own right, but do allow you to tackle considerably more challenging problems.
    We'll give you a selection of programming tools in the middle of the book, and then you'll see how they can combine with the data science tools to tackle interesting modelling problems.

Within each chapter, we try and stick to a similar pattern: start with some motivating examples so you can see the bigger picture, and then dive into the details.
Each section of the book is paired with exercises to help you practice what you've learned.
While it's tempting to skip the exercises, there's no better way to learn than practicing on real problems.

## What you won't learn

There are some important topics that this book doesn't cover.
We believe it's important to stay ruthlessly focused on the essentials so you can get up and running as quickly as possible.
That means this book can't cover every important topic.

### Big data

This book proudly focuses on small, in-memory datasets.
This is the right place to start because you can't tackle big data unless you have experience with small data.
The tools you learn in this book will easily handle hundreds of megabytes of data, and with a little care you can typically use them to work with 1-2 Gb of data.
If you're routinely working with larger data (10-100 Gb, say), you should learn more about [data.table](https://github.com/Rdatatable/data.table).
This book doesn't teach data.table because it has a very concise interface which makes it harder to learn since it offers fewer linguistic cues.
But if you're working with large data, the performance payoff is worth the extra effort required to learn it.

If your data is bigger than this, carefully consider if your big data problem might actually be a small data problem in disguise.
While the complete data might be big, often the data needed to answer a specific question is small.
You might be able to find a subset, subsample, or summary that fits in memory and still allows you to answer the question that you're interested in.
The challenge here is finding the right small data, which often requires a lot of iteration.

Another possibility is that your big data problem is actually a large number of small data problems.
Each individual problem might fit in memory, but you have millions of them.
For example, you might want to fit a model to each person in your dataset.
That would be trivial if you had just 10 or 100 people, but instead you have a million.
Fortunately each problem is independent of the others (a setup that is sometimes called embarrassingly parallel), so you just need a system (like Hadoop or Spark) that allows you to send different datasets to different computers for processing.
Once you've figured out how to answer the question for a single subset using the tools described in this book, you learn new tools like sparklyr, rhipe, and ddr to solve it for the full dataset.

### Python, Julia, and friends

In this book, you won't learn anything about Python, Julia, or any other programming language useful for data science.
This isn't because we think these tools are bad.
They're not!
And in practice, most data science teams use a mix of languages, often at least R and Python.

However, we strongly believe that it's best to master one tool at a time.
You will get better faster if you dive deep, rather than spreading yourself thinly over many topics.
This doesn't mean you should only know one thing, just that you'll generally learn faster if you stick to one thing at a time.
You should strive to learn new things throughout your career, but make sure your understanding is solid before you move on to the next interesting thing.

We think R is a great place to start your data science journey because it is an environment designed from the ground up to support data science.
R is not just a programming language, but it is also an interactive environment for doing data science.
To support interaction, R is a much more flexible language than many of its peers.
This flexibility comes with its downsides, but the big upside is how easy it is to evolve tailored grammars for specific parts of the data science process.
These mini languages help you think about problems as a data scientist, while supporting fluent interaction between your brain and the computer.

### Non-rectangular data

This book focuses exclusively on rectangular data: collections of values that are each associated with a variable and an observation.
There are lots of datasets that do not naturally fit in this paradigm, including images, sounds, trees, and text.
But rectangular data frames are extremely common in science and industry, and we believe that they are a great place to start your data science journey.

### Hypothesis confirmation

It's possible to divide data analysis into two camps: hypothesis generation and hypothesis confirmation (sometimes called confirmatory analysis).
The focus of this book is unabashedly on hypothesis generation, or data exploration.
Here you'll look deeply at the data and, in combination with your subject knowledge, generate many interesting hypotheses to help explain why the data behaves the way it does.
You evaluate the hypotheses informally, using your scepticism to challenge the data in multiple ways.

The complement of hypothesis generation is hypothesis confirmation.
Hypothesis confirmation is hard for two reasons:

1.  You need a precise mathematical model in order to generate falsifiable predictions.
    This often requires considerable statistical sophistication.

2.  You can only use an observation once to confirm a hypothesis.
    As soon as you use it more than once you're back to doing exploratory analysis.
    This means to do hypothesis confirmation you need to "preregister" (write out in advance) your analysis plan, and not deviate from it even when you have seen the data.

It's common to think about modelling as a tool for hypothesis confirmation, and visualisation as a tool for hypothesis generation.
But that's a false dichotomy: models are often used for exploration, and with a little care you can use visualisation for confirmation.
The key difference is how often do you look at each observation: if you look only once, it's confirmation; if you look more than once, it's exploration.

## Prerequisites

We've made a few assumptions about what you already know in order to get the most out of this book.
You should be generally numerically literate, and it's helpful if you have some programming experience already.
If you've never programmed before, you might find [Hands on Programming with R](http://amzn.com/1449359019) by Garrett to be a useful adjunct to this book.

There are four things you need to run the code in this book: R, RStudio, a collection of R packages called the **tidyverse**, and a handful of other packages.
Packages are the fundamental units of reproducible R code.
They include reusable functions, the documentation that describes how to use them, and sample data.

### R

To download R, go to CRAN, the **c**omprehensive **R** **a**rchive **n**etwork.
CRAN is composed of a set of mirror servers distributed around the world and is used to distribute R and R packages.
Don't try and pick a mirror that's close to you: instead use the cloud mirror, <https://cloud.r-project.org>, which automatically figures it out for you.

A new major version of R comes out once a year, and there are 2-3 minor releases each year.
It's a good idea to update regularly.
Upgrading can be a bit of a hassle, especially for major versions, which require you to reinstall all your packages, but putting it off only makes it worse.

### RStudio

RStudio is an integrated development environment, or IDE, for R programming.
Download and install it from <http://www.rstudio.com/download>.
RStudio is updated a couple of times a year.
When a new version is available, RStudio will let you know.
It's a good idea to upgrade regularly so you can take advantage of the latest and greatest features.
For this book, make sure you have at least RStudio 1.0.0.

When you start RStudio, you'll see two key regions in the interface:

<img src="diagrams/rstudio-console.png" width="75%" style="display: block; margin: auto;" />

For now, all you need to know is that you type R code in the console pane, and press enter to run it.
You'll learn more as we go along!

### The tidyverse

You'll also need to install some R packages.
An R **package** is a collection of functions, data, and documentation that extends the capabilities of base R.
Using packages is key to the successful use of R.
The majority of the packages that you will learn in this book are part of the so-called tidyverse.
The packages in the tidyverse share a common philosophy of data and R programming, and are designed to work together naturally.

You can install the complete tidyverse with a single line of code:


```r
install.packages("tidyverse")
```

On your own computer, type that line of code in the console, and then press enter to run it.
R will download the packages from CRAN and install them on to your computer.
If you have problems installing, make sure that you are connected to the internet, and that <https://cloud.r-project.org/> isn't blocked by your firewall or proxy.

You will not be able to use the functions, objects, and help files in a package until you load it with `library()`.
Once you have installed a package, you can load it with the `library()` function:


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

This tells you that tidyverse is loading the ggplot2, tibble, tidyr, readr, purrr, and dplyr packages.
These are considered to be the **core** of the tidyverse because you'll use them in almost every analysis.

Packages in the tidyverse change fairly frequently.
You can see if updates are available, and optionally install them, by running `tidyverse_update()`.

### Other packages

There are many other excellent packages that are not part of the tidyverse, because they solve problems in a different domain, or are designed with a different set of underlying principles.
This doesn't make them better or worse, just different.
In other words, the complement to the tidyverse is not the messyverse, but many other universes of interrelated packages.
As you tackle more data science projects with R, you'll learn new packages and new ways of thinking about data.

In this book we'll use three data packages from outside the tidyverse:


```r
install.packages(c("nycflights13", "gapminder", "Lahman"))
```

These packages provide data on airline flights, world development, and baseball that we'll use to illustrate key data science ideas.

## Running R code

The previous section showed you a couple of examples of running R code.
Code in the book looks like this:


```r
1 + 2
#> [1] 3
```

If you run the same code in your local console, it will look like this:

    > 1 + 2
    [1] 3

There are two main differences.
In your console, you type after the `>`, called the **prompt**; we don't show the prompt in the book.
In the book, output is commented out with `#>`; in your console it appears directly after your code.
These two differences mean that if you're working with an electronic version of the book, you can easily copy code out of the book and into the console.

Throughout the book we use a consistent set of conventions to refer to code:

-   Functions are in a code font and followed by parentheses, like `sum()`, or `mean()`.

-   Other R objects (like data or function arguments) are in a code font, without parentheses, like `flights` or `x`.

-   If we want to make it clear what package an object comes from, we'll use the package name followed by two colons, like `dplyr::mutate()`, or\
    `nycflights13::flights`. This is also valid R code.

## Getting help and learning more

This book is not an island; there is no single resource that will allow you to master R.
As you start to apply the techniques described in this book to your own data you will soon find questions that we do not answer.
This section describes a few tips on how to get help, and to help you keep learning.

If you get stuck, start with Google.
Typically adding "R" to a query is enough to restrict it to relevant results: if the search isn't useful, it often means that there aren't any R-specific results available.
Google is particularly useful for error messages.
If you get an error message and you have no idea what it means, try googling it!
Chances are that someone else has been confused by it in the past, and there will be help somewhere on the web.
(If the error message isn't in English, run `Sys.setenv(LANGUAGE = "en")` and re-run the code; you're more likely to find help for English error messages.)

If Google doesn't help, try [Stack Overflow](http://stackoverflow.com).
Start by spending a little time searching for an existing answer, including `[R]` to restrict your search to questions and answers that use R.
If you don't find anything useful, prepare a minimal reproducible example or **reprex**.
A good reprex makes it easier for other people to help you, and often you'll figure out the problem yourself in the course of making it.

There are three things you need to include to make your example reproducible: required packages, data, and code.

1.  **Packages** should be loaded at the top of the script, so it's easy to see which ones the example needs.
    This is a good time to check that you're using the latest version of each package; it's possible you've discovered a bug that's been fixed since you installed the package.
    For packages in the tidyverse, the easiest way to check is to run `tidyverse_update()`.

2.  The easiest way to include **data** in a question is to use `dput()` to generate the R code to recreate it.
    For example, to recreate the `mtcars` dataset in R, I'd perform the following steps:

    1.  Run `dput(mtcars)` in R
    2.  Copy the output
    3.  In my reproducible script, type `mtcars <-` then paste.

    Try and find the smallest subset of your data that still reveals the problem.

3.  Spend a little bit of time ensuring that your **code** is easy for others to read:

    -   Make sure you've used spaces and your variable names are concise, yet informative.

    -   Use comments to indicate where your problem lies.

    -   Do your best to remove everything that is not related to the problem.\
        The shorter your code is, the easier it is to understand, and the easier it is to fix.

Finish by checking that you have actually made a reproducible example by starting a fresh R session and copying and pasting your script in.

You should also spend some time preparing yourself to solve problems before they occur.
Investing a little time in learning R each day will pay off handsomely in the long run.
One way is to follow what Hadley, Garrett, and everyone else at RStudio are doing on the [RStudio blog](https://blog.rstudio.org).
This is where we post announcements about new packages, new IDE features, and in-person courses.
You might also want to follow Hadley ([\@hadleywickham](https://twitter.com/hadleywickham)) or Garrett ([\@statgarrett](https://twitter.com/statgarrett)) on Twitter, or follow [\@rstudiotips](https://twitter.com/rstudiotips) to keep up with new features in the IDE.

To keep up with the R community more broadly, we recommend reading <http://www.r-bloggers.com>: it aggregates over 500 blogs about R from around the world.
If you're an active Twitter user, follow the ([`#rstats`](https://twitter.com/search?q=%23rstats)) hashtag.
Twitter is one of the key tools that Hadley uses to keep up with new developments in the community.

## Acknowledgements

This book isn't just the product of Hadley and Garrett, but is the result of many conversations (in person and online) that we've had with the many people in the R community.
There are a few people we'd like to thank in particular, because they have spent many hours answering our questions and helping us to better think about data science:

-   Jenny Bryan and Lionel Henry for many helpful discussions around working with lists and list-columns.

-   The three chapters on workflow were adapted (with permission), from <http://stat545.com/block002_hello-r-workspace-wd-project.html> by Jenny Bryan.

-   Genevera Allen for discussions about models, modelling, the statistical learning perspective, and the difference between hypothesis generation and hypothesis confirmation.

-   Yihui Xie for his work on the [bookdown](https://github.com/rstudio/bookdown) package, and for tirelessly responding to my feature requests.

-   Bill Behrman for his thoughtful reading of the entire book, and for trying it out with his data science class at Stanford.

-   The \#rstats Twitter community who reviewed all of the draft chapters and provided tons of useful feedback.

-   Tal Galili for augmenting his dendextend package to support a section on clustering that did not make it into the final draft.

This book was written in the open, and many people contributed pull requests to fix minor problems.
Special thanks goes to everyone who contributed via GitHub:

Thanks go to all contributers in alphabetical order: @a-rosenberg, A. s, Abhinav Singh, adi pradhan, Ahmed ElGabbas, Ajay Deonarine, @AlanFeder, Albert Y. Kim, @Alex, Andrea Gilardi, Andrew Landgraf, @andrewmacfarland, Angela Li, Azza Ahmed, bahadir cankardes, @batpigandme, @behrman, Ben Herbertson, Ben Marwick, Ben Steinberg, Benjamin Yeh, Bianca Peterson, Bill Behrman, @BirgerNi, @boardtc, Brandon Greenwell, Brent Brewington, Brett Klamer, Brian G. Barkley, Charlotte Wickham, Christian G. Warden, Christian Heinrich, Christian Mongeau, Colin Gillespie, Cooper Morris, Curtis Alexander, Daniel Gromer, David Clark, David Rubinger, Derwin McGeary, Devin Pastoor, Dirk Eddelbuettel, @djbirke, @DSGeoff, Dylan Cashman, Earl Brown, Edwin Thoen, Eric Watt, Erik Erhardt, Etienne B. Racine, Everett Robinson, Flemming Villalona, Floris Vanderhaeghe, Garrick Aden-Buie, George Wang, Gregory Jefferis, Gustav W Delius, Hao Chen, @harrismcgehee, Hengni Cai, Hiroaki Yutani, Hojjat Salmasian, Ian Lyttle, Ian Sealy, Ivan Krukov, Jacek Kolacz, Jacob Kaplan, Jakub Nowosad, Jazz Weisman, Jeff Boichuk, Jeffrey Arnold, Jen Ren, Jennifer (Jenny) Bryan, @jennybc, Jeroen Janssens, Jim Hester, @jjchern, Joanne Jang, Johannes Gruber, John Blischak, John D. Storey, John Sears, Jon Calder, @Jonas, Jonathan Page, @jonathanflint, Jose Roberto Ayala Solares, Josh Goldberg, @juandering, Julia Stewart Lowndes, Julian During, Justinas Petuchovas, @kaetschap, Kara de la Marck, Kara Woo, Katrin Leinweber, @kdpsingh, Kenny Darrell, Kirill M??ller, Kirill Sevastyanenko, @koalabearski, Kunal Marwaha, @KyleHumphrey, Lawrence Wu, @lindbrook, Luke Smith, Luke W Johnston, Mara Averick, Maria Paula Caldas, Mark Beveridge, Matt Herman, Matthew Hendrickson, Matthew Sedaghatfar, @MattWittbrodt, Mauro Lepore, Michael Henry, Mine Cetinkaya-Rundel, @MJMarshall, Mustafa Ascha, @nate-d-olson, @nattalides, Nelson Areal, Nicholas Tierney, Nick Clark, @nickelas, Nina Munkholt Jakobsen, Nirmal Patel, Nischal Shrestha, Noah Landesberg, @nwaff, @OaCantona, Pablo E, Patrick Kennedy, @Paul, @pete, Peter Hurford, Rademeyer Vermaak, Radu Grosu, Ranae Dietzel, Riva Quiroga, @rlzijdeman, Rob Tenorio, Robert Schuessler, @robertchu03, Robin Gertenbach, @robinlovelace, @robinsones, Rohan Alexander, @RomeroBarata, S'busiso Mkhondwane, @Saghir, Sam Firke, @seamus-mckinsey, Seamus McKinsey, @seanpwilliams, Sebastian Kraus, Shannon Ellis, @shoili, @sibusiso16, @Sophiazj, @spirgel, St??phane Guillou, Steve Mortimer, @svenski, Tal Galili, Terence Teo, Thomas Klebel, Tim Waterhouse, TJ Mahr, Tom Prior, @twgardner2, Ulrik Lyngs, Will Beasley, @yahwes, Yihui Xie, Yiming (Paul) Li, Yu Yu Aung, Zach Bogart, @zeal626, Zhuoer Dong, @?????????.

## Colophon

An online version of this book is available at <http://r4ds.had.co.nz>.
It will continue to evolve in between reprints of the physical book.
The source of the book is available at <https://github.com/hadley/r4ds>.
The book is powered by <https://bookdown.org> which makes it easy to turn R markdown files into HTML, PDF, and EPUB.

This book was built with:


```r
sessioninfo::session_info(c("tidyverse"))
#> - Session info ---------------------------------------------------------------
#>  setting  value                         
#>  version  R version 4.1.1 (2021-08-10)  
#>  os       Windows 10 x64                
#>  system   x86_64, mingw32               
#>  ui       RTerm                         
#>  language (EN)                          
#>  collate  Chinese (Simplified)_China.936
#>  ctype    Chinese (Simplified)_China.936
#>  tz       Asia/Taipei                   
#>  date     2021-09-03                    
#> 
#> - Packages -------------------------------------------------------------------
#>  package       * version    date       lib source                            
#>  askpass         1.1        2019-01-13 [1] CRAN (R 4.1.0)                    
#>  assertthat      0.2.1      2019-03-21 [1] CRAN (R 4.1.0)                    
#>  backports       1.2.1      2020-12-09 [1] CRAN (R 4.1.0)                    
#>  base64enc       0.1-3      2015-07-28 [1] CRAN (R 4.1.0)                    
#>  bit             4.0.4      2020-08-04 [1] CRAN (R 4.1.0)                    
#>  bit64           4.0.5      2020-08-30 [1] CRAN (R 4.1.0)                    
#>  blob            1.2.2      2021-07-23 [1] CRAN (R 4.1.0)                    
#>  broom           0.7.9      2021-07-27 [1] CRAN (R 4.1.1)                    
#>  callr           3.7.0      2021-04-20 [1] CRAN (R 4.1.0)                    
#>  cellranger      1.1.0      2016-07-27 [1] CRAN (R 4.1.1)                    
#>  cli             3.0.1      2021-07-17 [1] CRAN (R 4.1.0)                    
#>  clipr           0.7.1      2020-10-08 [1] CRAN (R 4.1.0)                    
#>  colorspace      2.0-2      2021-06-24 [1] CRAN (R 4.1.0)                    
#>  cpp11           0.3.1      2021-06-25 [1] CRAN (R 4.1.0)                    
#>  crayon          1.4.1      2021-02-08 [1] CRAN (R 4.1.0)                    
#>  curl            4.3.2      2021-06-23 [1] CRAN (R 4.1.0)                    
#>  data.table      1.14.0     2021-02-21 [1] CRAN (R 4.1.0)                    
#>  DBI             1.1.1      2021-01-15 [1] CRAN (R 4.1.0)                    
#>  dbplyr          2.1.1      2021-04-06 [1] CRAN (R 4.1.0)                    
#>  digest          0.6.27     2020-10-24 [1] CRAN (R 4.1.0)                    
#>  dplyr         * 1.0.7      2021-06-18 [1] CRAN (R 4.1.0)                    
#>  dtplyr          1.1.0      2021-02-20 [1] CRAN (R 4.1.1)                    
#>  ellipsis        0.3.2      2021-04-29 [1] CRAN (R 4.1.0)                    
#>  evaluate        0.14       2019-05-28 [1] CRAN (R 4.1.0)                    
#>  fansi           0.5.0      2021-05-25 [1] CRAN (R 4.1.0)                    
#>  farver          2.1.0      2021-02-28 [1] CRAN (R 4.1.0)                    
#>  fastmap         1.1.0      2021-01-25 [1] CRAN (R 4.1.0)                    
#>  forcats       * 0.5.1      2021-01-27 [1] CRAN (R 4.1.1)                    
#>  fs              1.5.0      2020-07-31 [1] CRAN (R 4.1.0)                    
#>  gargle          1.2.0      2021-07-02 [1] CRAN (R 4.1.1)                    
#>  generics        0.1.0      2020-10-31 [1] CRAN (R 4.1.0)                    
#>  ggplot2       * 3.3.5      2021-06-25 [1] CRAN (R 4.1.0)                    
#>  glue            1.4.2      2020-08-27 [1] CRAN (R 4.1.0)                    
#>  googledrive     2.0.0      2021-07-08 [1] CRAN (R 4.1.1)                    
#>  googlesheets4   1.0.0      2021-07-21 [1] CRAN (R 4.1.1)                    
#>  gtable          0.3.0      2019-03-25 [1] CRAN (R 4.1.0)                    
#>  haven           2.4.3      2021-08-04 [1] CRAN (R 4.1.1)                    
#>  highr           0.9        2021-04-16 [1] CRAN (R 4.1.0)                    
#>  hms             1.1.0      2021-05-17 [1] CRAN (R 4.1.0)                    
#>  htmltools       0.5.2      2021-08-25 [1] CRAN (R 4.1.1)                    
#>  httr            1.4.2      2020-07-20 [1] CRAN (R 4.1.0)                    
#>  ids             1.0.1      2017-05-31 [1] CRAN (R 4.1.1)                    
#>  isoband         0.2.5      2021-07-13 [1] CRAN (R 4.1.0)                    
#>  jsonlite        1.7.2      2020-12-09 [1] CRAN (R 4.1.0)                    
#>  knitr           1.33       2021-04-24 [1] CRAN (R 4.1.0)                    
#>  labeling        0.4.2      2020-10-20 [1] CRAN (R 4.1.0)                    
#>  lattice         0.20-44    2021-05-02 [2] CRAN (R 4.1.1)                    
#>  lifecycle       1.0.0      2021-02-15 [1] CRAN (R 4.1.0)                    
#>  lubridate       1.7.10     2021-02-26 [1] CRAN (R 4.1.1)                    
#>  magrittr        2.0.1      2020-11-17 [1] CRAN (R 4.1.0)                    
#>  markdown        1.1        2019-08-07 [1] CRAN (R 4.1.0)                    
#>  MASS            7.3-54     2021-05-03 [2] CRAN (R 4.1.1)                    
#>  Matrix          1.3-4      2021-06-01 [2] CRAN (R 4.1.1)                    
#>  mgcv            1.8-36     2021-06-01 [2] CRAN (R 4.1.1)                    
#>  mime            0.11       2021-06-23 [1] CRAN (R 4.1.0)                    
#>  modelr          0.1.8      2020-05-19 [1] CRAN (R 4.1.1)                    
#>  munsell         0.5.0      2018-06-12 [1] CRAN (R 4.1.0)                    
#>  nlme            3.1-152    2021-02-04 [2] CRAN (R 4.1.1)                    
#>  openssl         1.4.4      2021-04-30 [1] CRAN (R 4.1.0)                    
#>  pillar          1.6.2      2021-07-29 [1] CRAN (R 4.1.0)                    
#>  pkgconfig       2.0.3      2019-09-22 [1] CRAN (R 4.1.0)                    
#>  prettyunits     1.1.1      2020-01-24 [1] CRAN (R 4.1.0)                    
#>  processx        3.5.2      2021-04-30 [1] CRAN (R 4.1.0)                    
#>  progress        1.2.2      2019-05-16 [1] CRAN (R 4.1.0)                    
#>  ps              1.6.0      2021-02-28 [1] CRAN (R 4.1.0)                    
#>  purrr         * 0.3.4      2020-04-17 [1] CRAN (R 4.1.0)                    
#>  R6              2.5.1      2021-08-19 [1] CRAN (R 4.1.1)                    
#>  rappdirs        0.3.3      2021-01-31 [1] CRAN (R 4.1.0)                    
#>  RColorBrewer    1.1-2      2014-12-07 [1] CRAN (R 4.1.0)                    
#>  Rcpp            1.0.7      2021-07-07 [1] CRAN (R 4.1.0)                    
#>  readr         * 2.0.1      2021-08-10 [1] CRAN (R 4.1.1)                    
#>  readxl          1.3.1      2019-03-13 [1] CRAN (R 4.1.1)                    
#>  rematch         1.0.1      2016-04-21 [1] CRAN (R 4.1.1)                    
#>  rematch2        2.1.2      2020-05-01 [1] CRAN (R 4.1.0)                    
#>  reprex          2.0.1      2021-08-05 [1] CRAN (R 4.1.1)                    
#>  rlang           0.4.11     2021-04-30 [1] CRAN (R 4.1.0)                    
#>  rmarkdown       2.10       2021-08-06 [1] CRAN (R 4.1.1)                    
#>  rstudioapi      0.13       2020-11-12 [1] CRAN (R 4.1.0)                    
#>  rvest           1.0.1      2021-07-26 [1] CRAN (R 4.1.0)                    
#>  scales          1.1.1      2020-05-11 [1] CRAN (R 4.1.0)                    
#>  selectr         0.4-2      2019-11-20 [1] CRAN (R 4.1.1)                    
#>  stringi         1.7.4      2021-08-25 [1] CRAN (R 4.1.1)                    
#>  stringr       * 1.4.0.9000 2021-09-02 [1] Github (tidyverse/stringr@6670a37)
#>  sys             3.4        2020-07-23 [1] CRAN (R 4.1.0)                    
#>  tibble        * 3.1.2      2021-05-16 [1] CRAN (R 4.1.0)                    
#>  tidyr         * 1.1.3      2021-03-03 [1] CRAN (R 4.1.0)                    
#>  tidyselect      1.1.1      2021-04-30 [1] CRAN (R 4.1.0)                    
#>  tidyverse     * 1.3.1      2021-04-15 [1] CRAN (R 4.1.1)                    
#>  tinytex         0.33       2021-08-05 [1] CRAN (R 4.1.1)                    
#>  tzdb            0.1.2      2021-07-20 [1] CRAN (R 4.1.1)                    
#>  utf8            1.2.1      2021-03-12 [1] CRAN (R 4.1.0)                    
#>  uuid            0.1-4      2020-02-26 [1] CRAN (R 4.1.0)                    
#>  vctrs           0.3.8      2021-04-29 [1] CRAN (R 4.1.0)                    
#>  viridisLite     0.4.0      2021-04-13 [1] CRAN (R 4.1.0)                    
#>  vroom           1.5.4      2021-08-05 [1] CRAN (R 4.1.1)                    
#>  withr           2.4.2      2021-04-18 [1] CRAN (R 4.1.0)                    
#>  xfun            0.25       2021-08-06 [1] CRAN (R 4.1.0)                    
#>  xml2            1.3.2      2020-04-23 [1] CRAN (R 4.1.0)                    
#>  yaml            2.2.1      2020-02-01 [1] CRAN (R 4.1.0)                    
#> 
#> [1] C:/Users/yangm/OneDrive/Documents/R/win-library/4.1
#> [2] C:/Program Files/R/R-4.1.1/library
```
