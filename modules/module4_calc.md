% R bootcamp, Module 4: Calculations
% August 2018, UC Berkeley
% Sean Wu (based on materials developed by Chris Paciorek)



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
##   0.064   0.000   0.064
```

```r
system.time(vals <- vals * (vals > 0))
```

```
##    user  system elapsed 
##   0.004   0.000   0.005
```

**Question**: What am I doing with `vals * (vals > 0)` ? What happens behind the scenes in R?

If you use a trick like this, having a comment in your code is a good idea.

Lots of functions in R are vectorized, such as some we've already used.


```r
tmp <- as.character(air$DepTime)
air$DepHour <- substring(tmp, 1, 2)
head(air$DepHour)
```

```
## [1] "12" "12" "12" NA   "12" "12"
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
##   2.640   0.016   2.655
```

The same holds for using `rbind()`, `cbind()`, or adding to a list, one element at a time.

This is both slow and unclear (in the sense that it appears the code has a bug, but it works):

```r
vals <- 0
n <- 50000
system.time({
for(i in 1:n)
      vals[i] <- i
})
```

```
##    user  system elapsed 
##   0.044   0.000   0.045
```

**Question**: Thoughts on why these are so slow? Think about what R might be doing behind the scenes

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
## [1] -0.213107
## 
## [[2]]
## [1] -1.263107
## 
## [[3]]
## [1] -0.5841702
```

```r
sapply(myList, min)
```

```
## [1] -0.2131070 -1.2631070 -0.5841702
```

Note that we don't generally want to use `apply()` on a data frame. 

You can use `lapply()` and `sapply()` on regular vectors, such as vectors of indices, which can come in handy, though this is a silly example:

```r
sapply(1:10, function(x) x^2)
```

```
##  [1]   1   4   9  16  25  36  49  64  81 100
```

Here's a cool trick to pull off a particular element of a list of lists:


```r
params <- list(a = list(mn = 7, sd = 3), b = list(mn = 6,sd = 1), 
  c = list(mn = 2, sd = 1))
sapply(params, "[[", 1)
```

```
## a b c 
## 7 6 2
```

**Challenge**: Think about why this works. 

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

# And more `apply()`s

There are a bunch of `apply()` variants, as well as parallelized versions of them:

* `tapply()`, `vapply()`, `mapply()`, `rapply()`, `eapply()`
* for parallelized versions see Module 10 or `?clusterApply`

# Tabulation 

- Sometimes we need to do some basic checking for the number of observations or types of observations in our dataset
- To do this quickly and easily, `table()` is our friend
- Let's look at our observations by year and airline


```r
unique(air$UniqueCarrier)
```

```
##  [1] "UA" "US" "NW" "OH" "OO" "TZ" "DL" "FL" "HA" "HP" "MQ" "AA" "AS" "CO"
## [15] "EV" "F9" "DH" "YV" "B6" "XE" "WN"
```

```r
tbl <- table(air$UniqueCarrier, air$Year)
tbl
```

```
##     
##       2005  2006  2007  2008
##   AA 12734 12770 12743 12008
##   AS  4525  4933  6088  5062
##   B6     0     0  1291  1923
##   CO  4557  4752  4919  4801
##   DH   470     0     0     0
##   DL  6338  5487  5212  4575
##   EV   179   140   423     0
##   F9  1196  2643  2893  1686
##   FL   642   815   960   951
##   HA   365   364   365   366
##   HP  4485     0     0     0
##   MQ  1922  1711  1766  1742
##   NW  3784  3928  3819  3827
##   OH    30     0     0     0
##   OO 38869 40407 41355 39027
##   TZ  1846   475     0     0
##   UA 43769 45766 45275 43736
##   US  3262  7454  7272  7112
##   WN     0     0  2675 12568
##   XE     0     0  1164  1203
##   YV     0   199   271     0
```

```r
round(prop.table(tbl, margin = 2), 3)
```

