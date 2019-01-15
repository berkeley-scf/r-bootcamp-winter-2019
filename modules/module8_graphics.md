% R Bootcamp, Module 8: Graphics
% January 2019, UC Berkeley
% [Nima Hejazi](https://nimahejazi.org) (built off of material by Kellie Ottoboni and Chris Krogslund)



# By way of introduction...

* 3 main facilities for producing graphics in R: **base**, **`ggplot2`**, and **`lattice`**
* In practice, these facilities are grouped into two camps: "basic" and "advanced"
* A better formulation: quick/dirty v. involved/fancy

# The data

* All Summer Olympic medalists from 1896-2008
* Variables include name, gender, country, country code (`NOC`), sporting event, and type of medal won
* We don't actually care about names of winners: we're interested in how many medals that different countries won

For more info, see [The Guardian Datablog Olympic medal winners: every one since 1896 as open data](https://www.theguardian.com/sport/datablog/2012/jun/25/olympic-medal-winner-list-data)

First, we'll use `dplyr` and `tidyr` to count the medals of each type won, for each country and each year.


```r
head(medals)
```

```
##     City    Sport Discipline               Athlete NOC Gender
## 1 Athens Aquatics   Swimming         HAJOS, Alfred HUN    Men
## 2 Athens Aquatics   Swimming      HERSCHMANN, Otto AUT    Men
## 3 Athens Aquatics   Swimming     DRIVAS, Dimitrios GRE    Men
## 4 Athens Aquatics   Swimming    MALOKINIS, Ioannis GRE    Men
## 5 Athens Aquatics   Swimming    CHASAPIS, Spiridon GRE    Men
## 6 Athens Aquatics   Swimming CHOROPHAS, Efstathios GRE    Men
##                        Event Event_gender  Medal Year Country
## 1             100m freestyle            M   Gold 1896 Hungary
## 2             100m freestyle            M Silver 1896 Austria
## 3 100m freestyle for sailors            M Bronze 1896  Greece
## 4 100m freestyle for sailors            M   Gold 1896  Greece
## 5 100m freestyle for sailors            M Silver 1896  Greece
## 6            1200m freestyle            M Bronze 1896  Greece
```

```r
# dplyr and tidyr refresher
medal_counts <- medals %>%
  group_by(Medal, Year, NOC) %>%
  summarise(count = n())
head(medal_counts)
```

```
## # A tibble: 6 x 4
## # Groups:   Medal, Year [1]
##   Medal   Year NOC   count
##   <chr>  <int> <chr> <int>
## 1 Bronze  1896 AUT       2
## 2 Bronze  1896 DEN       3
## 3 Bronze  1896 FRA       2
## 4 Bronze  1896 GBR       2
## 5 Bronze  1896 GER       2
## 6 Bronze  1896 GRE      22
```

This table is in tidy format. Wide (untidy) format can be useful for plotting in base plot (more on this later)


```r
medal_counts_wide <- medal_counts %>% spread(key = Medal, value = count) %>%
  ungroup() %>%
  mutate(Bronze = ifelse(is.na(Bronze), 0, Bronze)) %>%
  mutate(Silver = ifelse(is.na(Silver), 0, Silver)) %>%
  mutate(Gold = ifelse(is.na(Gold), 0, Gold))
head(medal_counts_wide)
```

```
## # A tibble: 6 x 5
##    Year NOC   Bronze  Gold Silver
##   <int> <chr>  <dbl> <dbl>  <dbl>
## 1  1896 AUS        0     2      0
## 2  1896 AUT        2     2      1
## 3  1896 DEN        3     1      2
## 4  1896 FRA        2     5      4
## 5  1896 GBR        2     2      3
## 6  1896 GER        2    26      5
```

Finally, let's subset the data to gold medal counts for the US, for easier plotting.


```r
usa_gold_medals <-  medal_counts %>%
  dplyr::filter(Medal == "Gold") %>%
  dplyr::filter(NOC == "USA")
```

# Base graphics

The general call for base plot looks something like this:


```r
plot(x = , y = , ...)
```
Additional parameters can be passed in to customize the plot:

* type: scatterplot? lines? etc
* main: a title
* xlab, ylab: x-axis and y-axis labels
* col: color, either a string with the color name or a vector of color names for each point

More layers can be added to the plot with additional calls to `lines`, `points`, `text`, etc.


```r
plot(medal_counts_wide$Year, medal_counts_wide$Gold) # Basic
```

![](figure/unnamed-chunk-5-1.png)

```r
plot(usa_gold_medals$Year, usa_gold_medals$count, type = "l",
     main = "USA Gold Medals",
     xlab = "Year", ylab = "Count") # with updated parameters
points(x = 1984, y = usa_gold_medals$count[usa_gold_medals$Year == 1984],
       col = "red", pch = 16)
```

![](figure/unnamed-chunk-5-2.png)

# Other plot types in base graphics

These are just a few other types of plots you can make in base graphics.


```r
boxplot(Gold~Year, data = medal_counts_wide)
```

![](figure/unnamed-chunk-6-1.png)

```r
hist(medal_counts_wide$Gold)
```

![](figure/unnamed-chunk-6-2.png)

```r
plot(density(medal_counts_wide$Gold))
```

![](figure/unnamed-chunk-6-3.png)

```r
barplot(usa_gold_medals$count, width = 4, names.arg = usa_gold_medals$Year,
                               main = "USA Gold Medals")
```

![](figure/unnamed-chunk-6-4.png)

```r
mosaicplot(Year~Medal, medal_counts)
```

![](figure/unnamed-chunk-6-5.png)

# Object-oriented plots
* Base graphics often recognizes the object type and will implement specific plot methods
* lattice and ggplot2 generally **don't** exhibit this sort of behavior


```r
medal_lm <- lm(Gold ~ Bronze + Silver, data = medal_counts_wide)

# Calls plotting method for class of the dataset ("data.frame")
plot(medal_counts_wide %>% select(-NOC))
```

![ ](figure/unnamed-chunk-7-1.png)

```r
# Calls plotting method for class of medal_lm object ("lm"), print first two plots only
plot(medal_lm, which=1:2)
```

![ ](figure/unnamed-chunk-7-2.png)![ ](figure/unnamed-chunk-7-3.png)


# Pros/cons of base graphics, ggplot2, and lattice

Base graphics is

a) good for exploratory data analysis and sanity checks

b) inconsistent in syntax across functions: some take x,y while others take formulas

