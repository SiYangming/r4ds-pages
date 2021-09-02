# Relational data

## Introduction

It's rare that a data analysis involves only a single data frame.
Typically you have many data frames, and you must combine them to answer the questions that you're interested in.
Collectively, multiple data frames are called **relational data** because it is the relations, not just the individual datasets, that are important.

Relations are always defined between a pair of data frames.
All other relations are built up from this simple idea: the relations of three or more data frames are always a property of the relations between each pair.
Sometimes both elements of a pair can be the same data frame!
This is needed if, for example, you have a data frame of people, and each person has a reference to their parents.

To work with relational data you need verbs that work with pairs of data frames.
There are three families of verbs designed to work with relational data:

-   **Mutating joins**, which add new variables to one data frame from matching observations in another.

-   **Filtering joins**, which filter observations from one data frame based on whether or not they match an observation in the other data frame.

-   **Set operations**, which treat observations as if they were set elements.

The most common place to find relational data is in a *relational* database management system (or RDBMS), a term that encompasses almost all modern databases.
If you've used a database before, you've almost certainly used SQL.
If so, you should find the concepts in this chapter familiar, although their expression in dplyr is a little different.
One other major terminology difference between databases and R is that what we generally refer to as data frames in R while the same concept is referred to as "table" in databases.
Hence you'll see references to one-table and two-table verbs in dplyr documentation.
Generally, dplyr is a little easier to use than SQL because dplyr is specialised to do data analysis: it makes common data analysis operations easier, at the expense of making it more difficult to do other things that aren't commonly needed for data analysis.

### Prerequisites

We will explore relational data from `nycflights13` using the two-table verbs from dplyr.


```r
library(tidyverse)
library(nycflights13)
```

## nycflights13 {#nycflights13-relational}

We will use the nycflights13 package to learn about relational data.
nycflights13 contains five tibbles : `airlines`, `airports`, `weather` and `planes` which are all related to the `flights` data frame that you used in Chapter \@ref(data-transform) on data transformation:

-   `airlines` lets you look up the full carrier name from its abbreviated code:

    
    ```r
    airlines
    #> [90m# A tibble: 16 x 2[39m
    #>   [1mcarrier[22m [1mname[22m                    
    #>   [3m[90m<chr>[39m[23m   [3m[90m<chr>[39m[23m                   
    #> [90m1[39m 9E      Endeavor Air Inc.       
    #> [90m2[39m AA      American Airlines Inc.  
    #> [90m3[39m AS      Alaska Airlines Inc.    
    #> [90m4[39m B6      JetBlue Airways         
    #> [90m5[39m DL      Delta Air Lines Inc.    
    #> [90m6[39m EV      ExpressJet Airlines Inc.
    #> [90m# ... with 10 more rows[39m
    ```

-   `airports` gives information about each airport, identified by the `faa` airport code:

    
    ```r
    airports
    #> [90m# A tibble: 1,458 x 8[39m
    #>   [1mfaa[22m   [1mname[22m                             [1mlat[22m   [1mlon[22m   [1malt[22m    [1mtz[22m [1mdst[22m   [1mtzone[22m      
    #>   [3m[90m<chr>[39m[23m [3m[90m<chr>[39m[23m                          [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m [3m[90m<chr>[39m[23m [3m[90m<chr>[39m[23m      
    #> [90m1[39m 04G   Lansdowne Airport               41.1 -[31m80[39m[31m.[39m[31m6[39m  [4m1[24m044    -[31m5[39m A     America/Ne~
    #> [90m2[39m 06A   Moton Field Municipal Airport   32.5 -[31m85[39m[31m.[39m[31m7[39m   264    -[31m6[39m A     America/Ch~
    #> [90m3[39m 06C   Schaumburg Regional             42.0 -[31m88[39m[31m.[39m[31m1[39m   801    -[31m6[39m A     America/Ch~
    #> [90m4[39m 06N   Randall Airport                 41.4 -[31m74[39m[31m.[39m[31m4[39m   523    -[31m5[39m A     America/Ne~
    #> [90m5[39m 09J   Jekyll Island Airport           31.1 -[31m81[39m[31m.[39m[31m4[39m    11    -[31m5[39m A     America/Ne~
    #> [90m6[39m 0A9   Elizabethton Municipal Airport  36.4 -[31m82[39m[31m.[39m[31m2[39m  [4m1[24m593    -[31m5[39m A     America/Ne~
    #> [90m# ... with 1,452 more rows[39m
    ```

-   `planes` gives information about each plane, identified by its `tailnum`:

    
    ```r
    planes
    #> [90m# A tibble: 3,322 x 9[39m
    #>   [1mtailnum[22m  [1myear[22m [1mtype[22m           [1mmanufacturer[22m   [1mmodel[22m  [1mengines[22m [1mseats[22m [1mspeed[22m [1mengine[22m 
    #>   [3m[90m<chr>[39m[23m   [3m[90m<int>[39m[23m [3m[90m<chr>[39m[23m          [3m[90m<chr>[39m[23m          [3m[90m<chr>[39m[23m    [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<chr>[39m[23m  
    #> [90m1[39m N10156   [4m2[24m004 Fixed wing mu~ EMBRAER        EMB-1~       2    55    [31mNA[39m Turbo-~
    #> [90m2[39m N102UW   [4m1[24m998 Fixed wing mu~ AIRBUS INDUST~ A320-~       2   182    [31mNA[39m Turbo-~
    #> [90m3[39m N103US   [4m1[24m999 Fixed wing mu~ AIRBUS INDUST~ A320-~       2   182    [31mNA[39m Turbo-~
    #> [90m4[39m N104UW   [4m1[24m999 Fixed wing mu~ AIRBUS INDUST~ A320-~       2   182    [31mNA[39m Turbo-~
    #> [90m5[39m N10575   [4m2[24m002 Fixed wing mu~ EMBRAER        EMB-1~       2    55    [31mNA[39m Turbo-~
    #> [90m6[39m N105UW   [4m1[24m999 Fixed wing mu~ AIRBUS INDUST~ A320-~       2   182    [31mNA[39m Turbo-~
    #> [90m# ... with 3,316 more rows[39m
    ```

-   `weather` gives the weather at each NYC airport for each hour:

    
    ```r
    weather
    #> [90m# A tibble: 26,115 x 15[39m
    #>   [1morigin[22m  [1myear[22m [1mmonth[22m   [1mday[22m  [1mhour[22m  [1mtemp[22m  [1mdewp[22m [1mhumid[22m [1mwind_dir[22m [1mwind_speed[22m [1mwind_gust[22m
    #>   [3m[90m<chr>[39m[23m  [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m    [3m[90m<dbl>[39m[23m      [3m[90m<dbl>[39m[23m     [3m[90m<dbl>[39m[23m
    #> [90m1[39m EWR     [4m2[24m013     1     1     1  39.0  26.1  59.4      270      10.4         [31mNA[39m
    #> [90m2[39m EWR     [4m2[24m013     1     1     2  39.0  27.0  61.6      250       8.06        [31mNA[39m
    #> [90m3[39m EWR     [4m2[24m013     1     1     3  39.0  28.0  64.4      240      11.5         [31mNA[39m
    #> [90m4[39m EWR     [4m2[24m013     1     1     4  39.9  28.0  62.2      250      12.7         [31mNA[39m
    #> [90m5[39m EWR     [4m2[24m013     1     1     5  39.0  28.0  64.4      260      12.7         [31mNA[39m
    #> [90m6[39m EWR     [4m2[24m013     1     1     6  37.9  28.0  67.2      240      11.5         [31mNA[39m
    #> [90m# ... with 26,109 more rows, and 4 more variables: [1mprecip[22m <dbl>,[39m
    #> [90m#   [1mpressure[22m <dbl>, [1mvisib[22m <dbl>, [1mtime_hour[22m <dttm>[39m
    ```