```
##     
##       2005  2006  2007  2008
##   AA 0.099 0.097 0.092 0.085
##   AS 0.035 0.037 0.044 0.036
##   B6 0.000 0.000 0.009 0.014
##   CO 0.035 0.036 0.036 0.034
##   DH 0.004 0.000 0.000 0.000
##   DL 0.049 0.042 0.038 0.033
##   EV 0.001 0.001 0.003 0.000
##   F9 0.009 0.020 0.021 0.012
##   FL 0.005 0.006 0.007 0.007
##   HA 0.003 0.003 0.003 0.003
##   HP 0.035 0.000 0.000 0.000
##   MQ 0.015 0.013 0.013 0.012
##   NW 0.029 0.030 0.028 0.027
##   OH 0.000 0.000 0.000 0.000
##   OO 0.301 0.306 0.299 0.278
##   TZ 0.014 0.004 0.000 0.000
##   UA 0.339 0.347 0.327 0.311
##   US 0.025 0.057 0.053 0.051
##   WN 0.000 0.000 0.019 0.089
##   XE 0.000 0.000 0.008 0.009
##   YV 0.000 0.002 0.002 0.000
```

```r
table(air$UniqueCarrier, air$Month, air$Cancelled)
```

```
## , ,  = 0
## 
##     
##          1     2     3     4     5     6     7     8     9    10    11
##   AA  3967  3542  3963  4089  4295  4200  4394  4370  4116  4260  4005
##   AS  1703  1579  1740  1683  1733  1706  1816  1775  1573  1604  1691
##   B6   154   143   154   150   352   349   336   344   297   305   296
##   CO  1438  1285  1482  1448  1580  1761  1875  1887  1536  1620  1442
##   DH     0     0     0     0    61    89    93    93    65    50    18
##   DL  1692  1518  1712  1674  1808  1933  2030  2032  1820  1845  1698
##   EV    88   105   113   114    84    25    23    27    24    30    28
##   F9   592   544   612   569   741   776   883   834   712   730   679
##   FL   138   113   152   218   299   403   480   485   325   310   220
##   HA   123   113   124   120   124   120   124   124   120   122   120
##   HP   398   351   393   383   396   364   355   361   352   361   348
##   MQ   563   516   582   587   612   592   597   604   567   587   520
##   NW  1139  1018  1125  1192  1358  1553  1651  1479  1205  1249  1123
##   OH    29     0     0     0     0     0     0     0     0     0     0
##   OO 12650 11942 13445 12757 13156 13138 13495 13790 12972 13449 12512
##   TZ   338   270   272   250   157   151   163   157   126   143   135
##   UA 14169 13110 14722 14474 15045 15050 15539 15719 14304 14884 14108
##   US  1986  1814  2013  1983  2150  2177  2333  2317  2055  2104  1926
##   WN   749   736   974  1000  1084  1061  1082  1185  1657  1705  1817
##   XE   143   123   145   140   142   268   354   336   184   182   174
##   YV    51    33    56    54    59    51    27    39    27    24    24
##     
##         12
##   AA  4021
##   AS  1723
##   B6   303
##   CO  1579
##   DH     0
##   DL  1684
##   EV    50
##   F9   694
##   FL   217
##   HA   123
##   HP   346
##   MQ   535
##   NW  1198
##   OH     0
##   OO 12765
##   TZ   142
##   UA 14445
##   US  1954
##   WN  1873
##   XE   162
##   YV    11
## 
## , ,  = 1
## 
##     
##          1     2     3     4     5     6     7     8     9    10    11
##   AA   137   157   115   113    54    70    53    60    56    49    62
##   AS    52    28    20    19    17    16    17    23    12    10    25
##   B6     1     2     1     0     0     7     6     6     0     0     1
##   CO     6    11    13     0     2     4     6     2    31     3     1
##   DH     0     0     0     0     0     1     0     0     0     0     0
##   DL    32    24    13    11    11    11    10     4     9     7     7
##   EV     4     3     7     2     5     1     1     0     1     0     0
##   F9     0     6     2     1     2     1     3     2     4     3     4
##   FL     0     0     0     0     0     2     5     1     0     0     0
##   HA     1     0     0     0     0     0     0     0     0     2     0
##   HP     4     9     6     2     4     7     6     3     4     6     6
##   MQ    36    33    17    10    16    13    27    21    18    20    32
##   NW     7     9     4     5     2     6     7    13     4     2     4
##   OH     1     0     0     0     0     0     0     0     0     0     0
##   OO   624   373   247   191   155   159   217   162   177   252   392
##   TZ     5     5     2     2     0     0     1     0     0     1     0
##   UA   357   269   231   249   205   255   326   253   179   178   153
##   US    37    29    36    17    10    28    29    18    16    16    26
##   WN    44    18    18    22     9    24    26    17    17    28    44
##   XE     5     0     0     0     0     3     1     1     0     0     2
##   YV     2     1     1     2     0     4     0     0     2     2     0
##     
##         12
##   AA   107
##   AS    43
##   B6     7
##   CO    17
##   DH     0
##   DL    27
##   EV     7
##   F9    24
##   FL     0
##   HA     0
##   HP    20
##   MQ    36
##   NW     5
##   OH     0
##   OO   638
##   TZ     1
##   UA   322
##   US    26
##   WN    53
##   XE     2
##   YV     0
```