c) defaults plotting parameters are ugly and it can be difficult to customize

d) that said, one can do essentially anything in base graphics with some work

`ggplot2` is

a) generally more elegant

b) more syntactically logical (and therefore simpler, once you learn it)

c) better at grouping

d) able to interface with maps

`lattice` is

a) faster than ggplot2 (though only noticeable over many and large plots)

b) simpler than ggplot2 (at first)

c) better at trellis graphs than ggplot2

d) able to do 3d graphs

We'll focus on ggplot2 as it is very powerful, very widely-used and allows one to produce very nice-looking graphics without a lot of coding.


# Basic usage: `ggplot2`

The general call for `ggplot2` graphics looks something like this:


```r
# NOT run
ggplot(data = , aes(x = ,y = , [options])) + geom_xxxx() + ... + ... + ...
```

Note that `ggplot2` graphs in layers in a *continuing call* (hence the endless +...+...+...), which really makes the extra layer part of the call


```r
... + geom_xxxx(data = , aes(x = , y = ,[options]), [options]) + ... + ... + ...
```
You can see the layering effect by comparing the same graph with different colors for each layer


```r
p <- ggplot(data = medal_counts_wide, aes(x = Year, y = Gold)) +
                 geom_point(color = "gold")
p
```

![ ](figure/unnamed-chunk-10-1.png)