One way to show the relationships between the different data frames is with a diagram:

<img src="diagrams/relational-nycflights.png" title="Diagram of the relationships between airports, planes, flights, weather, and airlines datasets from the nycflights13 package. The faa variable in the airports data frame is connected to the origin and dest variables in the flights data frame. The tailnum variable in the planes data frame is connected to the tailnum variable in flights. The year, month, day, hour, and origin variables are connected to the variables with the same name in the flights data frame. And finally the carrier variables in the airlines data frame is connected to the carrier variable in the flights data frame. There are no direct connections between airports, planes, airlines, and weather data frames." alt="Diagram of the relationships between airports, planes, flights, weather, and airlines datasets from the nycflights13 package. The faa variable in the airports data frame is connected to the origin and dest variables in the flights data frame. The tailnum variable in the planes data frame is connected to the tailnum variable in flights. The year, month, day, hour, and origin variables are connected to the variables with the same name in the flights data frame. And finally the carrier variables in the airlines data frame is connected to the carrier variable in the flights data frame. There are no direct connections between airports, planes, airlines, and weather data frames." width="70%" style="display: block; margin: auto;" />

This diagram is a little overwhelming, but it's simple compared to some you'll see in the wild!
The key to understanding diagrams like this is to remember each relation always concerns a pair of data frames.
You don't need to understand the whole thing; you just need to understand the chain of relations between the data frames that you are interested in.

For nycflights13:

-   `flights` connects to `planes` via a single variable, `tailnum`.

-   `flights` connects to `airlines` through the `carrier` variable.

-   `flights` connects to `airports` in two ways: via the `origin` and `dest` variables.

-   `flights` connects to `weather` via `origin` (the location), and `year`, `month`, `day` and `hour` (the time).

### Exercises

1.  Imagine you wanted to draw (approximately) the route each plane flies from its origin to its destination.
    What variables would you need?
    What data frames would you need to combine?

2.  I forgot to draw the relationship between `weather` and `airports`.
    What is the relationship and how should it appear in the diagram?

3.  `weather` only contains information for the origin (NYC) airports.
    If it contained weather records for all airports in the USA, what additional relation would it define with `flights`?

## Keys

The variables used to connect each pair of data frames are called **keys**.
A key is a variable (or set of variables) that uniquely identifies an observation.
In simple cases, a single variable is sufficient to identify an observation.
For example, each plane is uniquely identified by its `tailnum`.
In other cases, multiple variables may be needed.
For example, to identify an observation in `weather` you need five variables: `year`, `month`, `day`, `hour`, and `origin`.

There are two types of keys:

-   A **primary key** uniquely identifies an observation in its own data frame.
    For example, `planes$tailnum` is a primary key because it uniquely identifies each plane in the `planes` data frame.

-   A **foreign key** uniquely identifies an observation in another data frame.
    For example, `flights$tailnum` is a foreign key because it appears in the `flights` data frame where it matches each flight to a unique plane.

A variable can be both a primary key *and* a foreign key.
For example, `origin` is part of the `weather` primary key, and is also a foreign key for the `airports` data frame.

Once you've identified the primary keys in your data frames, it's good practice to verify that they do indeed uniquely identify each observation.
One way to do that is to `count()` the primary keys and look for entries where `n` is greater than one:


```r
planes %>% 
  count(tailnum) %>% 
  filter(n > 1)
#> [90m# A tibble: 0 x 2[39m
#> [90m# ... with 2 variables: [1mtailnum[22m <chr>, [1mn[22m <int>[39m

weather %>% 
  count(year, month, day, hour, origin) %>% 
  filter(n > 1)
#> [90m# A tibble: 3 x 6[39m
#>    [1myear[22m [1mmonth[22m   [1mday[22m  [1mhour[22m [1morigin[22m     [1mn[22m
#>   [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<chr>[39m[23m  [3m[90m<int>[39m[23m
#> [90m1[39m  [4m2[24m013    11     3     1 EWR        2
#> [90m2[39m  [4m2[24m013    11     3     1 JFK        2
#> [90m3[39m  [4m2[24m013    11     3     1 LGA        2
```

Sometimes a data frame doesn't have an explicit primary key: each row is an observation, but no combination of variables reliably identifies it.
For example, what's the primary key in the `flights` data frame?
You might think it would be the date plus the flight or tail number, but neither of those are unique:


```r
flights %>% 
  count(year, month, day, flight) %>% 
  filter(n > 1)
#> [90m# A tibble: 29,768 x 5[39m
#>    [1myear[22m [1mmonth[22m   [1mday[22m [1mflight[22m     [1mn[22m
#>   [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m  [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m
#> [90m1[39m  [4m2[24m013     1     1      1     2
#> [90m2[39m  [4m2[24m013     1     1      3     2
#> [90m3[39m  [4m2[24m013     1     1      4     2
#> [90m4[39m  [4m2[24m013     1     1     11     3
#> [90m5[39m  [4m2[24m013     1     1     15     2
#> [90m6[39m  [4m2[24m013     1     1     21     2
#> [90m# ... with 29,762 more rows[39m

flights %>% 
  count(year, month, day, tailnum) %>% 
  filter(n > 1)
#> [90m# A tibble: 64,928 x 5[39m
#>    [1myear[22m [1mmonth[22m   [1mday[22m [1mtailnum[22m     [1mn[22m
#>   [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<chr>[39m[23m   [3m[90m<int>[39m[23m
#> [90m1[39m  [4m2[24m013     1     1 N0EGMQ      2
#> [90m2[39m  [4m2[24m013     1     1 N11189      2
#> [90m3[39m  [4m2[24m013     1     1 N11536      2
#> [90m4[39m  [4m2[24m013     1     1 N11544      3
#> [90m5[39m  [4m2[24m013     1     1 N11551      2
#> [90m6[39m  [4m2[24m013     1     1 N12540      2
#> [90m# ... with 64,922 more rows[39m
```

When starting to work with this data, I had naively assumed that each flight number would be only used once per day: that would make it much easier to communicate problems with a specific flight.
Unfortunately that is not the case!
If a data frame lacks a primary key, it's sometimes useful to add one with `mutate()` and `row_number()`.
That makes it easier to match observations if you've done some filtering and want to check back in with the original data.
This is called a **surrogate key**.