```r
with(air[air$UniqueCarrier == 'UA', ], table(Month, Cancelled))
```

```
##      Cancelled
## Month     0     1
##    1  14169   357
##    2  13110   269
##    3  14722   231
##    4  14474   249
##    5  15045   205
##    6  15050   255
##    7  15539   326
##    8  15719   253
##    9  14304   179
##    10 14884   178
##    11 14108   153
##    12 14445   322
```

**Challenge**: Can you figure out what `with()` does just by example? 

# Stratified analyses I
Often we want to do individual analyses within subsets or clusters of our data.

As a first step, we might want to just split our dataset by a stratifying variable.  


```r
# restrict to a few destinations
DestSubset <- c('LAX','SEA','PHX','DEN','MSP','JFK','ATL','DFW','IAH', 'ORD') 
airSmall <- subset(air, Dest %in% DestSubset)
airSmall$DepDelay[airSmall$DepDelay > 60] <- 60
subsets <- split(airSmall, airSmall$Dest)
length(subsets)
```

```
## [1] 10
```

```r
dim(subsets[['LAX']])
```

```
## [1] 44265    30
```

```r
par(mfrow = c(1,2))
boxplot(DepDelay ~ Month, data = subsets[['IAH']], main = 'Houston')
abline(h = 0, col = 'grey')
boxplot(DepDelay ~ Month, data = subsets[['ORD']], main = 'Chicago')
abline(h = 0, col = 'grey')
```

![](figure/unnamed-chunk-17-1.png)

Note the use of the `%in%` operator can also be helpful.

# Stratified analyses II

Often we want to do individual analyses within subsets or clusters of our data. R has a variety of tools for this; for now we'll look at `aggregate()` and `by()`. These are wrappers of `tapply()`. 


```r
airSmallNum <- airSmall[ , c('DepDelay', 'ArrDelay', 'TaxiIn', 'TaxiOut')]
aggregate(airSmallNum, by = list(destination = airSmall$Dest), FUN = median, na.rm = TRUE)
```

```
##    destination DepDelay ArrDelay TaxiIn TaxiOut
## 1          ATL       -1       -2      8      16
## 2          DEN       -1       -1      7      15
## 3          DFW       -1       -2      9      15
## 4          IAH       -2       -1      7      15
## 5          JFK       -2        1      9      17
## 6          LAX       -2       -1      9      16
## 7          MSP       -4       -3      6      15
## 8          ORD        0        0      7      17
## 9          PHX       -1       -2      6      14
## 10         SEA       -1        0      5      16
```

```r
aggregate(DepDelay ~ Dest, data = airSmall, FUN = median)
```

```
##    Dest DepDelay
## 1   ATL       -1
## 2   DEN       -1
## 3   DFW       -1
## 4   IAH       -2
## 5   JFK       -2
## 6   LAX       -2
## 7   MSP       -4
## 8   ORD        0
## 9   PHX       -1
## 10  SEA       -1
```

```r
agg <- aggregate(DepDelay ~ Dest + Month, data = airSmall, FUN = median)
xtabs(DepDelay ~ ., data = agg)
```

```
##      Month
## Dest   1  2  3  4  5  6  7  8  9 10 11 12
##   ATL -1  0  0 -2 -2  1  0 -1 -2 -2 -1  1
##   DEN -2 -1 -1 -1 -1  1 -1 -1 -2 -2 -2  1
##   DFW -1 -1 -1 -1 -1  0 -1 -1 -2 -2 -2  0
##   IAH -3 -3 -2 -2 -2  0 -1 -1 -3 -2 -2  0
##   JFK -2 -2 -2 -2 -2  0 -1 -1 -2 -2 -2  0
##   LAX -2 -2 -2 -2 -2 -1 -2 -2 -3 -2 -2  0
##   MSP -4 -4 -4 -5 -5 -4 -5 -4 -4 -4 -3 -2
##   ORD  0  0  1  0 -1  2  0  0 -1 -1 -1  2
##   PHX -1 -1 -1 -2 -2  0 -1 -1 -2 -2 -2  0
##   SEA -2 -1 -1 -2 -1  1  0 -1 -3 -3 -2  1
```