```r
p + geom_point(aes(x = Year, y = Silver), color = "gray") + ylab("Medals")
```

![ ](figure/unnamed-chunk-10-2.png)

# Grammar of Graphics

`ggplot2` syntax is very different from base graphics and lattice. It's built on the **grammar of graphics**.
The basic idea is that the visualization of all data requires four items:

1) One or more **statistics** conveying information about the data (identities, means, medians, etc.)

2) A **coordinate system** that differentiates between the intersections of statistics (at most two for `ggplot`, three for `lattice`)

3) **Geometries** that differentiate between off-coordinate variation in *kind*

4) **Scales** that differentiate between off-coordinate variation in *degree*

`ggplot2` allows the user to manipulate all four of these items.



```r
ggplot(medal_counts_wide, aes(x = Year, y = Gold)) + geom_point() +
                          ggtitle("Gold Medal Counts")
```

![](figure/unnamed-chunk-11-1.png)

```r
ggplot(usa_gold_medals, aes(x = Year, y = count)) + geom_line() +
                        ggtitle("USA Gold Medals")
```

![](figure/unnamed-chunk-11-2.png)

```r
# Boxplots
ggplot(medal_counts_wide, aes(x = factor(Year), y = Gold)) +
                          geom_boxplot() + ggtitle("Gold Medal Counts")
```

![](figure/unnamed-chunk-11-3.png)

```r
# Histogram
ggplot(medal_counts_wide, aes(x = Gold)) + geom_histogram() +
                          ggtitle("Gold Medal Counts")
```

![](figure/unnamed-chunk-11-4.png)

```r
# Density plot
ggplot(medal_counts_wide, aes(x = Gold)) + geom_density() +
                          ggtitle("Gold Medal Counts")
```

![](figure/unnamed-chunk-11-5.png)

```r
# Bar chart
ggplot(usa_gold_medals, aes(x = Year, y = count)) + geom_bar(stat = "identity")
```

![](figure/unnamed-chunk-11-6.png)


# `ggplot2` and tidy data

* `ggplot2` plays nice with `dplyr` and pipes. If you want to manipulate your data specifically for one plot but not save the new dataset, you can call your `dplyr` chain and pipe it directly into a `ggplot` call.


```r
# This combines the subsetting and plotting into one step
medal_counts %>%
  dplyr::filter(Medal == "Gold") %>%
  dplyr::filter(NOC == "USA") %>%
  ggplot(aes(x = Year, y = count)) + geom_line()
```

![](figure/unnamed-chunk-12-1.png)

* Base graphics/lattice and `ggplot2` have one big difference: `ggplot2` **requires** your data to be in tidy format. For base graphics, it can actually be helpful *not* to have your data in tidy format.
The difference is that `ggplot` treats `Medal` as an aesthetic parameter that differentiates kinds of statistics, whereas base graphics treats each (year, medal) pair as a set of inputs to the plot.
Compare:


```r
usa_all_medals <- medal_counts %>%
  dplyr::filter(NOC == "USA")

# ggplot2 call
ggplot(data = usa_all_medals, aes(x = Year, y = count)) +
            geom_line(aes(color = Medal))
```

![](figure/unnamed-chunk-13-1.png)


```r
usa_all_medals_untidy <- medal_counts_wide %>%
  dplyr::filter(NOC == "USA")

# Base graphics call
plot(usa_all_medals_untidy$Year, usa_all_medals_untidy$Gold, col = "green",
                                 type = "l")
lines(usa_all_medals_untidy$Year, usa_all_medals_untidy$Silver, col = "blue")
lines(usa_all_medals_untidy$Year, usa_all_medals_untidy$Bronze, col = "red")
legend("right", legend = c("Gold", "Silver", "Bronze"),
                fill = c("green", "blue", "red"))
```

![](figure/unnamed-chunk-14-1.png)


# Pros/cons of `ggplot2`