A primary key and the corresponding foreign key in another data frame form a **relation**.
Relations are typically one-to-many.
For example, each flight has one plane, but each plane has many flights.
In other data, you'll occasionally see a 1-to-1 relationship.
You can think of this as a special case of 1-to-many.
You can model many-to-many relations with a many-to-1 relation plus a 1-to-many relation.
For example, in this data there's a many-to-many relationship between airlines and airports: each airline flies to many airports; each airport hosts many airlines.

### Exercises

1.  Add a surrogate key to `flights`.

2.  We know that some days of the year are "special", and fewer people than usual fly on them.
    How might you represent that data as a data frame?
    What would be the primary keys of that data frame?
    How would it connect to the existing data frames?

3.  Identify the keys in the following datasets

    a.  `Lahman::Batting`
    b.  `babynames::babynames`
    c.  `nasaweather::atmos`
    d.  `fueleconomy::vehicles`
    e.  `ggplot2::diamonds`

    (You might need to install some packages and read some documentation.)

4.  Draw a diagram illustrating the connections between the `Batting`, `People`, and `Salaries` data frames in the Lahman package.
    Draw another diagram that shows the relationship between `People`, `Managers`, `AwardsManagers`.

    How would you characterise the relationship between the `Batting`, `Pitching`, and `Fielding` data frames?

## Mutating joins {#mutating-joins}

The first tool we'll look at for combining a pair of data frames is the **mutating join**.
A mutating join allows you to combine variables from two data frames.
It first matches observations by their keys, then copies across variables from one data frame to the other.

Like `mutate()`, the join functions add variables to the right, so if you have a lot of variables already, the new variables won't get printed out.
For these examples, we'll make it easier to see what's going on in the examples by creating a narrower dataset:


```r
flights2 <- flights %>% 
  select(year:day, hour, origin, dest, tailnum, carrier)
flights2
#> [90m# A tibble: 336,776 x 8[39m
#>    [1myear[22m [1mmonth[22m   [1mday[22m  [1mhour[22m [1morigin[22m [1mdest[22m  [1mtailnum[22m [1mcarrier[22m
#>   [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<dbl>[39m[23m [3m[90m<chr>[39m[23m  [3m[90m<chr>[39m[23m [3m[90m<chr>[39m[23m   [3m[90m<chr>[39m[23m  
#> [90m1[39m  [4m2[24m013     1     1     5 EWR    IAH   N14228  UA     
#> [90m2[39m  [4m2[24m013     1     1     5 LGA    IAH   N24211  UA     
#> [90m3[39m  [4m2[24m013     1     1     5 JFK    MIA   N619AA  AA     
#> [90m4[39m  [4m2[24m013     1     1     5 JFK    BQN   N804JB  B6     
#> [90m5[39m  [4m2[24m013     1     1     6 LGA    ATL   N668DN  DL     
#> [90m6[39m  [4m2[24m013     1     1     5 EWR    ORD   N39463  UA     
#> [90m# ... with 336,770 more rows[39m
```