Notice the 'long' vs. 'wide' formats. You'll see more about that sort of thing in Module 6.

# Discretization

You may need to discretize a continuous variable [or a discrete variable with many levels], e.g., by departure delay severity:

```r
air$delayStatus <- cut(air$DepDelay, breaks = c(-Inf, -5, 5, 15, 30, 60, 180, Inf))
levels(air$delayStatus) <- c('early','on-time','slight',
                        'short','medium','long','hair-pulling')
tbl <- table(air$UniqueCarrier, air$delayStatus)
round( prop.table(tbl, margin = 1), 2 )
```

```
##     
##      early on-time slight short medium long hair-pulling
##   AA  0.23    0.48   0.09  0.06   0.07 0.07         0.01
##   AS  0.36    0.28   0.10  0.09   0.08 0.08         0.01
##   B6  0.31    0.36   0.09  0.07   0.07 0.09         0.01
##   CO  0.29    0.42   0.09  0.07   0.06 0.05         0.01
##   DH  0.23    0.51   0.13  0.05   0.04 0.03         0.01
##   DL  0.25    0.50   0.11  0.07   0.04 0.03         0.00
##   EV  0.17    0.55   0.10  0.06   0.05 0.07         0.00
##   F9  0.40    0.33   0.10  0.07   0.06 0.04         0.00
##   FL  0.29    0.43   0.09  0.08   0.06 0.06         0.01
##   HA  0.28    0.49   0.08  0.05   0.04 0.04         0.02
##   HP  0.23    0.50   0.11  0.06   0.05 0.04         0.00
##   MQ  0.36    0.40   0.07  0.06   0.06 0.04         0.00
##   NW  0.45    0.36   0.06  0.05   0.04 0.04         0.01
##   OH  0.21    0.48   0.03  0.07   0.17 0.03         0.00
##   OO  0.25    0.40   0.11  0.08   0.08 0.08         0.00
##   TZ  0.38    0.39   0.07  0.05   0.05 0.04         0.01
##   UA  0.24    0.48   0.09  0.06   0.06 0.06         0.01
##   US  0.20    0.49   0.14  0.07   0.06 0.04         0.00
##   WN  0.11    0.59   0.09  0.06   0.07 0.08         0.01
##   XE  0.37    0.34   0.08  0.06   0.07 0.07         0.01
##   YV  0.25    0.48   0.08  0.04   0.07 0.07         0.00
```


# Stratified analyses III

`aggregate()` works fine when the output is univariate, but what about more complicated analyses than computing the median, such as fitting a set of regressions?


```r
airSmall$late <- airSmall$DepDelay >= 15
airSmall$Month <- as.factor(airSmall$Month)
airSmall$Month <- relevel(airSmall$Month, "5")
airSmall$Dest <- as.character(airSmall$Dest)

out <- by(airSmall, airSmall$Dest, 
    function(x) {
      if(sum(!is.na(x$late))) 
        # fit logistic regression
        glm(late ~ Month, family = binomial, data = x) 
      else NA
    }
)
length(out)
```

```
## [1] 10
```

```r
summary(out[['IAH']])
```

```
## 
## Call:
## glm(formula = late ~ Month, family = binomial, data = x)
## 
## Deviance Residuals: 
##     Min       1Q   Median       3Q      Max  
## -0.7365  -0.6121  -0.5750  -0.5252   2.2963  
## 
## Coefficients:
##              Estimate Std. Error z value Pr(>|z|)    
## (Intercept) -1.816777   0.096439 -18.839  < 2e-16 ***
## Month1       0.100730   0.136079   0.740   0.4592    
## Month2       0.204195   0.136910   1.491   0.1358    
## Month3       0.329094   0.129520   2.541   0.0111 *  
## Month4       0.153730   0.133867   1.148   0.2508    
## Month6       0.568904   0.124816   4.558 5.17e-06 ***
## Month7       0.237044   0.129051   1.837   0.0662 .  
## Month8       0.136570   0.131184   1.041   0.2978    
## Month9      -0.745516   0.166948  -4.466 7.99e-06 ***
## Month10      0.002425   0.136671   0.018   0.9858    
## Month11     -0.094463   0.140825  -0.671   0.5024    
## Month12      0.650724   0.124100   5.244 1.58e-07 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 9334.1  on 10533  degrees of freedom
## Residual deviance: 9199.0  on 10522  degrees of freedom
##   (48 observations deleted due to missingness)
## AIC: 9223
## 
## Number of Fisher Scoring iterations: 5
```