* Allows you to add features in "layers"
* Automatically adjusts spacing and sizing as you add more layers
* Requires data to be in tidy format
* Syntax is different from base R -- there is a learning curve
* Plots are actually objects. You can assign them to a variable and do things with it (more on this later)

# An overview of syntax for various `ggplot2` plots

Densities:


```r
ggplot(data = usa_gold_medals, aes(x = count)) + geom_density() # ggplot2
```

![ ](figure/unnamed-chunk-15-1.png)

X-Y scatter plots:


```r
ggplot(data = usa_gold_medals, aes(x = Year, y = count)) + geom_point() # ggplot2
```

![ ](figure/unnamed-chunk-16-1.png)

X-Y line plots: 


```r
ggplot(data = usa_gold_medals, aes(x = Year, y = count)) + geom_line() # ggplot2
```

![ ](figure/unnamed-chunk-17-1.png)

bar plots:


```r
# Create a dataframe of median number of gold medals by country
# note that the conversion to 'double' is because the median
# calculation had an error with 'count' is stored as integer
median_gold_medals <- medal_counts %>%
  dplyr::filter(Medal == "Gold") %>%
  mutate(count = as.double(count)) %>%
  group_by(NOC) %>%
  summarise(med = median(count))

ggplot(data = median_gold_medals[1:15, ], aes(x = NOC, y = med)) +
            geom_bar(stat="identity") # ggplot2
```

![ ](figure/unnamed-chunk-18-1.png)

boxplots:


```r
# Notice that here, you must explicitly convert numeric years to factors
ggplot(data = medal_counts_wide, aes(x = factor(Year), y = Gold)) +
            geom_boxplot() # ggplot2
```

![ ](figure/unnamed-chunk-19-1.png)

"trellis" plots:

```r
# Subset the data to North America countries for easier viewing
northern_hem <- medal_counts_wide %>%
  dplyr::filter(NOC %in% c("USA",
                           "CAN", # Canada
                           "CUB", # Cuba
                           "MEX")) # Mexico

ggplot(data = northern_hem, aes(x = Year, y = Gold)) + geom_point() +
            facet_wrap(~NOC) # ggplot2
```

![ ](figure/unnamed-chunk-20-1.png)

contour plots:


```r
data(volcano) # Load volcano contour data
volcano[1:10, 1:10] # Examine volcano dataset (first 10 rows and columns)
```

```
##       [,1] [,2] [,3] [,4] [,5] [,6] [,7] [,8] [,9] [,10]
##  [1,]  100  100  101  101  101  101  101  100  100   100
##  [2,]  101  101  102  102  102  102  102  101  101   101
##  [3,]  102  102  103  103  103  103  103  102  102   102
##  [4,]  103  103  104  104  104  104  104  103  103   103
##  [5,]  104  104  105  105  105  105  105  104  104   103
##  [6,]  105  105  105  106  106  106  106  105  105   104
##  [7,]  105  106  106  107  107  107  107  106  106   105
##  [8,]  106  107  107  108  108  108  108  107  107   106
##  [9,]  107  108  108  109  109  109  109  108  108   107
## [10,]  108  109  109  110  110  110  110  109  109   108
```

```r
volcano3d <- melt(volcano) # Use reshape2 package to melt the data into tidy form
head(volcano3d) # Examine volcano3d dataset (head)
```

```
##   Var1 Var2 value
## 1    1    1   100
## 2    2    1   101
## 3    3    1   102
## 4    4    1   103
## 5    5    1   104
## 6    6    1   105
```

```r
names(volcano3d) <- c("xvar", "yvar", "zvar") # Rename volcano3d columns

ggplot(data = volcano3d, aes(x = xvar, y = yvar, z = zvar)) +
            geom_contour() # ggplot2
```

![ ](figure/unnamed-chunk-21-1.png)

tile/image/level plots:


```r
ggplot(data = volcano3d, aes(x = xvar, y = yvar, z = zvar)) +
            geom_tile(aes(fill = zvar)) # ggplot2
```

