% R bootcamp, Module 4: Calculations
% January 2019, UC Berkeley
% Chris Paciorek



# Reminder: Vectorized calculations and comparisons

At the core of R is the idea of doing calculations on entire vectors.


```r
gdpTotal <- gap$gdpPercap * gap$pop

gdpSubset <- gap2007$gdpPercap[1:10]

gdpSubset >= 30000
```

```
##  [1] FALSE FALSE FALSE FALSE FALSE  TRUE  TRUE FALSE FALSE  TRUE
```

```r
vec1 <- rnorm(5)
vec2 <- rnorm(5)
vec1 > vec2
```

```
## [1]  TRUE FALSE  TRUE  TRUE  TRUE
```

```r
vec1 == vec2
```

```
## [1] FALSE FALSE FALSE FALSE FALSE
```

```r
vec1 != vec2
```

```
## [1] TRUE TRUE TRUE TRUE TRUE
```

```r
## careful: 
vec1 = vec2
identical(vec1, vec2)
```

```
## [1] TRUE
```

```r
## using 'or'
gdpSubset >= 100000 | gdpSubset <= 1000
```

```
##  [1]  TRUE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
```

```r
## using 'and'
gap$gdpPercap[1:10] >= 100000 & gap$continent[1:10] == "Asia"
```

```
##  [1] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
```

An important related concept is that of recycling

```r
vec10 <- sample(1:10, 10, replace = TRUE)
vec3 <- sample(1:10, 3, replace = TRUE)
vec5 <- sample(1:10, 5, replace = TRUE)
vec10
```

```
##  [1] 10  9  4  9  9  3 10  4  2  1
```

```r
vec3
```

```
## [1] 1 3 2
```

```r
vec5
```

```
## [1] 7 8 3 2 4
```

```r
vec10 + vec5
```

```
##  [1] 17 17  7 11 13 10 18  7  4  5
```

```r
vec10 + vec3
```

```
## Warning in vec10 + vec3: longer object length is not a multiple of shorter
## object length
```

```
##  [1] 11 12  6 10 12  5 11  7  4  2
```

**Question**: Tell me what's going on. What choices were made by the R developers?

# Vectorized calculations

As we've seen, R has many functions that allow you to operate on each element of a vector all at once.


```r
vals <- rnorm(1000)
chi2vals <- vals^2
chi2_df1000 <- sum(chi2vals)
# imagine if the code above were a loop, or three separate loops
```

Advantages:

* much faster than looping
* easier to code
* easier to read and understand the code

Sometimes there are surprises in terms of what is fast, as well as tricks for vectorizing things in unexpected ways:

```r
vals <- rnorm(1e6)
system.time(trunc <- ifelse(vals > 0, vals, 0))
```

```
##    user  system elapsed 
##   0.132   0.004   0.137
```

```r
system.time(vals <- vals * (vals > 0))
```

```
##    user  system elapsed 
##   0.008   0.000   0.006
```

**Question**: What am I doing with `vals * (vals > 0)` ? What happens behind the scenes in R?

If you use a trick like this, having a comment in your code is a good idea.

Lots of functions in R are vectorized, such as some we've already used.


```r
tmp <- as.character(gap$year)
gap$year2 <- substring(tmp, 3, 4)
head(gap$year2)
```

```
## [1] "52" "57" "62" "67" "72" "77"
```

Question: Note the code above runs and the syntax is perfectly good R syntax, but in terms of what it does, there is a bug in it. See if you can see what it is.

# Linear algebra 

R can do essentially any linear algebra you need. It uses system-level packages called BLAS (basic linear algebra subroutines) and LAPACK (linear algebra package). Note that these calculations will be essentially as fast as if you wrote C code because R just calls C and Fortran routines to do the calculations.

The BLAS that comes with R is fairly slow. It's possible to use a faster BLAS, as well as one that uses multiple cores automatically. This can in some cases give you an order of magnitude speedup if your work involves a lot of matrix manipulations/linear algebra. More details in Module 10.


# Vectorized vector/matrix calculations

Recall that `+`, `-`,`*`, `/` do vectorized calculations:


```r
A <- matrix(1:9, 3)
B <- matrix(seq(4,36, by = 4), 3)

A + B
```

```
##      [,1] [,2] [,3]
## [1,]    5   20   35
## [2,]   10   25   40
## [3,]   15   30   45
```

```r
A + B[ , 1]
```