(Remember, when you're in RStudio, you can also use `View()` to avoid this problem.)

Imagine you want to add the full airline name to the `flights2` data.
You can combine the `airlines` and `flights2` data frames with `left_join()`:


```r
flights2 %>%
  select(-origin, -dest) %>% 
  left_join(airlines, by = "carrier")
#> [90m# A tibble: 336,776 x 7[39m
#>    [1myear[22m [1mmonth[22m   [1mday[22m  [1mhour[22m [1mtailnum[22m [1mcarrier[22m [1mname[22m                  
#>   [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<dbl>[39m[23m [3m[90m<chr>[39m[23m   [3m[90m<chr>[39m[23m   [3m[90m<chr>[39m[23m                 
#> [90m1[39m  [4m2[24m013     1     1     5 N14228  UA      United Air Lines Inc. 
#> [90m2[39m  [4m2[24m013     1     1     5 N24211  UA      United Air Lines Inc. 
#> [90m3[39m  [4m2[24m013     1     1     5 N619AA  AA      American Airlines Inc.
#> [90m4[39m  [4m2[24m013     1     1     5 N804JB  B6      JetBlue Airways       
#> [90m5[39m  [4m2[24m013     1     1     6 N668DN  DL      Delta Air Lines Inc.  
#> [90m6[39m  [4m2[24m013     1     1     5 N39463  UA      United Air Lines Inc. 
#> [90m# ... with 336,770 more rows[39m
```

The result of joining airlines to flights2 is an additional variable: `name`.
This is why I call this type of join a mutating join.
In this case, you could have got to the same place using `mutate()` and R's base subsetting:


```r
flights2 %>%
  select(-origin, -dest) %>% 
  mutate(name = airlines$name[match(carrier, airlines$carrier)])
#> [90m# A tibble: 336,776 x 7[39m
#>    [1myear[22m [1mmonth[22m   [1mday[22m  [1mhour[22m [1mtailnum[22m [1mcarrier[22m [1mname[22m                  
#>   [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<dbl>[39m[23m [3m[90m<chr>[39m[23m   [3m[90m<chr>[39m[23m   [3m[90m<chr>[39m[23m                 
#> [90m1[39m  [4m2[24m013     1     1     5 N14228  UA      United Air Lines Inc. 
#> [90m2[39m  [4m2[24m013     1     1     5 N24211  UA      United Air Lines Inc. 
#> [90m3[39m  [4m2[24m013     1     1     5 N619AA  AA      American Airlines Inc.
#> [90m4[39m  [4m2[24m013     1     1     5 N804JB  B6      JetBlue Airways       
#> [90m5[39m  [4m2[24m013     1     1     6 N668DN  DL      Delta Air Lines Inc.  
#> [90m6[39m  [4m2[24m013     1     1     5 N39463  UA      United Air Lines Inc. 
#> [90m# ... with 336,770 more rows[39m
```

But this is hard to generalise when you need to match multiple variables, and takes close reading to figure out the overall intent.

The following sections explain, in detail, how mutating joins work.
You'll start by learning a useful visual representation of joins.
We'll then use that to explain the four mutating join functions: the inner join, and the three outer joins.
When working with real data, keys don't always uniquely identify observations, so next we'll talk about what happens when there isn't a unique match.
Finally, you'll learn how to tell dplyr which variables are the keys for a given join.

### Understanding joins

To help you learn how joins work, I'm going to use a visual representation:

<img src="diagrams/join-setup.png" title="x and y are two data frames with 2 columns and 3 rows each. The first column in each is the key and the second is the value. The contents of these data frames are given in the subsequent code chunk." alt="x and y are two data frames with 2 columns and 3 rows each. The first column in each is the key and the second is the value. The contents of these data frames are given in the subsequent code chunk." width="118" style="display: block; margin: auto;" />


```r
x <- tribble(
  ~key, ~val_x,
     1, "x1",
     2, "x2",
     3, "x3"
)
y <- tribble(
  ~key, ~val_y,
     1, "y1",
     2, "y2",
     4, "y3"
)
```

The coloured column represents the "key" variable: these are used to match the rows between the data frames.
The grey column represents the "value" column that is carried along for the ride.
In these examples I'll show a single key variable, but the idea generalises in a straightforward way to multiple keys and multiple values.

A join is a way of connecting each row in `x` to zero, one, or more rows in `y`.
The following diagram shows each potential match as an intersection of a pair of lines.

<img src="diagrams/join-setup2.png" title="x and y data frames placed next to each other. with the key variable moved up front in y so that the key variable in x and key variable in y appear next to each other." alt="x and y data frames placed next to each other. with the key variable moved up front in y so that the key variable in x and key variable in y appear next to each other." width="166" style="display: block; margin: auto;" />

(If you look closely, you might notice that we've switched the order of the key and value columns in `x`. This is to emphasise that joins match based on the key; the value is just carried along for the ride.)

In an actual join, matches will be indicated with dots.
The number of dots = the number of matches = the number of rows in the output.

<img src="diagrams/join-inner.png" title="Keys 1 and 2 in x and y data frames are matched and indicated with lines joining these rows with dot in the middle. Hence, there are two dots in this diagram. The resulting joined data frame has two rows and 3 columns: key, val_x, and val_y. Values in the key column are 1 and 2 (the matched values)." alt="Keys 1 and 2 in x and y data frames are matched and indicated with lines joining these rows with dot in the middle. Hence, there are two dots in this diagram. The resulting joined data frame has two rows and 3 columns: key, val_x, and val_y. Values in the key column are 1 and 2 (the matched values)." width="338" style="display: block; margin: auto;" />

### Inner join {#inner-join}

The simplest type of join is the **inner join**.
An inner join matches pairs of observations whenever their keys are equal:

<img src="diagrams/join-inner.png" title="Keys 1 and 2 in x and y data frames are matched and indicated with lines joining these rows with dot in the middle. Hence, there are two dots in this diagram. The resulting joined data frame has two rows and 3 columns: key, val_x, and val_y. Values in the key column are 1 and 2 (the matched values)." alt="Keys 1 and 2 in x and y data frames are matched and indicated with lines joining these rows with dot in the middle. Hence, there are two dots in this diagram. The resulting joined data frame has two rows and 3 columns: key, val_x, and val_y. Values in the key column are 1 and 2 (the matched values)." width="338" style="display: block; margin: auto;" />

(To be precise, this is an inner **equijoin** because the keys are matched using the equality operator. Since most joins are equijoins we usually drop that specification.)

The output of an inner join is a new data frame that contains the key, the x values, and the y values.
We use `by` to tell dplyr which variable is the key:


```r
x %>% 
  inner_join(y, by = "key")
#> [90m# A tibble: 2 x 3[39m
#>     [1mkey[22m [1mval_x[22m [1mval_y[22m
#>   [3m[90m<dbl>[39m[23m [3m[90m<chr>[39m[23m [3m[90m<chr>[39m[23m
#> [90m1[39m     1 x1    y1   
#> [90m2[39m     2 x2    y2
```

The most important property of an inner join is that unmatched rows are not included in the result.
This means that generally inner joins are usually not appropriate for use in analysis because it's too easy to lose observations.

### Outer joins {#outer-join}

An inner join keeps observations that appear in both data frames.
An **outer join** keeps observations that appear in at least one of the data frames.
There are three types of outer joins:

-   A **left join** keeps all observations in `x`.
-   A **right join** keeps all observations in `y`.
-   A **full join** keeps all observations in `x` and `y`.

These joins work by adding an additional "virtual" observation to each data frame.
This observation has a key that always matches (if no other key matches), and a value filled with `NA`.

Graphically, that looks like:

<img src="diagrams/join-outer.png" title="Three diagrams for left, right, and full joins. In each diagram data frame x is on the left and y is on the right. The result of the join is always a data frame with three columns (key, val_x, and val_y). Left join: keys 1 and 2 from x are matched to those in y, key 3 is also carried along to the joined result since it's on the left data frame, but key 4 from y is not carried along since it's on the right but not on the left. The result is a data frame with 3 rows: keys 1, 2, and 3, all values from val_x, and the corresponding values from val_y for keys 1 and 2 with an NA for key 3, val_y. Right join: keys 1 and 2 from x are matched to those in y, key 4 is also carried along to the joined result since it's on the right data frame, but key 3 from x is not carried along since it's on the left but not on the right. The result is a data frame with 3 rows: keys 1, 2, and 4, all values from val_y, and the corresponding values from val_x for keys 1 and 2 with an NA for key 4, val_x. Full join: The resulting data frame has 4 rows: keys 1, 2, 3, and 4 with all values from val_x and val_y, however key 2, val_y and key 4, val_x are NAs since those keys aren't present in their respective data frames." alt="Three diagrams for left, right, and full joins. In each diagram data frame x is on the left and y is on the right. The result of the join is always a data frame with three columns (key, val_x, and val_y). Left join: keys 1 and 2 from x are matched to those in y, key 3 is also carried along to the joined result since it's on the left data frame, but key 4 from y is not carried along since it's on the right but not on the left. The result is a data frame with 3 rows: keys 1, 2, and 3, all values from val_x, and the corresponding values from val_y for keys 1 and 2 with an NA for key 3, val_y. Right join: keys 1 and 2 from x are matched to those in y, key 4 is also carried along to the joined result since it's on the right data frame, but key 3 from x is not carried along since it's on the left but not on the right. The result is a data frame with 3 rows: keys 1, 2, and 4, all values from val_y, and the corresponding values from val_x for keys 1 and 2 with an NA for key 4, val_x. Full join: The resulting data frame has 4 rows: keys 1, 2, 3, and 4 with all values from val_x and val_y, however key 2, val_y and key 4, val_x are NAs since those keys aren't present in their respective data frames." width="355" style="display: block; margin: auto;" />

The most commonly used join is the left join: you use this whenever you look up additional data from another data frame, because it preserves the original observations even when there isn't a match.
The left join should be your default join: use it unless you have a strong reason to prefer one of the others.

Another way to depict the different types of joins is with a Venn diagram:

<img src="diagrams/join-venn.png" title="Venn diagrams for inner, full, left, and right joins. Each join represented with two intersecting circles representing data frames x and y, with x on the right and y on the left. Shading indicates the result of the join. Inner join: Only intersection is shaded. Full join: Everything is shaded. Left join: Only x is shaded, but not the area in y that doesn't intersect with x. Right join: Only y is shaded, but not the area in x that doesn't intersect with y." alt="Venn diagrams for inner, full, left, and right joins. Each join represented with two intersecting circles representing data frames x and y, with x on the right and y on the left. Shading indicates the result of the join. Inner join: Only intersection is shaded. Full join: Everything is shaded. Left join: Only x is shaded, but not the area in y that doesn't intersect with x. Right join: Only y is shaded, but not the area in x that doesn't intersect with y." width="551" style="display: block; margin: auto;" />

However, this is not a great representation.
It might jog your memory about which join preserves the observations in which data frame, but it suffers from a major limitation: a Venn diagram can't show what happens when keys don't uniquely identify an observation.

### Duplicate keys {#join-matches}

So far all the diagrams have assumed that the keys are unique.
But that's not always the case.
This section explains what happens when the keys are not unique.
There are two possibilities:

1.  One data frame has duplicate keys.
    This is useful when you want to add in additional information as there is typically a one-to-many relationship.

    <img src="diagrams/join-one-to-many.png" title="Diagram describing a left join where one of the data frames (x) has duplicate keys. Data frame x is on the left, has 4 rows and 2 columns (key, val_x), and has the keys 1, 2, 2, and 1. Data frame y is on the right, has 2 rows and 2 columns (key, val_y), and has the keys 1 and 2. Left joining these two data frames yields a data frame with 4 rows (keys 1, 2, 2, and 1) and 3 columns (val_x, key, val_y). All values from x$val_x are carried along, values in y for key 1 and 2 are duplicated." alt="Diagram describing a left join where one of the data frames (x) has duplicate keys. Data frame x is on the left, has 4 rows and 2 columns (key, val_x), and has the keys 1, 2, 2, and 1. Data frame y is on the right, has 2 rows and 2 columns (key, val_y), and has the keys 1 and 2. Left joining these two data frames yields a data frame with 4 rows (keys 1, 2, 2, and 1) and 3 columns (val_x, key, val_y). All values from x$val_x are carried along, values in y for key 1 and 2 are duplicated." width="279" style="display: block; margin: auto;" />

    Note that I've put the key column in a slightly different position in the output.
    This reflects that the key is a primary key in `y` and a foreign key in `x`.

    
    ```r
    x <- tribble(
      ~key, ~val_x,
         1, "x1",
         2, "x2",
         2, "x3",
         1, "x4"
    )
    y <- tribble(
      ~key, ~val_y,
         1, "y1",
         2, "y2"
    )
    left_join(x, y, by = "key")
    #> [90m# A tibble: 4 x 3[39m
    #>     [1mkey[22m [1mval_x[22m [1mval_y[22m
    #>   [3m[90m<dbl>[39m[23m [3m[90m<chr>[39m[23m [3m[90m<chr>[39m[23m
    #> [90m1[39m     1 x1    y1   
    #> [90m2[39m     2 x2    y2   
    #> [90m3[39m     2 x3    y2   
    #> [90m4[39m     1 x4    y1
    ```

2.  Both data frames have duplicate keys.
    This is usually an error because in neither data frame do the keys uniquely identify an observation.
    When you join duplicated keys, you get all possible combinations, the Cartesian product:

    <img src="diagrams/join-many-to-many.png" title="Diagram describing a left join where both data frames (x and y) have duplicate keys. Data frame x is on the left, has 4 rows and 2 columns (key, val_x), and has the keys 1, 2, 2, and 3. Data frame y is on the right, has 4 rows and 2 columns (key, val_y), and has the keys 1, 2, 2, and 3 as well. Left joining these two data frames yields a data frame with 6 rows (keys 1, 2, 2, 2, 2, and 3) and 3 columns (key, val_x, val_y). All values from both datasets are included." alt="Diagram describing a left join where both data frames (x and y) have duplicate keys. Data frame x is on the left, has 4 rows and 2 columns (key, val_x), and has the keys 1, 2, 2, and 3. Data frame y is on the right, has 4 rows and 2 columns (key, val_y), and has the keys 1, 2, 2, and 3 as well. Left joining these two data frames yields a data frame with 6 rows (keys 1, 2, 2, 2, 2, and 3) and 3 columns (key, val_x, val_y). All values from both datasets are included." width="342" style="display: block; margin: auto;" />

    
    ```r
    x <- tribble(
      ~key, ~val_x,
         1, "x1",
         2, "x2",
         2, "x3",
         3, "x4"
    )
    y <- tribble(
      ~key, ~val_y,
         1, "y1",
         2, "y2",
         2, "y3",
         3, "y4"
    )
    left_join(x, y, by = "key")
    #> [90m# A tibble: 6 x 3[39m
    #>     [1mkey[22m [1mval_x[22m [1mval_y[22m
    #>   [3m[90m<dbl>[39m[23m [3m[90m<chr>[39m[23m [3m[90m<chr>[39m[23m
    #> [90m1[39m     1 x1    y1   
    #> [90m2[39m     2 x2    y2   
    #> [90m3[39m     2 x2    y3   
    #> [90m4[39m     2 x3    y2   
    #> [90m5[39m     2 x3    y3   
    #> [90m6[39m     3 x4    y4
    ```

### Defining the key columns {#join-by}

So far, the pairs of data frames have always been joined by a single variable, and that variable has the same name in both data frames.
That constraint was encoded by `by = "key"`.
You can use other values for `by` to connect the data frames in other ways:

-   The default, `by = NULL`, uses all variables that appear in both data frames, the so called **natural** join.
    For example, the flights and weather data frames match on their common variables: `year`, `month`, `day`, `hour` and `origin`.

    
    ```r
    flights2 %>% 
      left_join(weather)
    #> Joining, by = c("year", "month", "day", "hour", "origin")
    #> [90m# A tibble: 336,776 x 18[39m
    #>    [1myear[22m [1mmonth[22m   [1mday[22m  [1mhour[22m [1morigin[22m [1mdest[22m  [1mtailnum[22m [1mcarrier[22m  [1mtemp[22m  [1mdewp[22m [1mhumid[22m
    #>   [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<dbl>[39m[23m [3m[90m<chr>[39m[23m  [3m[90m<chr>[39m[23m [3m[90m<chr>[39m[23m   [3m[90m<chr>[39m[23m   [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m
    #> [90m1[39m  [4m2[24m013     1     1     5 EWR    IAH   N14228  UA       39.0  28.0  64.4
    #> [90m2[39m  [4m2[24m013     1     1     5 LGA    IAH   N24211  UA       39.9  25.0  54.8
    #> [90m3[39m  [4m2[24m013     1     1     5 JFK    MIA   N619AA  AA       39.0  27.0  61.6
    #> [90m4[39m  [4m2[24m013     1     1     5 JFK    BQN   N804JB  B6       39.0  27.0  61.6
    #> [90m5[39m  [4m2[24m013     1     1     6 LGA    ATL   N668DN  DL       39.9  25.0  54.8
    #> [90m6[39m  [4m2[24m013     1     1     5 EWR    ORD   N39463  UA       39.0  28.0  64.4
    #> [90m# ... with 336,770 more rows, and 7 more variables: [1mwind_dir[22m <dbl>,[39m
    #> [90m#   [1mwind_speed[22m <dbl>, [1mwind_gust[22m <dbl>, [1mprecip[22m <dbl>, [1mpressure[22m <dbl>,[39m
    #> [90m#   [1mvisib[22m <dbl>, [1mtime_hour[22m <dttm>[39m
    ```

-   A character vector, `by = "x"`.
    This is like a natural join, but uses only some of the common variables.
    For example, `flights` and `planes` have `year` variables, but they mean different things so we only want to join by `tailnum`.

    
    ```r
    flights2 %>% 
      left_join(planes, by = "tailnum")
    #> [90m# A tibble: 336,776 x 16[39m
    #>   [1myear.x[22m [1mmonth[22m   [1mday[22m  [1mhour[22m [1morigin[22m [1mdest[22m  [1mtailnum[22m [1mcarrier[22m [1myear.y[22m [1mtype[22m             
    #>    [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<dbl>[39m[23m [3m[90m<chr>[39m[23m  [3m[90m<chr>[39m[23m [3m[90m<chr>[39m[23m   [3m[90m<chr>[39m[23m    [3m[90m<int>[39m[23m [3m[90m<chr>[39m[23m            
    #> [90m1[39m   [4m2[24m013     1     1     5 EWR    IAH   N14228  UA        [4m1[24m999 Fixed wing multi~
    #> [90m2[39m   [4m2[24m013     1     1     5 LGA    IAH   N24211  UA        [4m1[24m998 Fixed wing multi~
    #> [90m3[39m   [4m2[24m013     1     1     5 JFK    MIA   N619AA  AA        [4m1[24m990 Fixed wing multi~
    #> [90m4[39m   [4m2[24m013     1     1     5 JFK    BQN   N804JB  B6        [4m2[24m012 Fixed wing multi~
    #> [90m5[39m   [4m2[24m013     1     1     6 LGA    ATL   N668DN  DL        [4m1[24m991 Fixed wing multi~
    #> [90m6[39m   [4m2[24m013     1     1     5 EWR    ORD   N39463  UA        [4m2[24m012 Fixed wing multi~
    #> [90m# ... with 336,770 more rows, and 6 more variables: [1mmanufacturer[22m <chr>,[39m
    #> [90m#   [1mmodel[22m <chr>, [1mengines[22m <int>, [1mseats[22m <int>, [1mspeed[22m <int>, [1mengine[22m <chr>[39m
    ```

    Note that the `year` variables (which appear in both input data frames, but are not constrained to be equal) are disambiguated in the output with a suffix.

-   A named character vector: `by = c("a" = "b")`.
    This will match variable `a` in data frame `x` to variable `b` in data frame `y`.
    The variables from `x` will be used in the output.

    For example, if we want to draw a map we need to combine the flights data with the airports data which contains the location (`lat` and `lon`) of each airport.
    Each flight has an origin and destination `airport`, so we need to specify which one we want to join to:

    
    ```r
    flights2 %>% 
      left_join(airports, c("dest" = "faa"))
    #> [90m# A tibble: 336,776 x 15[39m
    #>    [1myear[22m [1mmonth[22m   [1mday[22m  [1mhour[22m [1morigin[22m [1mdest[22m  [1mtailnum[22m [1mcarrier[22m [1mname[22m      [1mlat[22m   [1mlon[22m   [1malt[22m
    #>   [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<dbl>[39m[23m [3m[90m<chr>[39m[23m  [3m[90m<chr>[39m[23m [3m[90m<chr>[39m[23m   [3m[90m<chr>[39m[23m   [3m[90m<chr>[39m[23m   [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m
    #> [90m1[39m  [4m2[24m013     1     1     5 EWR    IAH   N14228  UA      George~  30.0 -[31m95[39m[31m.[39m[31m3[39m    97
    #> [90m2[39m  [4m2[24m013     1     1     5 LGA    IAH   N24211  UA      George~  30.0 -[31m95[39m[31m.[39m[31m3[39m    97
    #> [90m3[39m  [4m2[24m013     1     1     5 JFK    MIA   N619AA  AA      Miami ~  25.8 -[31m80[39m[31m.[39m[31m3[39m     8
    #> [90m4[39m  [4m2[24m013     1     1     5 JFK    BQN   N804JB  B6      [31mNA[39m       [31mNA[39m    [31mNA[39m      [31mNA[39m
    #> [90m5[39m  [4m2[24m013     1     1     6 LGA    ATL   N668DN  DL      Hartsf~  33.6 -[31m84[39m[31m.[39m[31m4[39m  [4m1[24m026
    #> [90m6[39m  [4m2[24m013     1     1     5 EWR    ORD   N39463  UA      Chicag~  42.0 -[31m87[39m[31m.[39m[31m9[39m   668
    #> [90m# ... with 336,770 more rows, and 3 more variables: [1mtz[22m <dbl>, [1mdst[22m <chr>,[39m
    #> [90m#   [1mtzone[22m <chr>[39m
    
    flights2 %>% 
      left_join(airports, c("origin" = "faa"))
    #> [90m# A tibble: 336,776 x 15[39m
    #>    [1myear[22m [1mmonth[22m   [1mday[22m  [1mhour[22m [1morigin[22m [1mdest[22m  [1mtailnum[22m [1mcarrier[22m [1mname[22m      [1mlat[22m   [1mlon[22m   [1malt[22m
    #>   [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<dbl>[39m[23m [3m[90m<chr>[39m[23m  [3m[90m<chr>[39m[23m [3m[90m<chr>[39m[23m   [3m[90m<chr>[39m[23m   [3m[90m<chr>[39m[23m   [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m
    #> [90m1[39m  [4m2[24m013     1     1     5 EWR    IAH   N14228  UA      Newark~  40.7 -[31m74[39m[31m.[39m[31m2[39m    18
    #> [90m2[39m  [4m2[24m013     1     1     5 LGA    IAH   N24211  UA      La Gua~  40.8 -[31m73[39m[31m.[39m[31m9[39m    22
    #> [90m3[39m  [4m2[24m013     1     1     5 JFK    MIA   N619AA  AA      John F~  40.6 -[31m73[39m[31m.[39m[31m8[39m    13
    #> [90m4[39m  [4m2[24m013     1     1     5 JFK    BQN   N804JB  B6      John F~  40.6 -[31m73[39m[31m.[39m[31m8[39m    13
    #> [90m5[39m  [4m2[24m013     1     1     6 LGA    ATL   N668DN  DL      La Gua~  40.8 -[31m73[39m[31m.[39m[31m9[39m    22
    #> [90m6[39m  [4m2[24m013     1     1     5 EWR    ORD   N39463  UA      Newark~  40.7 -[31m74[39m[31m.[39m[31m2[39m    18
    #> [90m# ... with 336,770 more rows, and 3 more variables: [1mtz[22m <dbl>, [1mdst[22m <chr>,[39m
    #> [90m#   [1mtzone[22m <chr>[39m
    ```

### Exercises

1.  Compute the average delay by destination, then join on the `airports` data frame so you can show the spatial distribution of delays.
    Here's an easy way to draw a map of the United States:

    
    ```r
    airports %>%
      semi_join(flights, c("faa" = "dest")) %>%
      ggplot(aes(lon, lat)) +
        borders("state") +
        geom_point() +
        coord_quickmap()
    ```

    (Don't worry if you don't understand what `semi_join()` does --- you'll learn about it next.)

    You might want to use the `size` or `colour` of the points to display the average delay for each airport.

2.  Add the location of the origin *and* destination (i.e. the `lat` and `lon`) to `flights`.

3.  Is there a relationship between the age of a plane and its delays?

4.  What weather conditions make it more likely to see a delay?

5.  What happened on June 13 2013?
    Display the spatial pattern of delays, and then use Google to cross-reference with the weather.

    

### Other implementations

`base::merge()` can perform all four types of mutating join:

| dplyr              | merge                                     |
|--------------------|-------------------------------------------|
| `inner_join(x, y)` | `merge(x, y)`                             |
| `left_join(x, y)`  | `merge(x, y, all.x = TRUE)`               |
| `right_join(x, y)` | `merge(x, y, all.y = TRUE)`,              |
| `full_join(x, y)`  | `merge(x, y, all.x = TRUE, all.y = TRUE)` |

The advantages of the specific dplyr verbs is that they more clearly convey the intent of your code: the difference between the joins is really important but concealed in the arguments of `merge()`.
dplyr's joins are considerably faster and don't mess with the order of the rows.

SQL is the inspiration for dplyr's conventions, so the translation is straightforward:

| dplyr                        | SQL                                            |
|------------------------------|------------------------------------------------|
| `inner_join(x, y, by = "z")` | `SELECT * FROM x INNER JOIN y USING (z)`       |
| `left_join(x, y, by = "z")`  | `SELECT * FROM x LEFT OUTER JOIN y USING (z)`  |
| `right_join(x, y, by = "z")` | `SELECT * FROM x RIGHT OUTER JOIN y USING (z)` |
| `full_join(x, y, by = "z")`  | `SELECT * FROM x FULL OUTER JOIN y USING (z)`  |

Note that "INNER" and "OUTER" are optional, and often omitted.

Joining different variables between the data frames, e.g. `inner_join(x, y, by = c("a" = "b"))` uses a slightly different syntax in SQL: `SELECT * FROM x INNER JOIN y ON x.a = y.b`.
As this syntax suggests, SQL supports a wider range of join types than dplyr because you can connect the data frames using constraints other than equality (sometimes called non-equijoins).

## Filtering joins {#filtering-joins}

Filtering joins match observations in the same way as mutating joins, but affect the observations, not the variables.
There are two types:

-   `semi_join(x, y)` **keeps** all observations in `x` that have a match in `y`.
-   `anti_join(x, y)` **drops** all observations in `x` that have a match in `y`.

Semi-joins are useful for matching filtered summary data frames back to the original rows.
For example, imagine you've found the top ten most popular destinations:


```r
top_dest <- flights %>%
  count(dest, sort = TRUE) %>%
  head(10)
top_dest
#> [90m# A tibble: 10 x 2[39m
#>   [1mdest[22m      [1mn[22m
#>   [3m[90m<chr>[39m[23m [3m[90m<int>[39m[23m
#> [90m1[39m ORD   [4m1[24m[4m7[24m283
#> [90m2[39m ATL   [4m1[24m[4m7[24m215
#> [90m3[39m LAX   [4m1[24m[4m6[24m174
#> [90m4[39m BOS   [4m1[24m[4m5[24m508
#> [90m5[39m MCO   [4m1[24m[4m4[24m082
#> [90m6[39m CLT   [4m1[24m[4m4[24m064
#> [90m# ... with 4 more rows[39m
```

Now you want to find each flight that went to one of those destinations.
You could construct a filter yourself:


```r
flights %>% 
  filter(dest %in% top_dest$dest)
#> [90m# A tibble: 141,145 x 19[39m
#>    [1myear[22m [1mmonth[22m   [1mday[22m [1mdep_time[22m [1msched_dep_time[22m [1mdep_delay[22m [1marr_time[22m [1msched_arr_time[22m
#>   [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m    [3m[90m<int>[39m[23m          [3m[90m<int>[39m[23m     [3m[90m<dbl>[39m[23m    [3m[90m<int>[39m[23m          [3m[90m<int>[39m[23m
#> [90m1[39m  [4m2[24m013     1     1      542            540         2      923            850
#> [90m2[39m  [4m2[24m013     1     1      554            600        -[31m6[39m      812            837
#> [90m3[39m  [4m2[24m013     1     1      554            558        -[31m4[39m      740            728
#> [90m4[39m  [4m2[24m013     1     1      555            600        -[31m5[39m      913            854
#> [90m5[39m  [4m2[24m013     1     1      557            600        -[31m3[39m      838            846
#> [90m6[39m  [4m2[24m013     1     1      558            600        -[31m2[39m      753            745
#> [90m# ... with 141,139 more rows, and 11 more variables: [1marr_delay[22m <dbl>,[39m
#> [90m#   [1mcarrier[22m <chr>, [1mflight[22m <int>, [1mtailnum[22m <chr>, [1morigin[22m <chr>, [1mdest[22m <chr>,[39m
#> [90m#   [1mair_time[22m <dbl>, [1mdistance[22m <dbl>, [1mhour[22m <dbl>, [1mminute[22m <dbl>, [1mtime_hour[22m <dttm>[39m
```

But it's difficult to extend that approach to multiple variables.
For example, imagine that you'd found the 10 days with highest average delays.
How would you construct the filter statement that used `year`, `month`, and `day` to match it back to `flights`?

Instead you can use a semi-join, which connects the two data frames like a mutating join, but instead of adding new columns, only keeps the rows in `x` that have a match in `y`:


```r
flights %>% 
  semi_join(top_dest)
#> Joining, by = "dest"
#> [90m# A tibble: 141,145 x 19[39m
#>    [1myear[22m [1mmonth[22m   [1mday[22m [1mdep_time[22m [1msched_dep_time[22m [1mdep_delay[22m [1marr_time[22m [1msched_arr_time[22m
#>   [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m [3m[90m<int>[39m[23m    [3m[90m<int>[39m[23m          [3m[90m<int>[39m[23m     [3m[90m<dbl>[39m[23m    [3m[90m<int>[39m[23m          [3m[90m<int>[39m[23m
#> [90m1[39m  [4m2[24m013     1     1      542            540         2      923            850
#> [90m2[39m  [4m2[24m013     1     1      554            600        -[31m6[39m      812            837
#> [90m3[39m  [4m2[24m013     1     1      554            558        -[31m4[39m      740            728
#> [90m4[39m  [4m2[24m013     1     1      555            600        -[31m5[39m      913            854
#> [90m5[39m  [4m2[24m013     1     1      557            600        -[31m3[39m      838            846
#> [90m6[39m  [4m2[24m013     1     1      558            600        -[31m2[39m      753            745
#> [90m# ... with 141,139 more rows, and 11 more variables: [1marr_delay[22m <dbl>,[39m
#> [90m#   [1mcarrier[22m <chr>, [1mflight[22m <int>, [1mtailnum[22m <chr>, [1morigin[22m <chr>, [1mdest[22m <chr>,[39m
#> [90m#   [1mair_time[22m <dbl>, [1mdistance[22m <dbl>, [1mhour[22m <dbl>, [1mminute[22m <dbl>, [1mtime_hour[22m <dttm>[39m
```

Graphically, a semi-join looks like this:

<img src="diagrams/join-semi.png" title="Diagram of a semi join. Data frame x is on the left and has two columns (key and val_x) with keys 1, 2, and 3. Diagram y is on the right and also has two columns (key and val_y) with keys 1, 2, and 4. Semi joining these two results in a data frame with two rows and two columns (key and val_x), with keys 1 and 2 (the only keys that match between the two data frames)." alt="Diagram of a semi join. Data frame x is on the left and has two columns (key and val_x) with keys 1, 2, and 3. Diagram y is on the right and also has two columns (key and val_y) with keys 1, 2, and 4. Semi joining these two results in a data frame with two rows and two columns (key and val_x), with keys 1 and 2 (the only keys that match between the two data frames)." width="307" style="display: block; margin: auto;" />

Only the existence of a match is important; it doesn't matter which observation is matched.
This means that filtering joins never duplicate rows like mutating joins do:

<img src="diagrams/join-semi-many.png" title="Diagram of a semi join with data frames with duplicated keys. Data frame x is on the left and has two columns (key and val_x) with keys 1, 2, 2, and 3. Diagram y is on the right and also has two columns (key and val_y) with keys 1, 2, 2, and 3 as well. Semi joining these two results in a data frame with four rows and two columns (key and val_x), with keys 1, 2, 2, and 3 (the matching keys, each appearing as many times as they do in x)." alt="Diagram of a semi join with data frames with duplicated keys. Data frame x is on the left and has two columns (key and val_x) with keys 1, 2, 2, and 3. Diagram y is on the right and also has two columns (key and val_y) with keys 1, 2, 2, and 3 as well. Semi joining these two results in a data frame with four rows and two columns (key and val_x), with keys 1, 2, 2, and 3 (the matching keys, each appearing as many times as they do in x)." width="312" style="display: block; margin: auto;" />

The inverse of a semi-join is an anti-join.
An anti-join keeps the rows that *don't* have a match:

<img src="diagrams/join-anti.png" title="Diagram of an anti join. Data frame x is on the left and has two columns (key and val_x) with keys 1, 2, and 3. Diagram y is on the right and also has two columns (key and val_y) with keys 1, 2, and 4. Anti joining these two results in a data frame with one row and two columns (key and val_x), with keys 3 only (the only key in x that is not in y)." alt="Diagram of an anti join. Data frame x is on the left and has two columns (key and val_x) with keys 1, 2, and 3. Diagram y is on the right and also has two columns (key and val_y) with keys 1, 2, and 4. Anti joining these two results in a data frame with one row and two columns (key and val_x), with keys 3 only (the only key in x that is not in y)." width="307" style="display: block; margin: auto;" />

Anti-joins are useful for diagnosing join mismatches.
For example, when connecting `flights` and `planes`, you might be interested to know that there are many `flights` that don't have a match in `planes`:


```r
flights %>%
  anti_join(planes, by = "tailnum") %>%
  count(tailnum, sort = TRUE)
#> [90m# A tibble: 722 x 2[39m
#>   [1mtailnum[22m     [1mn[22m
#>   [3m[90m<chr>[39m[23m   [3m[90m<int>[39m[23m
#> [90m1[39m [31mNA[39m       [4m2[24m512
#> [90m2[39m N725MQ    575
#> [90m3[39m N722MQ    513
#> [90m4[39m N723MQ    507
#> [90m5[39m N713MQ    483
#> [90m6[39m N735MQ    396
#> [90m# ... with 716 more rows[39m
```

### Exercises

1.  What does it mean for a flight to have a missing `tailnum`?
    What do the tail numbers that don't have a matching record in `planes` have in common?
    (Hint: one variable explains \~90% of the problems.)

2.  Filter flights to only show flights with planes that have flown at least 100 flights.

3.  Combine `fueleconomy::vehicles` and `fueleconomy::common` to find only the records for the most common models.

4.  Find the 48 hours (over the course of the whole year) that have the worst delays.
    Cross-reference it with the `weather` data.
    Can you see any patterns?

5.  What does `anti_join(flights, airports, by = c("dest" = "faa"))` tell you?
    What does `anti_join(airports, flights, by = c("faa" = "dest"))` tell you?

6.  You might expect that there's an implicit relationship between plane and airline, because each plane is flown by a single airline.
    Confirm or reject this hypothesis using the tools you've learned above.

## Join problems

The data you've been working with in this chapter has been cleaned up so that you'll have as few problems as possible.
Your own data is unlikely to be so nice, so there are a few things that you should do with your own data to make your joins go smoothly.

1.  Start by identifying the variables that form the primary key in each data frame.
    You should usually do this based on your understanding of the data, not empirically by looking for a combination of variables that give a unique identifier.
    If you just look for variables without thinking about what they mean, you might get (un)lucky and find a combination that's unique in your current data but the relationship might not be true in general.

    For example, the altitude and longitude uniquely identify each airport, but they are not good identifiers!

    
    ```r
    airports %>% count(alt, lon) %>% filter(n > 1)
    #> [90m# A tibble: 0 x 3[39m
    #> [90m# ... with 3 variables: [1malt[22m <dbl>, [1mlon[22m <dbl>, [1mn[22m <int>[39m
    ```

2.  Check that none of the variables in the primary key are missing.
    If a value is missing then it can't identify an observation!

3.  Check that your foreign keys match primary keys in another data frame.
    The best way to do this is with an `anti_join()`.
    It's common for keys not to match because of data entry errors.
    Fixing these is often a lot of work.

    If you do have missing keys, you'll need to be thoughtful about your use of inner vs. outer joins, carefully considering whether or not you want to drop rows that don't have a match.

Be aware that simply checking the number of rows before and after the join is not sufficient to ensure that your join has gone smoothly.
If you have an inner join with duplicate keys in both data frames, you might get unlucky as the number of dropped rows might exactly equal the number of duplicated rows!

## Set operations {#set-operations}

The final type of two-table verb are the set operations.
Generally, I use these the least frequently, but they are occasionally useful when you want to break a single complex filter into simpler pieces.
All these operations work with a complete row, comparing the values of every variable.
These expect the `x` and `y` inputs to have the same variables, and treat the observations like sets:

-   `intersect(x, y)`: return only observations in both `x` and `y`.
-   `union(x, y)`: return unique observations in `x` and `y`.
-   `setdiff(x, y)`: return observations in `x`, but not in `y`.

Given this simple data:


```r
df1 <- tribble(
  ~x, ~y,
   1,  1,
   2,  1
)
df2 <- tribble(
  ~x, ~y,
   1,  1,
   1,  2
)
```

The four possibilities are:


```r
intersect(df1, df2)
#> [90m# A tibble: 1 x 2[39m
#>       [1mx[22m     [1my[22m
#>   [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m
#> [90m1[39m     1     1

# Note that we get 3 rows, not 4
union(df1, df2)
#> [90m# A tibble: 3 x 2[39m
#>       [1mx[22m     [1my[22m
#>   [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m
#> [90m1[39m     1     1
#> [90m2[39m     2     1
#> [90m3[39m     1     2

setdiff(df1, df2)
#> [90m# A tibble: 1 x 2[39m
#>       [1mx[22m     [1my[22m
#>   [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m
#> [90m1[39m     2     1

setdiff(df2, df1)
#> [90m# A tibble: 1 x 2[39m
#>       [1mx[22m     [1my[22m
#>   [3m[90m<dbl>[39m[23m [3m[90m<dbl>[39m[23m
#> [90m1[39m     1     2
```