![ ](figure/unnamed-chunk-22-1.png)

# Anatomy of `aes()`


```r
# NOT run
ggplot(data = , aes(x = , y = , color = , linetype = , shape = , size = ))
```

These four aesthetic parameters (`color`, `linetype`, `shape`, `size`) can be used to show variation in *kind* (categories) and variation in *degree* (numeric).

Parameters passed into `aes` should be *variables* in your dataset.

Parameters passed to `geom_xxx` outside of `aes` should *not* be related to your dataset -- they apply to the whole figure.


```r
ggplot(data = usa_all_medals, aes(x = Year, y = count)) +
            geom_line(aes(color = Medal))
```

![ ](figure/unnamed-chunk-24-1.png)

Note what happens when we specify the color parameter outside of the aesthetic operator. `ggplot2` views these specifications as invalid graphical parameters.


```r
ggplot(data = usa_all_medals, aes(x = Year, y = count)) +
            geom_point(color = Medal)
```

```
## Error in layer(data = data, mapping = mapping, stat = stat, geom = GeomPoint, : object 'Medal' not found
```

```r
ggplot(data = usa_all_medals, aes(x = Year, y = count)) +
            geom_point(color = "Medal")
```

```
## Error in grDevices::col2rgb(colour, TRUE): invalid color name 'Medal'
```

```r
ggplot(data = usa_all_medals, aes(x = Year, y = count)) +
            geom_point(color = "red")
```

![ ](figure/unnamed-chunk-25-1.png)

# Using aesthetics to highlight features

Differences in kind


```r
ggplot(data = northern_hem, aes(x = Year, y = Gold)) +
            geom_line(aes(linetype = NOC))
ggplot(data = northern_hem, aes(x = Year, y = Gold)) +
            geom_point(aes(shape = NOC, color = NOC))
```

![ ](figure/unnamed-chunk-26-1.png)![ ](figure/unnamed-chunk-26-2.png)

Differences in degree


```r
ggplot(data = northern_hem, aes(x = Year, y = Silver)) +
            geom_point(aes(color = Gold))
ggplot(data = northern_hem, aes(x = Year, y = Silver)) +
            geom_point(aes(size = Gold))
```

![ ](figure/unnamed-chunk-27-1.png)![ ](figure/unnamed-chunk-27-2.png)

Multiple non-coordinate aesthetics (differences in kind using color, degree using point size)


```r
ggplot(data = northern_hem, aes(x = Year, y = Silver)) +
            geom_point(aes(size = Gold, color = NOC))
```

![ ](figure/unnamed-chunk-28-1.png)

# Changing options in ggplot2

`ggplot` handles options in additional layers.

### Labels


```r
ggplot(data = usa_gold_medals, aes(x = Year, y = count)) + geom_point() +
  xlab(label = "Year") +
  ylab(label = "Number of Gold Medals Won") +
  ggtitle(label = "Cool Graph") # ggplot2
```

![ ](figure/unnamed-chunk-29-1.png)

### Axis and point scales


```r
ggplot(data = usa_gold_medals, aes(x = Year, y = count)) +
            geom_point() # ggplot2
ggplot(data = usa_gold_medals, aes(x = Year, y = count)) +
            geom_point(size=3) # ggplot2
ggplot(data = usa_gold_medals, aes(x = Year, y = count)) +
            geom_point(size=1) # ggplot2
```

![ ](figure/unnamed-chunk-30-1.png)![ ](figure/unnamed-chunk-30-2.png)![ ](figure/unnamed-chunk-30-3.png)

### Colors

```r
ggplot(data = usa_gold_medals, aes(x = Year, y = count)) +
            geom_point(color = colors()[11]) # ggplot2
ggplot(data = usa_gold_medals, aes(x = Year, y = count)) +
            geom_point(color = "red") # ggplot2
```

![ ](figure/unnamed-chunk-31-1.png)![ ](figure/unnamed-chunk-31-2.png)

