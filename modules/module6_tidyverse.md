% R Bootcamp, Module 6: Data manipulation with the `tidyverse`
% January 2019, UC Berkeley
% [Nima Hejazi](https://nimahejazi.org) (based on materials developed by Kellie Ottoboni, Rochelle Terman, and Chris Krogslund)




# Overview

> It is often said that 80% of data analysis is spent on the process of cleaning and preparing the data. (Dasu and Johnson, 2003)

Thus before you can even get to doing any sort of sophisticated analysis or plotting, you'll generally first need to:

1. ***Manipulating*** data frames, e.g., filtering, summarizing, and conducting calculations across groups.
2. ***Tidying*** data into the appropriate format

There are two competing schools of thought within the R community.

* We should stick to the base R functions to do manipulating and tidying; `tidyverse` uses syntax that's unlike base R and is superfluous.
* We should start teaching students to manipulate data using `tidyverse` tools because they are straightfoward to use, more readable than base R, and speed up the tidying process.

We'll show you some of the `tidyverse` tools so you can make an informed decision about whether you want to use base R or these newfangled packages.

# Data frame Manipulation using Base R Functions

So far, you've seen the basics of manipulating data frames, e.g. subsetting, merging, and basic calculations.
For instance, we can use base R functions to calculate summary statistics across groups of observations,
e.g., the mean GDP per capita within each region:


```r
mean(gap[gap$continent == "Africa", "gdpPercap"])
```

```
## [1] 2193.755
```

```r
mean(gap[gap$continent == "Americas", "gdpPercap"])
```

```
## [1] 7136.11
```

```r
mean(gap[gap$continent == "Asia", "gdpPercap"])
```

```
## [1] 7902.15
```

But this isn't ideal because it involves a fair bit of repetition. Repeating yourself will cost you time, both now and later, and potentially introduce some nasty bugs.

# Data frame Manipulation using `dplyr`

Luckily, the [`dplyr`](https://cran.r-project.org/web/packages/dplyr/dplyr.pdf) package provides a number of very useful functions for manipulating data frames. These functions will save you time by reducing repetition. As an added bonus, you might even find the `dplyr` grammar easier to read.

Here we're going to cover 6 of the most commonly used functions as well as using pipes (`%>%`) to combine them.

1. `select()`
2. `filter()`
3. `group_by()`
4. `summarize()`
5. `mutate()`
6. `arrange()`

If you have have not installed this package earlier, please do so now:


```r
# NOT run
install.packages('dplyr')
```

Now let's load the package:


```r
library(dplyr)
```

# `dplyr::select`

Imagine that we just received the gapminder dataset, but are only interested in a few variables in it. We could use the `select()` function to keep only the columns corresponding to variables we select.


```r
year_country_gdp_dplyr <- select(gap, year, country, gdpPercap)
head(year_country_gdp_dplyr)
```

```
##   year     country gdpPercap
## 1 1952 Afghanistan  779.4453
## 2 1957 Afghanistan  820.8530
## 3 1962 Afghanistan  853.1007
## 4 1967 Afghanistan  836.1971
## 5 1972 Afghanistan  739.9811
## 6 1977 Afghanistan  786.1134
```

![](img/dplyr-fig1.png)

If we open up `year_country_gdp`, we'll see that it only contains the year, country and gdpPercap. This is equivalent to the base R subsetting function:


```r
year_country_gdp_base <- gap[,c("year", "country", "gdpPercap")]
head(year_country_gdp_base)
```

```
##   year     country gdpPercap
## 1 1952 Afghanistan  779.4453
## 2 1957 Afghanistan  820.8530
## 3 1962 Afghanistan  853.1007
## 4 1967 Afghanistan  836.1971
## 5 1972 Afghanistan  739.9811
## 6 1977 Afghanistan  786.1134
```

We can even check that these two data frames are equivalent:


```r
# checking equivalence: TRUE indicates an exact match between these objects
all.equal(year_country_gdp_dplyr, year_country_gdp_base)
```

```
## [1] TRUE
```

But, as we will see, `dplyr` makes for much more readable, efficient code because of its *pipe* operator.

# piping with `dplyr`

![](img/magrittr_hex.png)

Above, we used what's called "normal" grammar, but the strengths of `dplyr` lie in combining several functions using *pipes*.
Pipes take the input on the left side of the `%>%` symbol and pass it in as the first argument to the function on the right side.
Since the pipe grammar is unlike anything we've seen in R before, let's repeat what we've done above using pipes.


```r
year_country_gdp <- gap %>% select(year, country, gdpPercap)
```

First we summon the gapminder dataframe and pass it on to the next step using the pipe symbol `%>%`
The second steps is the `select()` function.
In this case we don't specify which data object we use in the call to `select()` since we've piped it in.

**Fun Fact**: There is a good chance you have encountered pipes before in the shell. In R, a pipe symbol is `%>%` while in the shell it is `|.` But the concept is the same!

# `dplyr::filter`

Now let's say we're only interested in African countries. We can combine `select` and `filter` to select only the observations where `continent` is `Africa`.


```r
year_country_gdp_africa <- gap %>%
    filter(continent == "Africa") %>%
    select(year,country,gdpPercap)
```

As with last time, first we pass the gapminder data frame to the `filter()` function, then we pass the filtered version of the gapminder data frame to the `select()` function.

To clarify, both the `select` and `filter` functions subsets the data frame. The difference is that `select` extracts certain *columns*, while `filter` extracts certain *rows*.

**Note:** The order of operations is very important in this case. If we used 'select' first, filter would not be able to find the variable `continent` since we would have removed it in the previous step.

# `dplyr` Calculations Across Groups

A common task you'll encounter when working with data is running calculations on different groups within the data. For instance, what if we wanted to calculate the mean GDP per capita for each continent?

In base R, you would have to run the `mean()` function for each subset of data:


```r
mean(gap$gdpPercap[gap$continent == "Africa"])
```

```
## [1] 2193.755
```

```r
mean(gap$gdpPercap[gap$continent == "Americas"])
```

```
## [1] 7136.11
```

```r
mean(gap$gdpPercap[gap$continent == "Asia"])
```

```
## [1] 7902.15
```

```r
mean(gap$gdpPercap[gap$continent == "Europe"])
```

```
## [1] 14469.48
```

```r
mean(gap$gdpPercap[gap$continent == "Oceania"])
```

```
## [1] 18621.61
```

That's a lot of repetition! To make matters worse, what if we wanted to add these values to our original data frame as a new column? We would have to write something like this:


```r
gap$mean.continent.GDP <- NA

gap$mean.continent.GDP[gap$continent == "Africa"] <- mean(gap$gdpPercap[gap$continent == "Africa"])

gap$mean.continent.GDP[gap$continent == "Americas"] <- mean(gap$gdpPercap[gap$continent == "Americas"])

gap$mean.continent.GDP[gap$continent == "Asia"] <- mean(gap$gdpPercap[gap$continent == "Asia"])

gap$mean.continent.GDP[gap$continent == "Europe"] <- mean(gap$gdpPercap[gap$continent == "Europe"])

gap$mean.continent.GDP[gap$continent == "Oceania"] <- mean(gap$gdpPercap[gap$continent == "Oceania"])
```

You can see how this can get pretty tedious, especially if we want to calculate more complicated or refined statistics. We could use loops or apply functions, but these can be difficult, slow, or error-prone.

# `dplyr` split-apply-combine

The abstract problem we're encountering here is know as "split-apply-combine":

![](img/splitapply.png)

We want to *split* our data into groups (in this case continents), *apply* some calculations on each group, then  *combine* the results together afterwards.

Module 4 gave some ways to do split-apply-combine type stuff using the `apply` family of functions, but those are error prone and messy.

Luckily, `dplyr` offers a much cleaner, straight-forward solution to this problem.


```r
# remove this column -- there are two easy ways!
gap <- gap %>% select(-mean.continent.GDP)
# OR
gap$mean.continent.GDP <- NULL
```

# `dplyr::group_by`

We've already seen how `filter()` can help us select observations that meet certain criteria (in the above: `continent == "Europe"`). More helpful, however, is the `group_by()` function, which will essentially use every unique criteria that we could have used in `filter()`.

A `grouped_df` can be thought of as a `list` where each item in the `list` is a `data.frame` which contains only the rows that correspond to the a particular value `continent` (at least in the example above).

![](img/dplyr-fig2.png)

# `dplyr::summarize`

`group_by()` on its own is not particularly interesting.
It's much more exciting used in conjunction with the `summarize()` function.
This will allow use to create new variable(s) by applying transformations to variables in each of the continent-specific data frames.
In other words, using the `group_by()` function, we split our original data frame into multiple pieces, which we then apply summary functions to (e.g. `mean()` or `sd()`) within `summarize()`.
The output is a new data frame reduced in size, with one row per group.


```r
gdp_bycontinents <- gap %>%
    group_by(continent) %>%
    summarize(mean_gdpPercap = mean(gdpPercap))
head(gdp_bycontinents)
```

```
## # A tibble: 5 x 2
##   continent mean_gdpPercap
##   <chr>              <dbl>
## 1 Africa             2194.
## 2 Americas           7136.
## 3 Asia               7902.
## 4 Europe            14469.
## 5 Oceania           18622.
```

![](img/dplyr-fig3.png)

That allowed us to calculate the mean gdpPercap for each continent. But it gets even better -- the function `group_by()` allows us to group by multiple variables. Let's group by `year` and `continent`.


```r
gdp_bycontinents_byyear <- gap %>%
    group_by(continent, year) %>%
    summarize(mean_gdpPercap = mean(gdpPercap))
head(gdp_bycontinents_byyear)
```

```
## # A tibble: 6 x 3
## # Groups:   continent [1]
##   continent  year mean_gdpPercap
##   <chr>     <int>          <dbl>
## 1 Africa     1952          1253.
## 2 Africa     1957          1385.
## 3 Africa     1962          1598.
## 4 Africa     1967          2050.
## 5 Africa     1972          2340.
## 6 Africa     1977          2586.
```

That is already quite powerful, but it gets even better! You're not limited to defining 1 new variable in `summarize()`.


```r
gdp_pop_bycontinents_byyear <- gap %>%
    group_by(continent, year) %>%
    summarize(mean_gdpPercap = mean(gdpPercap),
              sd_gdpPercap = sd(gdpPercap),
              mean_pop = mean(pop),
              sd_pop = sd(pop))
head(gdp_pop_bycontinents_byyear)
```

```
## # A tibble: 6 x 6
## # Groups:   continent [1]
##   continent  year mean_gdpPercap sd_gdpPercap mean_pop    sd_pop
##   <chr>     <int>          <dbl>        <dbl>    <dbl>     <dbl>
## 1 Africa     1952          1253.         983. 4570010.  6317450.
## 2 Africa     1957          1385.        1135. 5093033.  7076042.
## 3 Africa     1962          1598.        1462. 5702247.  7957545.
## 4 Africa     1967          2050.        2848. 6447875.  8985505.
## 5 Africa     1972          2340.        3287. 7305376. 10130833.
## 6 Africa     1977          2586.        4142. 8328097. 11585184.
```

# `dplyr::mutate`

What if we wanted to add these values to our original data frame instead of creating a new object? For this, we can use the `mutate()` function, which is similar to `summarize()` except it creates new variables to the same data frame that you pass into it.


```r
gap_with_extra_vars <- gap %>%
    group_by(continent, year) %>%
    mutate(mean_gdpPercap = mean(gdpPercap),
              sd_gdpPercap = sd(gdpPercap),
              mean_pop = mean(pop),
              sd_pop = sd(pop))
head(gap_with_extra_vars)
```

```
## # A tibble: 6 x 10
## # Groups:   continent, year [6]
##   country      year      pop continent lifeExp gdpPercap mean_gdpPercap
##   <chr>       <int>    <dbl> <chr>       <dbl>     <dbl>          <dbl>
## 1 Afghanistan  1952  8425333 Asia         28.8      779.          5195.
## 2 Afghanistan  1957  9240934 Asia         30.3      821.          5788.
## 3 Afghanistan  1962 10267083 Asia         32.0      853.          5729.
## 4 Afghanistan  1967 11537966 Asia         34.0      836.          5971.
## 5 Afghanistan  1972 13079460 Asia         36.1      740.          8187.
## 6 Afghanistan  1977 14880372 Asia         38.4      786.          7791.
## # ... with 3 more variables: sd_gdpPercap <dbl>, mean_pop <dbl>,
## #   sd_pop <dbl>
```

We can use also use `mutate()` to create new variables prior to (or even after) summarizing information.


```r
gdp_pop_bycontinents_byyear <- gap %>%
    mutate(gdp_billion = gdpPercap*pop/10^9) %>%
    group_by(continent, year) %>%
    summarize(mean_gdpPercap = mean(gdpPercap),
              sd_gdpPercap = sd(gdpPercap),
              mean_pop = mean(pop),
              sd_pop = sd(pop),
              mean_gdp_billion = mean(gdp_billion),
              sd_gdp_billion = sd(gdp_billion))
head(gdp_pop_bycontinents_byyear)
```

```
## # A tibble: 6 x 8
## # Groups:   continent [1]
##   continent  year mean_gdpPercap sd_gdpPercap mean_pop    sd_pop
##   <chr>     <int>          <dbl>        <dbl>    <dbl>     <dbl>
## 1 Africa     1952          1253.         983. 4570010.  6317450.
## 2 Africa     1957          1385.        1135. 5093033.  7076042.
## 3 Africa     1962          1598.        1462. 5702247.  7957545.
## 4 Africa     1967          2050.        2848. 6447875.  8985505.
## 5 Africa     1972          2340.        3287. 7305376. 10130833.
## 6 Africa     1977          2586.        4142. 8328097. 11585184.
## # ... with 2 more variables: mean_gdp_billion <dbl>, sd_gdp_billion <dbl>
```

# `dplyr::arrange`

As a last step, let's say we want to sort the rows in our data frame according to values in a certain column. We can use the `arrange()` function to do this. For instance, let's organize our rows by `year` (recent first), and then by `continent`.


```r
gap_with_extra_vars <- gap %>%
    group_by(continent, year) %>%
    mutate(mean_gdpPercap = mean(gdpPercap),
              sd_gdpPercap = sd(gdpPercap),
              mean_pop = mean(pop),
              sd_pop = sd(pop)) %>%
    arrange(desc(year), continent)
head(gap_with_extra_vars)
```

```
## # A tibble: 6 x 10
## # Groups:   continent, year [1]
##   country       year      pop continent lifeExp gdpPercap mean_gdpPercap
##   <chr>        <int>    <dbl> <chr>       <dbl>     <dbl>          <dbl>
## 1 Algeria       2007 33333216 Africa       72.3     6223.          3089.
## 2 Angola        2007 12420476 Africa       42.7     4797.          3089.
## 3 Benin         2007  8078314 Africa       56.7     1441.          3089.
## 4 Botswana      2007  1639131 Africa       50.7    12570.          3089.
## 5 Burkina Faso  2007 14326203 Africa       52.3     1217.          3089.
## 6 Burundi       2007  8390505 Africa       49.6      430.          3089.
## # ... with 3 more variables: sd_gdpPercap <dbl>, mean_pop <dbl>,
## #   sd_pop <dbl>
```

# `dplyr` Take-aways

* Human readable: the function names describe the action being done
* Piping: chain functions in a step-by-step way, rather than nesting


```r
# without pipes:
gap_with_extra_vars <- arrange(
    mutate(
      group_by(gap, continent, year),
      mean_gdpPercap = mean(gdpPercap)
      ),
    desc(year), continent)
```
* Facilitates split-apply-combine manipulations

# Tidying Data

Even before we conduct analysis or calculations, we need to put our data into the correct format. The goal here is to rearrange a messy dataset into one that is **tidy**

The two most important properties of tidy data are:

1) Each column is a variable.
2) Each row is an observation.