```
##      [,1] [,2] [,3]
## [1,]    5    8   11
## [2,]   10   13   16
## [3,]   15   18   21
```

```r
A * B
```

```
##      [,1] [,2] [,3]
## [1,]    4   64  196
## [2,]   16  100  256
## [3,]   36  144  324
```

```r
A * B[ , 1]
```

```
##      [,1] [,2] [,3]
## [1,]    4   16   28
## [2,]   16   40   64
## [3,]   36   72  108
```

Matrix/vector multiplication


```r
A %*% B[ , 1]
```

```
##      [,1]
## [1,]  120
## [2,]  144
## [3,]  168
```

```r
A %*% B
```

```
##      [,1] [,2] [,3]
## [1,]  120  264  408
## [2,]  144  324  504
## [3,]  168  384  600
```

```r
identical(t(A)%*%A, crossprod(A))
```

```
## [1] TRUE
```

Some decompositions


```r
## next 3 lines generate a positive definite matrix
library(fields)
times <- seq(0, 1, length = 100)
R <- exp(-rdist(times) / 0.2) # a correlation matrix
######################################################
e <- eigen(R)
range(e$values)
```

```
## [1]  0.02525338 32.85537225
```

```r
e$vectors[ , 1]
```

```
##   [1] 0.05195413 0.05448567 0.05698864 0.05946173 0.06190363 0.06431308
##   [7] 0.06668879 0.06902954 0.07133409 0.07360123 0.07582978 0.07801856
##  [13] 0.08016643 0.08227226 0.08433494 0.08635341 0.08832659 0.09025345
##  [19] 0.09213298 0.09396420 0.09574615 0.09747789 0.09915851 0.10078713
##  [25] 0.10236291 0.10388500 0.10535262 0.10676499 0.10812137 0.10942106
##  [31] 0.11066337 0.11184765 0.11297327 0.11403965 0.11504623 0.11599249
##  [37] 0.11687791 0.11770205 0.11846447 0.11916476 0.11980256 0.12037754
##  [43] 0.12088940 0.12133786 0.12172270 0.12204370 0.12230071 0.12249358
##  [49] 0.12262222 0.12268655 0.12268655 0.12262222 0.12249358 0.12230071
##  [55] 0.12204370 0.12172270 0.12133786 0.12088940 0.12037754 0.11980256
##  [61] 0.11916476 0.11846447 0.11770205 0.11687791 0.11599249 0.11504623
##  [67] 0.11403965 0.11297327 0.11184765 0.11066337 0.10942106 0.10812137
##  [73] 0.10676499 0.10535262 0.10388500 0.10236291 0.10078713 0.09915851
##  [79] 0.09747789 0.09574615 0.09396420 0.09213298 0.09025345 0.08832659
##  [85] 0.08635341 0.08433494 0.08227226 0.08016643 0.07801856 0.07582978
##  [91] 0.07360123 0.07133409 0.06902954 0.06668879 0.06431308 0.06190363
##  [97] 0.05946173 0.05698864 0.05448567 0.05195413
```

```r
sv <- svd(R)
U <- chol(R)

devs <- rnorm(100)
Rinvb <- solve(R, devs)  # R^{-1} b
Rinv <- solve(R) # R^{-1} -- try to avoid this
```


# Pre-allocation

This is slow.

```r
vals <- 0
n <- 50000
system.time({
for(i in 1:n)
      vals <- c(vals, i)
})
```

```
##    user  system elapsed 
##   2.892   0.008   2.900
```

The same holds for using `rbind()`, `cbind()`, or adding to a list, one element at a time.


**Question**: Thoughts on why this are so slow? Think about what R might be doing behind the scenes

# The answer is to pre-allocate memory

This is not so slow. (Please ignore the for-loop hypocrisy and the fact that I could do this as `vals <- 1:n`.)


```r
n <- 50000
system.time({
vals <- rep(0, n)
# alternatively: vals <- as.numeric(NA); length(vals) <- n
for(i in 1:n)
      vals[i] <- i
})
```

```
##    user  system elapsed 
##   0.036   0.000   0.037
```

Here's how to pre-allocate an empty list: 

```r
vals <- list(); length(vals) <- n
head(vals)
```

```
## [[1]]
## NULL
## 
## [[2]]
## NULL
## 
## [[3]]
## NULL
## 
## [[4]]
## NULL
## 
## [[5]]
## NULL
## 
## [[6]]
## NULL
```