### Point Styles and Widths


```r
ggplot(data = usa_gold_medals, aes(x = Year, y = count)) +
            geom_point(shape = 3) # ggplot2
ggplot(data = usa_gold_medals, aes(x = Year, y = count)) +
            geom_point(shape = "w") # ggplot2
ggplot(data = usa_gold_medals, aes(x = Year, y = count)) +
            geom_point(shape = "$", size=5) # ggplot2
```

![ ](figure/unnamed-chunk-32-1.png)![ ](figure/unnamed-chunk-32-2.png)![ ](figure/unnamed-chunk-32-3.png)

### Line Styles and Widths


```r
ggplot(data = usa_gold_medals, aes(x = Year, y = count)) +
            geom_line(linetype = 1) # ggplot2
ggplot(data = usa_gold_medals, aes(x = Year, y = count)) +
            geom_line(linetype = 2) # ggplot2
ggplot(data = usa_gold_medals, aes(x = Year, y = count)) +
            geom_line(linetype = 5, size = 2) # ggplot2
```

![ ](figure/unnamed-chunk-33-1.png)![ ](figure/unnamed-chunk-33-2.png)![ ](figure/unnamed-chunk-33-3.png)

# Fitted lines and curves with `ggplot2`


```r
ggplot(data = usa_gold_medals, aes(x = Year, y = count)) + geom_point()
```

![ ](figure/unnamed-chunk-34-1.png)

```r
# Add linear model (lm) smoother
ggplot(data = usa_gold_medals, aes(x = Year, y = count)) + geom_point() +
  geom_smooth(method = "lm")
```

![ ](figure/unnamed-chunk-34-2.png)

```r
# Add local linear model (loess) smoother, span of 0.75 (more smoothed)
ggplot(data = usa_gold_medals, aes(x = Year, y = count)) + geom_point() +
  geom_smooth(method = "loess", span = .75)
```

![ ](figure/unnamed-chunk-34-3.png)

```r
# Add local linear model (loess) smoother, span of 0.25 (less smoothed)
ggplot(data = usa_gold_medals, aes(x = Year, y = count)) + geom_point() +
  geom_smooth(method = "loess", span = .25)
```

![ ](figure/unnamed-chunk-34-4.png)

```r
# Add linear model (lm) smoother, no standard error shading
ggplot(data = usa_gold_medals, aes(x = Year, y = count)) + geom_point() +
  geom_smooth(method = "lm", se = FALSE)
```

![ ](figure/unnamed-chunk-34-5.png)

```r
# Add local linear model (loess) smoother, no standard error shading
ggplot(data = usa_gold_medals, aes(x = Year, y = count)) + geom_point() +
  geom_smooth(method = "loess", se = FALSE)
```

![ ](figure/unnamed-chunk-34-6.png)

```r
# Add a local linear (loess) smoother for each medal, no standard error shading
ggplot(data = usa_all_medals, aes(x = Year, y = count)) +
  geom_point(aes(color = Medal)) +
  geom_smooth(aes(color = Medal), method = "loess", se = FALSE)
```

![ ](figure/unnamed-chunk-34-7.png)

# Combining Multiple Plots

* `ggplot2` graphs can be combined using the *`grid.arrange()`* function in the **`gridExtra`** package


```r
# Initialize gridExtra library
library(gridExtra)

# Create 3 plots to combine in a table
plot1 <- ggplot(data = medal_counts_wide, aes(x = Year, y = Gold)) +
  geom_point(color = "gold") +
  geom_point(aes(x = Year, y = Silver), color = "gray") +
  geom_point(aes(x = Year, y = Bronze), color = "brown") +
  ylab("Medals")
plot2 <- ggplot(data = usa_all_medals, aes(x = Year, y = count)) +
      geom_line(aes(color = Medal))
plot3 <- ggplot(data = northern_hem, aes(x = Year, y = Gold)) +
      geom_line(aes(linetype = NOC))


# Call grid.arrange
grid.arrange(plot1, plot2, plot3, nrow=3, ncol = 1)
```