Tidy data is easier to work with, because you have a consistent way of referring to variables (as column names) and observations (as row indices). It then becomes easy to manipulate, visualize, and model.

For more on the concept of *tidy* data, read Hadley Wickham's paper [here](http://vita.had.co.nz/papers/tidy-data.html)

# Tidying Data/Wide vs. Long Formats

> "Tidy datasets are all alike but every messy dataset is messy in its own way." – Hadley Wickham

Tabular datasets can be arranged in many ways. For instance, consider the data below. Each data set displays information on heart rate observed in individuals across 3 different time periods. But the data are organized differently in each table.


```r
wide <- data.frame(
  name = c("Wilbur", "Petunia", "Gregory"),
  time1 = c(67, 80, 64),
  time2 = c(56, 90, 50),
  time3 = c(70, 67, 101)
)
wide
```

```
##      name time1 time2 time3
## 1  Wilbur    67    56    70
## 2 Petunia    80    90    67
## 3 Gregory    64    50   101
```

```r
long <- data.frame(
  name = c("Wilbur", "Petunia", "Gregory", "Wilbur", "Petunia", "Gregory", "Wilbur", "Petunia", "Gregory"),
  time = c(1, 1, 1, 2, 2, 2, 3, 3, 3),
  heartrate = c(67, 80, 64, 56, 90, 50, 70, 67, 10)
)
long
```