# apply

Some functions aren't vectorized, or you may want to use a function on every row or column of a matrix/data frame, every element of a list, etc.

For this we use the `apply()` family of functions.


```r
mat <- matrix(rnorm(100*1000), nr = 100)
row_min <- apply(mat, MARGIN = 1, FUN = min)
col_max <- apply(mat, MARGIN = 2, FUN = max)
```

There are actually some even faster specialized functions:

```r
row_mean <- rowMeans(mat)
col_sum <- colSums(mat)
```

# `lapply()` and `sapply()`


```r
myList <- list(rnorm(3), rnorm(3), rnorm(5))
lapply(myList, min)
```

```
## [[1]]
## [1] -0.8286919
## 
## [[2]]
## [1] -1.543495
## 
## [[3]]
## [1] -0.07627372
```

```r
sapply(myList, min)
```

```
## [1] -0.82869190 -1.54349527 -0.07627372
```

Note that we don't generally want to use `apply()` on a data frame. 

You can use `lapply()` and `sapply()` on regular vectors, such as vectors of indices, which can come in handy, though this is a silly example:

```r
sapply(1:10, function(x) x^2)
```

```
##  [1]   1   4   9  16  25  36  49  64  81 100
```


# And more `apply()`s

There are a bunch of `apply()` variants, as well as parallelized versions of them:

* `tapply()`, `vapply()`, `mapply()`, `rapply()`, `eapply()`
* for parallelized versions see Module 10 or `?clusterApply`

# Tabulation 

- Sometimes we need to do some basic checking for the number of observations or types of observations in our dataset
- To do this quickly and easily, `table()` is our friend


```r
tbl <- table(gap$country, gap$continent)
tbl
```