![ ](figure/unnamed-chunk-35-1.png)

# `patchwork`: Combining Multiple `ggplot2` plots

* The `patchwork` package may be used to combine multiple `ggplot2` plots using
  a small set of operators similar to the pipe.
* This requires less syntax than using `gridExtra` and allows complex
  arrangements to be built nearly effortlessly.


```r
# Install and initialize patchwork library
#devtools::install_github("thomasp85/patchwork")
library(patchwork)

# Create 3 plots to combine in a table
plot1 <- ggplot(data = medal_counts_wide, aes(x = Year, y = Gold)) +
  geom_point(color = "gold") +
  geom_point(aes(x = Year, y = Silver), color = "gray") +
  geom_point(aes(x = Year, y = Bronze), color = "brown") +
  ylab("Medals")
plot2 <- ggplot(data = usa_all_medals, aes(x = Year, y = count)) +
      geom_line(aes(color = Medal))
plot3 <- ggplot(data = northern_hem, aes(x = Year, y = Gold)) +
      geom_line(aes(linetype = NOC))


# use the patchwork operators
# stack plots horizontally
plot1 + plot2 + plot3
```

![ ](figure/unnamed-chunk-36-1.png)

```r
# stack plots vertically
plot1 / plot2 / plot3
```

![ ](figure/unnamed-chunk-36-2.png)

```r
# side-by-side plots with third plot below
(plot1 | plot2) / plot3
```

![ ](figure/unnamed-chunk-36-3.png)

```r
# side-by-side plots with a space in between, and a third plot below
(plot1 | plot_spacer() | plot2) / plot3
```

![ ](figure/unnamed-chunk-36-4.png)

```r
# stack plots vertically and alter with a single "gg_theme"
(plot1 / plot2 / plot3) & theme_bw()
```

![ ](figure/unnamed-chunk-36-5.png)

Feel free to explore more at [https://github.com/thomasp85/patchwork](https://github.com/thomasp85/patchwork).

# Exporting

Two basic image types:

### **Raster/Bitmap** (.png, .jpeg)

Every pixel of a plot contains its own separate coding; not so great if you want to resize the image


```r
jpeg(filename = "example.jpg", width=, height=)
plot(x,y)
dev.off()
```

### **Vector** (.pdf, .ps)

Every element of a plot is encoded with a function that gives its coding conditional on several factors; great for resizing


```r
# NOT run
pdf(file = "example.pdf", width=, height=)
plot(x,y)
dev.off()
```

### Exporting with `ggplot`


```r
# NOT run

# Assume we saved our plot is an object called example.plot

ggsave(filename = "example.pdf", plot = example.plot, scale = , width = ,
       height = )
```


# Breakout

These questions ask you to work with the gapminder dataset.

### Basics

1) Plot a histogram of life expectancy. 

2) Plot the life expectancy against gdpPercap. 

3) Clean up your scatterplot with a title and axis labels. Output it as a PDF and see if you'd be comfortable with including it in a report/paper.

4) Make a boxplot of life expectancy conditional on year.

### Using the ideas

5) Create a trellis plot of life expectancy by gdpPercap scatterplots, one subplot per year. Use a 3x4 layout of panels in the plot.

6) Plot life expectancy versus gdpPercap. Now plot so that different continents are in different colors. Use `scale_x_continuous()` to set the x-axis limits to be in the range from 100 to 50000.

7) Figure out how to use the log-scale for gdpPercap, without manually calculating the log values.

### Advanced

8) Create a "trellis" plot where, for a given year, each panel uses a) hollow circles to plot lifeExp as a function of log(gdpPercap), and b) a red loess smoother without standard errors to plot the trend. Turn off the grey background. Figure out how to use partially-transparent points to reduce the effect of the overplotting of points.