```
##      name time heartrate
## 1  Wilbur    1        67
## 2 Petunia    1        80
## 3 Gregory    1        64
## 4  Wilbur    2        56
## 5 Petunia    2        90
## 6 Gregory    2        50
## 7  Wilbur    3        70
## 8 Petunia    3        67
## 9 Gregory    3        10
```

**Question**: Which one of these do you think is the *tidy* format?

**Answer**: The first dataframe (the "wide" one) would not be considered *tidy* because values (i.e., heartrate) are spread across multiple columns.

We often refer to these different structurs as "long" vs. "wide" formats. In the "long" format, you usually have 1 column for the observed variable and the other columns are ID variables.

For the "wide" format each row is often a site/subject/patient and you have multiple observation variables containing the same type of data. These can be either repeated observations over time, or observation of multiple variables (or a mix of both). In the above case, we had the same kind of data (heart rate) entered across 3 different columns, corresponding to three different time periods.

![](img/tidyr-fig1.png)

You may find data input may be simpler and some programs/functions may prefer the "wide" format. However, many of R’s functions have been designed assuming you have "long" format data.

# Tidying the Gapminder Data

Lets look at the structure of our original gapminder dataframe:


```r
head(gap)
```

```
##       country year      pop continent lifeExp gdpPercap
## 1 Afghanistan 1952  8425333      Asia  28.801  779.4453
## 2 Afghanistan 1957  9240934      Asia  30.332  820.8530
## 3 Afghanistan 1962 10267083      Asia  31.997  853.1007
## 4 Afghanistan 1967 11537966      Asia  34.020  836.1971
## 5 Afghanistan 1972 13079460      Asia  36.088  739.9811
## 6 Afghanistan 1977 14880372      Asia  38.438  786.1134
```