```
##                           
##                            Africa Americas Asia Europe Oceania
##   Afghanistan                   0        0   12      0       0
##   Albania                       0        0    0     12       0
##   Algeria                      12        0    0      0       0
##   Angola                       12        0    0      0       0
##   Argentina                     0       12    0      0       0
##   Australia                     0        0    0      0      12
##   Austria                       0        0    0     12       0
##   Bahrain                       0        0   12      0       0
##   Bangladesh                    0        0   12      0       0
##   Belgium                       0        0    0     12       0
##   Benin                        12        0    0      0       0
##   Bolivia                       0       12    0      0       0
##   Bosnia and Herzegovina        0        0    0     12       0
##   Botswana                     12        0    0      0       0
##   Brazil                        0       12    0      0       0
##   Bulgaria                      0        0    0     12       0
##   Burkina Faso                 12        0    0      0       0
##   Burundi                      12        0    0      0       0
##   Cambodia                      0        0   12      0       0
##   Cameroon                     12        0    0      0       0
##   Canada                        0       12    0      0       0
##   Central African Republic     12        0    0      0       0
##   Chad                         12        0    0      0       0
##   Chile                         0       12    0      0       0
##   China                         0        0   12      0       0
##   Colombia                      0       12    0      0       0
##   Comoros                      12        0    0      0       0
##   Congo Dem. Rep.              12        0    0      0       0
##   Congo Rep.                   12        0    0      0       0
##   Costa Rica                    0       12    0      0       0
##   Cote d'Ivoire                12        0    0      0       0
##   Croatia                       0        0    0     12       0
##   Cuba                          0       12    0      0       0
##   Czech Republic                0        0    0     12       0
##   Denmark                       0        0    0     12       0
##   Djibouti                     12        0    0      0       0
##   Dominican Republic            0       12    0      0       0
##   Ecuador                       0       12    0      0       0
##   Egypt                        12        0    0      0       0
##   El Salvador                   0       12    0      0       0
##   Equatorial Guinea            12        0    0      0       0
##   Eritrea                      12        0    0      0       0
##   Ethiopia                     12        0    0      0       0
##   Finland                       0        0    0     12       0
##   France                        0        0    0     12       0
##   Gabon                        12        0    0      0       0
##   Gambia                       12        0    0      0       0
##   Germany                       0        0    0     12       0
##   Ghana                        12        0    0      0       0
##   Greece                        0        0    0     12       0
##   Guatemala                     0       12    0      0       0
##   Guinea                       12        0    0      0       0
##   Guinea-Bissau                12        0    0      0       0
##   Haiti                         0       12    0      0       0
##   Honduras                      0       12    0      0       0
##   Hong Kong China               0        0   12      0       0
##   Hungary                       0        0    0     12       0
##   Iceland                       0        0    0     12       0
##   India                         0        0   12      0       0
##   Indonesia                     0        0   12      0       0
##   Iran                          0        0   12      0       0
##   Iraq                          0        0   12      0       0
##   Ireland                       0        0    0     12       0
##   Israel                        0        0   12      0       0
##   Italy                         0        0    0     12       0
##   Jamaica                       0       12    0      0       0
##   Japan                         0        0   12      0       0
##   Jordan                        0        0   12      0       0
##   Kenya                        12        0    0      0       0
##   Korea Dem. Rep.               0        0   12      0       0
##   Korea Rep.                    0        0   12      0       0
##   Kuwait                        0        0   12      0       0
##   Lebanon                       0        0   12      0       0
##   Lesotho                      12        0    0      0       0
##   Liberia                      12        0    0      0       0
##   Libya                        12        0    0      0       0
##   Madagascar                   12        0    0      0       0
##   Malawi                       12        0    0      0       0
##   Malaysia                      0        0   12      0       0
##   Mali                         12        0    0      0       0
##   Mauritania                   12        0    0      0       0
##   Mauritius                    12        0    0      0       0
##   Mexico                        0       12    0      0       0
##   Mongolia                      0        0   12      0       0
##   Montenegro                    0        0    0     12       0
##   Morocco                      12        0    0      0       0
##   Mozambique                   12        0    0      0       0
##   Myanmar                       0        0   12      0       0
##   Namibia                      12        0    0      0       0
##   Nepal                         0        0   12      0       0
##   Netherlands                   0        0    0     12       0
##   New Zealand                   0        0    0      0      12
##   Nicaragua                     0       12    0      0       0
##   Niger                        12        0    0      0       0
##   Nigeria                      12        0    0      0       0
##   Norway                        0        0    0     12       0
##   Oman                          0        0   12      0       0
##   Pakistan                      0        0   12      0       0
##   Panama                        0       12    0      0       0
##   Paraguay                      0       12    0      0       0
##   Peru                          0       12    0      0       0
##   Philippines                   0        0   12      0       0
##   Poland                        0        0    0     12       0
##   Portugal                      0        0    0     12       0
##   Puerto Rico                   0       12    0      0       0
##   Reunion                      12        0    0      0       0
##   Romania                       0        0    0     12       0
##   Rwanda                       12        0    0      0       0
##   Sao Tome and Principe        12        0    0      0       0
##   Saudi Arabia                  0        0   12      0       0
##   Senegal                      12        0    0      0       0
##   Serbia                        0        0    0     12       0
##   Sierra Leone                 12        0    0      0       0
##   Singapore                     0        0   12      0       0
##   Slovak Republic               0        0    0     12       0
##   Slovenia                      0        0    0     12       0
##   Somalia                      12        0    0      0       0
##   South Africa                 12        0    0      0       0
##   Spain                         0        0    0     12       0
##   Sri Lanka                     0        0   12      0       0
##   Sudan                        12        0    0      0       0
##   Swaziland                    12        0    0      0       0
##   Sweden                        0        0    0     12       0
##   Switzerland                   0        0    0     12       0
##   Syria                         0        0   12      0       0
##   Taiwan                        0        0   12      0       0
##   Tanzania                     12        0    0      0       0
##   Thailand                      0        0   12      0       0
##   Togo                         12        0    0      0       0
##   Trinidad and Tobago           0       12    0      0       0
##   Tunisia                      12        0    0      0       0
##   Turkey                        0        0    0     12       0
##   Uganda                       12        0    0      0       0
##   United Kingdom                0        0    0     12       0
##   United States                 0       12    0      0       0
##   Uruguay                       0       12    0      0       0
##   Venezuela                     0       12    0      0       0
##   Vietnam                       0        0   12      0       0
##   West Bank and Gaza            0        0   12      0       0
##   Yemen Rep.                    0        0   12      0       0
##   Zambia                       12        0    0      0       0
##   Zimbabwe                     12        0    0      0       0
```

```r
rowSums(tbl)
```