```r
summary(out[['ORD']])
```

```
## 
## Call:
## glm(formula = late ~ Month, family = binomial, data = x)
## 
## Deviance Residuals: 
##     Min       1Q   Median       3Q      Max  
## -0.8681  -0.7585  -0.7065  -0.6374   1.8404  
## 
## Coefficients:
##             Estimate Std. Error z value Pr(>|z|)    
## (Intercept) -1.36737    0.05601 -24.414  < 2e-16 ***
## Month1       0.34839    0.07732   4.506 6.61e-06 ***
## Month2       0.33658    0.07946   4.236 2.28e-05 ***
## Month3       0.26875    0.07777   3.456 0.000549 ***
## Month4       0.10664    0.07912   1.348 0.177701    
## Month6       0.48219    0.07538   6.397 1.59e-10 ***
## Month7       0.15244    0.07742   1.969 0.048939 *  
## Month8       0.12886    0.07773   1.658 0.097355 .  
## Month9      -0.11699    0.08168  -1.432 0.152086    
## Month10     -0.12304    0.08087  -1.521 0.128169    
## Month11      0.11599    0.07911   1.466 0.142599    
## Month12      0.58578    0.07505   7.805 5.95e-15 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 24639  on 22500  degrees of freedom
## Residual deviance: 24453  on 22489  degrees of freedom
##   (483 observations deleted due to missingness)
## AIC: 24477
## 
## Number of Fisher Scoring iterations: 4
```

**Question**: What's the business with the `if` statement? Why is this good practice?

# Sorting

`sort()` applied to a vector does what you expect.

Sorting a matrix or dataframe based on one or more columns is a somewhat manual process, but once you get the hang of it, it's not bad.


```r
ord <- order(air$DepDelay, air$ArrDelay, decreasing = TRUE)
ord[1:5]
```

```
## [1] 115794 396990 337213 168750 415550
```

```r
air_ordered <- air[ord, ]
head(air_ordered)
```

```
##        Year Month DayofMonth DayOfWeek DepTime CRSDepTime ArrTime
## 115794 2005    11          2         3    1009        630    1526
## 396990 2007    12         31         1     745       1225    1318
## 337213 2007     7         16         1     858       1410    1805
## 168750 2006     4         11         2    1001       1543    1505
## 415550 2008     2          9         6    1003       1605    1302
## 290635 2007     3         22         4     729       1545    1257
##        CRSArrTime UniqueCarrier FlightNum TailNum ActualElapsedTime
## 115794       1207            NW       368  N552NW               197
## 396990       1809            NW       360  N525US               213
## 337213       2245            AA       194  N5EVAA               367
## 168750       2112            NW       354  N541US               184
## 415550       1946            UA        77  N667UA               299
## 290635       2115            NW       354  N504US               208
##        CRSElapsedTime AirTime ArrDelay DepDelay Origin Dest Distance
## 115794            217     175     1639     1659    SFO  MSP     1589
## 396990            224     191     1149     1160    SFO  MSP     1589
## 337213            335     323     1160     1128    SFO  BOS     2704
## 168750            209     165     1073     1098    SFO  MSP     1589
## 415550            341     272     1036     1078    SFO  HNL     2398
## 290635            210     188      942      944    SFO  MSP     1589
##        TaxiIn TaxiOut Cancelled CancellationCode Diverted CarrierDelay
## 115794      3      19         0                         0         1639
## 396990      9      13         0                         0         1106
## 337213     25      19         0                         0          993
## 168750      5      14         0                         0         1048
## 415550      2      25         0                         0          710
## 290635      4      16         0                         0          884
##        WeatherDelay NASDelay SecurityDelay LateAircraftDelay DepHour
## 115794            0        0             0                 0      10
## 396990            0        0             0                43      74
## 337213            0       32             0               135      85
## 168750            6        0             0                19      10
## 415550            0        0             0               326      10
## 290635            0        0             0                58      72
##         delayStatus
## 115794 hair-pulling
## 396990 hair-pulling
## 337213 hair-pulling
## 168750 hair-pulling
## 415550 hair-pulling
## 290635 hair-pulling
```

You could of course write your own *sort* function that uses `order()`. More in Module 6.

# Merging Data