**Question**: Is this data frame **wide** or **long**?

**Answer**: This data frame is somewhere in between the purely 'long' and 'wide' formats. We have 3 "ID variables" (`continent`, `country`, `year`) and 3 "Observation variables" (`pop`, `lifeExp`, `gdpPercap`).

Despite not having ALL observations in 1 column, this intermediate format makes sense given that all 3 observation variables have different units. As we have seen, many of the functions in R are often vector based, and you usually do not want to do mathematical operations on values with different units.

On the other hand, there are some instances in which a purely long or wide format is ideal (e.g. plotting). Likewise, sometimes you'll get data on your desk that is poorly organized, and you'll need to **reshape** it.

# `tidyr`

Thankfully, the `tidyr` package will help you efficiently transform your data regardless of original format.


```r
# Install the "tidyr" package (only necessary one time)
# install.packages("tidyr") # Not Run

# Load the "tidyr" package (necessary every new R session)
library(tidyr)
```

# `tidyr::gather`

Until now, we've been using the nicely formatted original gapminder data set. This data set is not quite wide and not quite long -- it's something in the middle, but "real" data (i.e., our own research data) will never be so well organized. Here let's start with the wide format version of the gapminder data set.


```r
gap_wide <- read.csv("../data/gap_wide.csv", stringsAsFactors = FALSE)
```