```
##              Afghanistan                  Albania                  Algeria 
##                       12                       12                       12 
##                   Angola                Argentina                Australia 
##                       12                       12                       12 
##                  Austria                  Bahrain               Bangladesh 
##                       12                       12                       12 
##                  Belgium                    Benin                  Bolivia 
##                       12                       12                       12 
##   Bosnia and Herzegovina                 Botswana                   Brazil 
##                       12                       12                       12 
##                 Bulgaria             Burkina Faso                  Burundi 
##                       12                       12                       12 
##                 Cambodia                 Cameroon                   Canada 
##                       12                       12                       12 
## Central African Republic                     Chad                    Chile 
##                       12                       12                       12 
##                    China                 Colombia                  Comoros 
##                       12                       12                       12 
##          Congo Dem. Rep.               Congo Rep.               Costa Rica 
##                       12                       12                       12 
##            Cote d'Ivoire                  Croatia                     Cuba 
##                       12                       12                       12 
##           Czech Republic                  Denmark                 Djibouti 
##                       12                       12                       12 
##       Dominican Republic                  Ecuador                    Egypt 
##                       12                       12                       12 
##              El Salvador        Equatorial Guinea                  Eritrea 
##                       12                       12                       12 
##                 Ethiopia                  Finland                   France 
##                       12                       12                       12 
##                    Gabon                   Gambia                  Germany 
##                       12                       12                       12 
##                    Ghana                   Greece                Guatemala 
##                       12                       12                       12 
##                   Guinea            Guinea-Bissau                    Haiti 
##                       12                       12                       12 
##                 Honduras          Hong Kong China                  Hungary 
##                       12                       12                       12 
##                  Iceland                    India                Indonesia 
##                       12                       12                       12 
##                     Iran                     Iraq                  Ireland 
##                       12                       12                       12 
##                   Israel                    Italy                  Jamaica 
##                       12                       12                       12 
##                    Japan                   Jordan                    Kenya 
##                       12                       12                       12 
##          Korea Dem. Rep.               Korea Rep.                   Kuwait 
##                       12                       12                       12 
##                  Lebanon                  Lesotho                  Liberia 
##                       12                       12                       12 
##                    Libya               Madagascar                   Malawi 
##                       12                       12                       12 
##                 Malaysia                     Mali               Mauritania 
##                       12                       12                       12 
##                Mauritius                   Mexico                 Mongolia 
##                       12                       12                       12 
##               Montenegro                  Morocco               Mozambique 
##                       12                       12                       12 
##                  Myanmar                  Namibia                    Nepal 
##                       12                       12                       12 
##              Netherlands              New Zealand                Nicaragua 
##                       12                       12                       12 
##                    Niger                  Nigeria                   Norway 
##                       12                       12                       12 
##                     Oman                 Pakistan                   Panama 
##                       12                       12                       12 
##                 Paraguay                     Peru              Philippines 
##                       12                       12                       12 
##                   Poland                 Portugal              Puerto Rico 
##                       12                       12                       12 
##                  Reunion                  Romania                   Rwanda 
##                       12                       12                       12 
##    Sao Tome and Principe             Saudi Arabia                  Senegal 
##                       12                       12                       12 
##                   Serbia             Sierra Leone                Singapore 
##                       12                       12                       12 
##          Slovak Republic                 Slovenia                  Somalia 
##                       12                       12                       12 
##             South Africa                    Spain                Sri Lanka 
##                       12                       12                       12 
##                    Sudan                Swaziland                   Sweden 
##                       12                       12                       12 
##              Switzerland                    Syria                   Taiwan 
##                       12                       12                       12 
##                 Tanzania                 Thailand                     Togo 
##                       12                       12                       12 
##      Trinidad and Tobago                  Tunisia                   Turkey 
##                       12                       12                       12 
##                   Uganda           United Kingdom            United States 
##                       12                       12                       12 
##                  Uruguay                Venezuela                  Vietnam 
##                       12                       12                       12 
##       West Bank and Gaza               Yemen Rep.                   Zambia 
##                       12                       12                       12 
##                 Zimbabwe 
##                       12
```

```r
all(rowSums(tbl) == 12)
```

```
## [1] TRUE
```


# Discretization

You may need to discretize a continuous variable [or a discrete variable with many levels], e.g., by life expectancy:

```r
gap2007$lifeExpBin <- cut(gap2007$lifeExp, breaks = c(0, 40, 50, 60, 70, 75, 80, Inf))
tbl <- table(gap2007$continent, gap2007$lifeExpBin)
round( prop.table(tbl, margin = 1), 2 )
```