We often need to combine data across multiple data frames, merging on common fields (i.e., *keys*). In database terminology, this is a *join* operation.

Here's an example where we want to add a column indicating the number of flights to the given destination.


```r
numFlights <- as.data.frame(table(air$Dest))
head(numFlights)
```

```
##   Var1  Freq
## 1  ABQ  2467
## 2  ACV 10402
## 3  ANC   613
## 4  ASE   166
## 5  ATL 13374
## 6  AUS  1705
```

```r
air2 <- merge(air, numFlights, by.x = 'Dest', by.y = 'Var1',
     all.x = TRUE, all.y = FALSE)
dim(air)
```

```
## [1] 539895     31
```

```r
dim(air2)
```

```
## [1] 539895     32
```

```r
head(air2)
```

```
##   Dest Year Month DayofMonth DayOfWeek DepTime CRSDepTime ArrTime
## 1  ABQ 2007     3          3         6    1208       1218    1525
## 2  ABQ 2007     4         24         2    2030       2025    2333
## 3  ABQ 2007     9         13         4    2036       2042    2348
## 4  ABQ 2007    12         18         2    2315       2100     227
## 5  ABQ 2008     1         29         2    1327       1230    1653
## 6  ABQ 2007     9         12         3    1346       1218    1704
##   CRSArrTime UniqueCarrier FlightNum TailNum ActualElapsedTime
## 1       1539            OO      6416  N969SW               137
## 2       2341            OO      5588  N728SK               123
## 3       2356            OO      6422  N738SK               132
## 4         16            OO      6422  N978SW               132
## 5       1548            OO      6416  N405SW               146
## 6       1535            OO      6416  N923SW               138
##   CRSElapsedTime AirTime ArrDelay DepDelay Origin Distance TaxiIn TaxiOut
## 1            141     120      -14      -10    SFO      896      5      12
## 2            136     107       -8        5    SFO      896      4      12
## 3            134     111       -8       -6    SFO      896      5      16
## 4            136     110      131      135    SFO      896      6      16
## 5            138     102       65       57    SFO      896      6      38
## 6            137     118       89       88    SFO      896      6      14
##   Cancelled CancellationCode Diverted CarrierDelay WeatherDelay NASDelay
## 1         0                         0            0            0        0
## 2         0                         0            0            0        0
## 3         0                         0            0            0        0
## 4         0                         0          131            0        0
## 5         0                         0            0            0        0
## 6         0                         0            0            0        0
##   SecurityDelay LateAircraftDelay DepHour delayStatus Freq
## 1             0                 0      12       early 2467
## 2             0                 0      20     on-time 2467
## 3             0                 0      20       early 2467
## 4             0                 0      23        long 2467
## 5             0                65      13      medium 2467
## 6             0                89      13        long 2467
```

What's the deal with the `all.x` and `all.y` ?  We can tell R whether we want to keep all of the `x` observations, all the `y` observations, or neither, or both, when there may be rows in either of the datasets that don't match the other dataset.

# Breakout

### Basics

1) Create a vector that concatenates the Dest and Origin to create a 'Route' variable in a vectorized way using the string processing functions.

2) Use `table()` to figure out the number of flights for each year.

### Using the ideas

3) Compute the number of NAs in each column of the *air* dataset using `sapply()` and making use of the `is.na()` function.

4) Merge the info from data/carriers.csv with the airline dataset so that we have the full names of the airlines as a new column in the data frame.

5) Discretize distance into some bins and create a Dist_binned variable. Count the number of flights in each bin.

6) Create a boxplot of delay by binned distance.

7) Sort the dataset and find the shortest flight. Now consider the use of `which.min()` and why using that should be much quicker with large datasets.
 
8)  Suppose we have two categorical variables and we conduct a hypothesis test of independence. The chi-square statistic is: 

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

9) For each combination of UniqueCarrier, Month, and DayOfWeek, find the 95th percentile of delay. 

10) This requires a bit of setup. Run the following code:

```r
h <- air$DepTime %/% 100
m <- air$DepTime %% 100
h <- as.character(h)
h[!is.na(h) & nchar(h) == 1] <- paste('0', h[!is.na(h) & nchar(h) == 1], sep = '')
air$DepTimeChar <- paste(h, m, sep = '-')
```

Now suppose the `DepTimeChar` field is the format the departure time field came in.

Create hour and minute fields using `strsplit()` and `sapply()`. What format is the result of `strsplit()`. Why do you need `sapply()`? 