```
## Warning in file(file, "rt"): cannot open file '../data/gap_wide.csv': No
## such file or directory
```

```
## Error in file(file, "rt"): cannot open the connection
```

```r
head(gap_wide)
```

```
## Error in head(gap_wide): object 'gap_wide' not found
```

The first step towards getting our nice intermediate data format is to first convert from the wide to the long format.
The function `gather()` will 'gather' the observation variables into a single variable. This is sometimes called "melting" your data, because it melts the table from wide to long. Those data will be melted into two variables: one for the variable names, and the other for the variable values.


```r
gap_long <- gap_wide %>%
    gather(obstype_year, obs_values, 3:38)
```

```
## Error in eval(lhs, parent, parent): object 'gap_wide' not found
```

```r
head(gap_long)
```

```
## Error in head(gap_long): object 'gap_long' not found
```

Notice that we put 3 arguments into the `gather()` function:

1. the name the new column for the new ID variable (`obstype_year`),
2. the name for the new amalgamated observation variable (`obs_value`),
3. the indices of the old variables (`3:38`, signalling columns 3 through 38) that we want to gather into one variable. Notice that we don't want to melt down columns 1 and 2, as these are considered "ID" variables.

# `tidyr::select`

If there are a lot of columns or they're named in a consistent pattern, we might not want to select them using the column numbers.
It'd be easier to use some information contained in the names themselves.
We can select variables using:

* variable indices
* variable names (without quotes)
* `x:z` to select all variables between x and z
* `-y` to *exclude* y
* `starts_with(x, ignore.case = TRUE)`: all names that starts with `x`
* `ends_with(x, ignore.case = TRUE)`: all names that ends with `x`
* `contains(x, ignore.case = TRUE)`: all names that contain `x`

See the `select()` function in `dplyr` for more options.

For instance, here we do the same gather operation with (1) the `starts_with` function, and (2) the `-` operator:


```r
# with the starts_with() function
gap_long <- gap_wide %>%
    gather(obstype_year, obs_values, starts_with('pop'),
           starts_with('lifeExp'), starts_with('gdpPercap'))
```

```
## Error in eval(lhs, parent, parent): object 'gap_wide' not found
```

```r
head(gap_long)
```

```
## Error in head(gap_long): object 'gap_long' not found
```

```r
# with the - operator
gap_long <- gap_wide %>%
  gather(obstype_year, obs_values, -continent, -country)
```

```
## Error in eval(lhs, parent, parent): object 'gap_wide' not found
```

```r
head(gap_long)
```

```
## Error in head(gap_long): object 'gap_long' not found
```

However you choose to do it, notice that the output collapses all of the measure variables into two columns: one containing new ID variable, the other containing the observation value for that row.

# `tidyr::separate`

You'll notice that in our long dataset, `obstype_year` actually contains 2 pieces of information, the observation type (`pop`, `lifeExp`, or `gdpPercap`) and the `year`.