```
##           
##            (0,40] (40,50] (50,60] (60,70] (70,75] (75,80] (80,Inf]
##   Africa     0.02    0.33    0.42    0.10    0.12    0.02     0.00
##   Americas   0.00    0.00    0.00    0.12    0.48    0.36     0.04
##   Asia       0.00    0.03    0.06    0.24    0.39    0.18     0.09
##   Europe     0.00    0.00    0.00    0.00    0.27    0.50     0.23
##   Oceania    0.00    0.00    0.00    0.00    0.00    0.00     1.00
```

# Stratified analyses I
Often we want to do individual analyses within subsets or clusters of our data.

As a first step, we might want to just split our dataset by a stratifying variable.  


```r
subsets <- split(gap,  gap$year)
length(subsets)
```

```
## [1] 12
```

```r
dim(subsets[['2007']])
```

```
## [1] 142   7
```

```r
par(mfrow = c(1,2))
plot(lifeExp ~ gdpPercap, data = subsets[['1952']], main = '1952')
abline(h = 0, col = 'grey')
plot(lifeExp ~ gdpPercap, data = subsets[['2007']], main = '2007')
abline(h = 0, col = 'grey')
```

![](figure/unnamed-chunk-17-1.png)

Obviously, we'd want to iterate to improve that plot given the outlier.

# Stratified analyses II

Often we want to do individual analyses within subsets or clusters of our data. R has a variety of tools for this; for now we'll look at `aggregate()` and `by()`. These are wrappers of `tapply()`. 


```r
gmSmall <- gap[ , c('lifeExp', 'gdpPercap')]  # reduce to only numeric columns
aggregate(gmSmall, by = list(year = gap$year), FUN = median, na.rm = TRUE) # na.rm not needed here but illustrates use of additional arguments to FUN
```

```
##    year lifeExp gdpPercap
## 1  1952 45.1355  1968.528
## 2  1957 48.3605  2173.220
## 3  1962 50.8810  2335.440
## 4  1967 53.8250  2678.335
## 5  1972 56.5300  3339.129
## 6  1977 59.6720  3798.609
## 7  1982 62.4415  4216.228
## 8  1987 65.8340  4280.300
## 9  1992 67.7030  4386.086
## 10 1997 69.3940  4781.825
## 11 2002 70.8255  5319.805
## 12 2007 71.9355  6124.371
```

```r
aggregate(lifeExp ~ year + continent, data = gap, FUN = median)
```

```
##    year continent lifeExp
## 1  1952    Africa 38.8330
## 2  1957    Africa 40.5925
## 3  1962    Africa 42.6305
## 4  1967    Africa 44.6985
## 5  1972    Africa 47.0315
## 6  1977    Africa 49.2725
## 7  1982    Africa 50.7560
## 8  1987    Africa 51.6395
## 9  1992    Africa 52.4290
## 10 1997    Africa 52.7590
## 11 2002    Africa 51.2355
## 12 2007    Africa 52.9265
## 13 1952  Americas 54.7450
## 14 1957  Americas 56.0740
## 15 1962  Americas 58.2990
## 16 1967  Americas 60.5230
## 17 1972  Americas 63.4410
## 18 1977  Americas 66.3530
## 19 1982  Americas 67.4050
## 20 1987  Americas 69.4980
## 21 1992  Americas 69.8620
## 22 1997  Americas 72.1460
## 23 2002  Americas 72.0470
## 24 2007  Americas 72.8990
## 25 1952      Asia 44.8690
## 26 1957      Asia 48.2840
## 27 1962      Asia 49.3250
## 28 1967      Asia 53.6550
## 29 1972      Asia 56.9500
## 30 1977      Asia 60.7650
## 31 1982      Asia 63.7390
## 32 1987      Asia 66.2950
## 33 1992      Asia 68.6900
## 34 1997      Asia 70.2650
## 35 2002      Asia 71.0280
## 36 2007      Asia 72.3960
## 37 1952    Europe 65.9000
## 38 1957    Europe 67.6500
## 39 1962    Europe 69.5250
## 40 1967    Europe 70.6100
## 41 1972    Europe 70.8850
## 42 1977    Europe 72.3350
## 43 1982    Europe 73.4900
## 44 1987    Europe 74.8150
## 45 1992    Europe 75.4510
## 46 1997    Europe 76.1160
## 47 2002    Europe 77.5365
## 48 2007    Europe 78.6085
## 49 1952   Oceania 69.2550
## 50 1957   Oceania 70.2950
## 51 1962   Oceania 71.0850
## 52 1967   Oceania 71.3100
## 53 1972   Oceania 71.9100
## 54 1977   Oceania 72.8550
## 55 1982   Oceania 74.2900
## 56 1987   Oceania 75.3200
## 57 1992   Oceania 76.9450
## 58 1997   Oceania 78.1900
## 59 2002   Oceania 79.7400
## 60 2007   Oceania 80.7195
```

```r
agg <- aggregate(lifeExp ~ year + continent , data = gap, FUN = median)
xtabs(lifeExp ~ ., data = agg)
```

```
##       continent
## year    Africa Americas    Asia  Europe Oceania
##   1952 38.8330  54.7450 44.8690 65.9000 69.2550
##   1957 40.5925  56.0740 48.2840 67.6500 70.2950
##   1962 42.6305  58.2990 49.3250 69.5250 71.0850
##   1967 44.6985  60.5230 53.6550 70.6100 71.3100
##   1972 47.0315  63.4410 56.9500 70.8850 71.9100
##   1977 49.2725  66.3530 60.7650 72.3350 72.8550
##   1982 50.7560  67.4050 63.7390 73.4900 74.2900
##   1987 51.6395  69.4980 66.2950 74.8150 75.3200
##   1992 52.4290  69.8620 68.6900 75.4510 76.9450
##   1997 52.7590  72.1460 70.2650 76.1160 78.1900
##   2002 51.2355  72.0470 71.0280 77.5365 79.7400
##   2007 52.9265  72.8990 72.3960 78.6085 80.7195
```

Notice the 'long' vs. 'wide' formats. You'll see more about that sort of thing in Module 6.


# Stratified analyses III

`aggregate()` works fine when the output is univariate, but what about more complicated analyses than computing the median, such as fitting a set of regressions?


```r
out <- by(gap, gap$year, 
    function(sub) {
      lm(lifeExp ~ log(gdpPercap), data = sub)
    }          
)
length(out)
```

```
## [1] 12
```

```r
summary(out[['2007']])
```

```
## 
## Call:
## lm(formula = lifeExp ~ log(gdpPercap), data = sub)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -25.947  -2.661   1.215   4.469  13.115 
## 
## Coefficients:
##                Estimate Std. Error t value Pr(>|t|)    
## (Intercept)      4.9496     3.8577   1.283    0.202    
## log(gdpPercap)   7.2028     0.4423  16.283   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 7.122 on 140 degrees of freedom
## Multiple R-squared:  0.6544,	Adjusted R-squared:  0.652 
## F-statistic: 265.2 on 1 and 140 DF,  p-value: < 2.2e-16
```

```r
summary(out[['1952']])
```

```
## 
## Call:
## lm(formula = lifeExp ~ log(gdpPercap), data = sub)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -28.9571  -5.7319   0.7517   6.5770  13.7361 
## 
## Coefficients:
##                Estimate Std. Error t value Pr(>|t|)    
## (Intercept)    -17.8457     5.0668  -3.522 0.000578 ***
## log(gdpPercap)   8.8298     0.6626  13.326  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 8.146 on 140 degrees of freedom
## Multiple R-squared:  0.5592,	Adjusted R-squared:  0.556 
## F-statistic: 177.6 on 1 and 140 DF,  p-value: < 2.2e-16
```

# Sorting

`sort()` applied to a vector does what you expect.

Sorting a matrix or dataframe based on one or more columns is a somewhat manual process, but once you get the hang of it, it's not bad.


```r
ord <- order(gap$year, gap$lifeExp, decreasing = TRUE)
ord[1:5]
```

```
## [1]  804  672  696 1488   72
```

```r
gm_ord <- gap[ord, ]
head(gm_ord)
```

```
##              country year       pop continent lifeExp gdpPercap year2
## 804            Japan 2007 127467972      Asia  82.603  31656.07    07
## 672  Hong Kong China 2007   6980412      Asia  82.208  39724.98    07
## 696          Iceland 2007    301931    Europe  81.757  36180.79    07
## 1488     Switzerland 2007   7554661    Europe  81.701  37506.42    07
## 72         Australia 2007  20434176   Oceania  81.235  34435.37    07
## 1428           Spain 2007  40448191    Europe  80.941  28821.06    07
```

You could of course write your own *sort* function that uses `order()`. More in Module 6.

# Merging Data

We often need to combine data across multiple data frames, merging on common fields (i.e., *keys*). In database terminology, this is a *join* operation.