We can use the `separate()` function to split the character strings into multiple variables:


```r
gap_long_sep <- gap_long %>%
  separate(obstype_year, into = c('obs_type','year'), sep = "_") %>%
  mutate(year = as.integer(year))
```

```
## Error in eval(lhs, parent, parent): object 'gap_long' not found
```

```r
head(gap_long_sep)
```

```
## Error in head(gap_long_sep): object 'gap_long_sep' not found
```

If you didn't use `tidyr` to do this, you'd have to use the `strsplit` function and use multiple lines of code to replace the column in `gap_long` with two new columns. This solution is much cleaner.

# `tidyr::spread`

The opposite of `gather()` is `spread()`. It spreads our observation variables back out to make a wider table. We can use this function to spread our `gap_long()` to the original "medium" format.


```r
gap_medium <- gap_long_sep %>%
  spread(obs_type, obs_values)
```

```
## Error in eval(lhs, parent, parent): object 'gap_long_sep' not found
```

```r
head(gap_medium)
```

```
## Error in head(gap_medium): object 'gap_medium' not found
```

All we need is some quick fixes to make this dataset identical to the original `gap` dataset:


```r
gap <- read.csv("../data/gapminder-FiveYearData.csv")
head(gap_medium)
```

```
## Error in head(gap_medium): object 'gap_medium' not found
```

```r
head(gap)
```

```
##       country year      pop continent lifeExp gdpPercap
## 1 Afghanistan 1952  8425333      Asia  28.801  779.4453
## 2 Afghanistan 1957  9240934      Asia  30.332  820.8530
## 3 Afghanistan 1962 10267083      Asia  31.997  853.1007
## 4 Afghanistan 1967 11537966      Asia  34.020  836.1971
## 5 Afghanistan 1972 13079460      Asia  36.088  739.9811
## 6 Afghanistan 1977 14880372      Asia  38.438  786.1134
```

```r
# rearrange columns
gap_medium <- gap_medium[,names(gap)]
```

```
## Error in eval(expr, envir, enclos): object 'gap_medium' not found
```

```r
head(gap_medium)
```

```
## Error in head(gap_medium): object 'gap_medium' not found
```

```r
# arrange by country, continent, and year
gap_medium <- gap_medium %>%
  arrange(country,continent,year)
```

```
## Error in eval(lhs, parent, parent): object 'gap_medium' not found
```

```r
head(gap_medium)
```

```
## Error in head(gap_medium): object 'gap_medium' not found
```

# Extra Resources

`dplyr` and `tidyr` have many more functions to help you wrangle and manipulate your data. See the  [Data Wrangling Cheat Sheet](https://www.rstudio.com/wp-content/uploads/2015/02/data-wrangling-cheatsheet.pdf) for more.

There are some other useful packages in the [tidyverse](http://www.tidyverse.org):

* `ggplot2` for plotting (I'll cover this in module 8)
* `readr` and `haven` for reading in data with structure other than csv
* `stringr`, `lubridate`, `forcats` for manipulating strings, dates, and factors, respectively
* many more!

# Breakout

### `dplyr`

1. Use `dplyr` to create a data frame containing the median `lifeExp` for each continent

2. Use `dplyr` to add a column to the gapminder dataset that contains the total population of the continent of each observation in a given year. For example, if the first observation is Afghanistan in 1952, the new column would contain the population of Asia in 1952.

3. Use `dplyr` to add a column called `gdpPercap_diff` that contains the difference between the observation's `gdpPercap` and the mean `gdpPercap` of the continent in that year. Arrange the dataframe by the column you just created, in descending order (so that the relatively richest country/years are listed first)

### `tidyr`

4. Subset the results from question #3 to select only the `country`, `year`, and `gdpPercap_diff` columns. Use tidyr put it in wide format so that countries are rows and years are columns.

Hint: you'll probably see a message about a missing grouping variable. If you don't want continent included, you can pass the output of problem 3 through `ungroup()` to get rid of the continent information.