Suppose that our dataset did not have 'continent' in it, but that we had a separate data frame that matches country to continent.


```r
# ignore the 'wizard' behind the curtain...
c2c <- unique(gap[ , c('country', 'continent')])
gapSave <- gap
gap <- gap[ , -which(names(gap) == "continent")]
```

Now let's add the continent info in:


```r
head(c2c)
```

```
##        country continent
## 1  Afghanistan      Asia
## 13     Albania    Europe
## 25     Algeria    Africa
## 37      Angola    Africa
## 49   Argentina  Americas
## 61   Australia   Oceania
```

```r
head(gap)
```

```
##       country year      pop lifeExp gdpPercap year2
## 1 Afghanistan 1952  8425333  28.801  779.4453    52
## 2 Afghanistan 1957  9240934  30.332  820.8530    57
## 3 Afghanistan 1962 10267083  31.997  853.1007    62
## 4 Afghanistan 1967 11537966  34.020  836.1971    67
## 5 Afghanistan 1972 13079460  36.088  739.9811    72
## 6 Afghanistan 1977 14880372  38.438  786.1134    77
```

```r
gap <- merge(gap, c2c, by.x = 'country', by.y = 'country',
                   all.x = TRUE, all.y = FALSE)

dim(gapSave)
```

```
## [1] 1704    7
```

```r
dim(gap)
```

```
## [1] 1704    7
```

```r
identical(gapSave, gap)
```

```
## [1] FALSE
```

```r
identical(gapSave, gap[ , names(gapSave)])
```

```
## [1] TRUE
```

What's the deal with the `all.x` and `all.y` ?  We can tell R whether we want to keep all of the `x` observations, all the `y` observations, or neither, or both, when there may be rows in either of the datasets that don't match the other dataset.

# Breakout

### Basics

1) Create a vector that concatenates the country and year to create a 'country-year' variable in a vectorized way using the string processing functions.

2) Use `table()` to figure out the number of countries available for each continent.

### Using the ideas

3) Explain the steps of what this code is doing: `tmp <- gap[ , -which(names(gap) == "continent")]`.

4) Compute the number of NAs in each column of the gapminder dataset using `sapply()` and making use of the `is.na()` function.

5) Discretize gdpPercap into some bins and create a gdpPercap_binned variable. Count the number of values in each bin.

6) Create a boxplot of life expectancy by binned gdpPercap.

7) Sort the dataset and find the shortest life expectancy value. Now consider the use of `which.min()` and why using that should be much quicker with large datasets.
 
8) Create a dataframe that has the total population across all the countries for each year.

9) Merge the info from problem 8 back into the original gapminder dataset. Now plot life expectancy as a function of world population. 

10)  Suppose we have two categorical variables and we conduct a hypothesis test of independence. The chi-square statistic is: 

$$
\chi^2 = \sum_{i=1}^{n}\sum_{j=1}^{m} \frac{(y_{ij} - e_{ij})^2}{e_{ij}}, 
$$ 

where $e_{ij} = \frac{y_{i\cdot} y_{\cdot j}}{y_{\cdot \cdot}}$, with $y_{i\cdot}$ the sum of the values in the i'th row, $y_{\cdot j}$ the sum of values in the j'th column, and $y_{\cdot\cdot}$ the sum of all the values. Suppose I give you a matrix in R with the $y_{ij}$ values. 

You can generate a test matrix as: 

```r
y <- matrix(sample(1:10, 12, replace = TRUE), 
nrow = 3, ncol = 4)
```

Compute the statistic without *any* loops as follows:

a. First, assume you have the *e* matrix. How do you compute the statistic without loops as a function of `y` and `e`?
b. How can you construct the *e* matrix? Hint: the numerator of *e* is just an *outer product* for which the `outer()` function can be used.

### Advanced 

11) For each combination of year and continent, find the 95th percentile of life expectancy. 

12) Here's a cool trick to pull off a particular element of a list of lists:


```r
params <- list(a = list(mn = 7, sd = 3), b = list(mn = 6,sd = 1), 
  c = list(mn = 2, sd = 1))
sapply(params, "[[", 1)
```

```
## a b c 
## 7 6 2
```

Explain what that does and why it works.

Hint:

```r
test <- list(5, 7, 3)
test[[2]]
```

```
## [1] 7
```

```r
# `[[`(test, 2)  # need it commented or R Markdown processing messes it up...

# `+`(3, 7)
```
