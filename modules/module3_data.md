% R bootcamp, Module 3: Working with objects and data
% August 2018, UC Berkeley
% Chris Paciorek



# Matrices and Arrays

Let's review matrices

- Matrices are two-dimensional collections of values of the same type
- We can have numeric, integer, character, or logical matrices, character matrices.
- You can't mix types within a matrix

```r
mat <- matrix(rnorm(12), nrow = 3, ncol = 4)
mat
```

```
##            [,1]       [,2]       [,3]       [,4]
## [1,] -1.7228439 -0.7207214 -0.2877622  0.5133103
## [2,]  0.2321758  0.8871828  1.2808443 -0.5546136
## [3,]  1.5920781  2.5924839 -1.6912609  0.0142042
```

```r
# vectorized calcs work with matrices too
mat*4
```

```
##           [,1]      [,2]      [,3]       [,4]
## [1,] -6.891376 -2.882886 -1.151049  2.0532412
## [2,]  0.928703  3.548731  5.123377 -2.2184545
## [3,]  6.368312 10.369936 -6.765044  0.0568168
```

```r
mat <- cbind(mat, 1:3)
mat
```

```
##            [,1]       [,2]       [,3]       [,4] [,5]
## [1,] -1.7228439 -0.7207214 -0.2877622  0.5133103    1
## [2,]  0.2321758  0.8871828  1.2808443 -0.5546136    2
## [3,]  1.5920781  2.5924839 -1.6912609  0.0142042    3
```

Arrays are like matrices but can have more or fewer than two dimensions.

```r
arr <- array(rnorm(12), c(2, 3, 4))
arr
```

```
## , , 1
## 
##            [,1]       [,2]        [,3]
## [1,] -0.9071718 -0.9638432 -0.04292199
## [2,]  0.3912341  1.4366121 -0.49244939
## 
## , , 2
## 
##            [,1]        [,2]        [,3]
## [1,] -0.8790961 -0.09792996 -0.03015456
## [2,] -0.1004457  1.13332547 -0.54939557
## 
## , , 3
## 
##            [,1]       [,2]        [,3]
## [1,] -0.9071718 -0.9638432 -0.04292199
## [2,]  0.3912341  1.4366121 -0.49244939
## 
## , , 4
## 
##            [,1]        [,2]        [,3]
## [1,] -0.8790961 -0.09792996 -0.03015456
## [2,] -0.1004457  1.13332547 -0.54939557
```

# Attributes

Objects have *attributes*.


```r
attributes(mat)
```

```
## $dim
## [1] 3 5
```

```r
rownames(mat) <- c('first', 'middle', 'last')
mat
```

```
##              [,1]       [,2]       [,3]       [,4] [,5]
## first  -1.7228439 -0.7207214 -0.2877622  0.5133103    1
## middle  0.2321758  0.8871828  1.2808443 -0.5546136    2
## last    1.5920781  2.5924839 -1.6912609  0.0142042    3
```

```r
attributes(mat)
```

```
## $dim
## [1] 3 5
## 
## $dimnames
## $dimnames[[1]]
## [1] "first"  "middle" "last"  
## 
## $dimnames[[2]]
## NULL
```

```r
names(attributes(air))
```

```
## [1] "names"     "class"     "row.names"
```

```r
attributes(air)$names
```

```
##  [1] "Year"              "Month"             "DayOfMonth"       
##  [4] "DayOfWeek"         "DepTime"           "CRSDepTime"       
##  [7] "ArrTime"           "CRSArrTime"        "UniqueCarrier"    
## [10] "FlightNum"         "TailNum"           "ActualElapsedTime"
## [13] "CRSElapsedTime"    "AirTime"           "ArrDelay"         
## [16] "DepDelay"          "Origin"            "Dest"             
## [19] "Distance"          "TaxiIn"            "TaxiOut"          
## [22] "Cancelled"         "CancellationCode"  "Diverted"         
## [25] "CarrierDelay"      "WeatherDelay"      "NASDelay"         
## [28] "SecurityDelay"     "LateAircraftDelay"
```

```r
attributes(air)$row.names[1:10]
```

```
##  [1]  1  2  3  4  5  6  7  8  9 10
```

Now let's do a bit of manipulation and see if you can infer how R represents matrices internally.


```r
mat[4]
```

```
## [1] -0.7207214
```

```r
attributes(mat) <- NULL
mat
```

```
##  [1] -1.7228439  0.2321758  1.5920781 -0.7207214  0.8871828  2.5924839
##  [7] -0.2877622  1.2808443 -1.6912609  0.5133103 -0.5546136  0.0142042
## [13]  1.0000000  2.0000000  3.0000000
```

```r
is.matrix(mat)
```

```
## [1] FALSE
```

**Question**: What can you infer about what a matrix is in R?

**Question**: What kind of object are the attributes themselves? How do I check?

# Matrices are stored column-major

This is like Fortran, MATLAB and Julia but not like C or Python(numpy). 


```r
mat <- matrix(1:12, 3, 4)
mat
```

```
##      [,1] [,2] [,3] [,4]
## [1,]    1    4    7   10
## [2,]    2    5    8   11
## [3,]    3    6    9   12
```

```r
c(mat)
```

```
##  [1]  1  2  3  4  5  6  7  8  9 10 11 12
```
You can go smoothly back and forth between a matrix (or an array) and a vector:

```r
identical(mat, matrix(c(mat), 3, 4))
```

```
## [1] TRUE
```

```r
identical(mat, matrix(c(mat), 3, 4, byrow = TRUE))
```

```
## [1] FALSE
```

This is a common cause of bugs!


# Missing values and other special values

Since it was designed by statisticians, R handles missing values very well relative to other languages.

* `NA` is a missing value

```r
vec <- rnorm(12)
vec[c(3, 5)] <- NA
vec
```

```
##  [1]  2.3025243 -0.2552535         NA -0.6053567         NA  0.9591721
##  [7]  1.2998584  0.7039802  0.7709010 -0.1919022 -0.3071393  0.9991563
```

```r
length(vec)
```

```
## [1] 12
```

```r
sum(vec)
```

```
## [1] NA
```

```r
sum(vec, na.rm = TRUE)
```

```
## [1] 5.67594
```

```r
hist(vec)
```

![](figure/unnamed-chunk-5-1.png)

```r
is.na(vec)
```

```
##  [1] FALSE FALSE  TRUE FALSE  TRUE FALSE FALSE FALSE FALSE FALSE FALSE
## [12] FALSE
```
Be careful because many R functions won't warn you that they are ignoring the missing values.

* To infinity and beyond

```r
big <- 1e500 
big
```

```
## [1] Inf
```

```r
big + 7
```

```
## [1] Inf
```

* `NaN` stands for Not a Number

```r
sqrt(-5)
```

```
## Warning in sqrt(-5): NaNs produced
```

```
## [1] NaN
```

```r
big - big
```

```
## [1] NaN
```

```r
1/0
```

```
## [1] Inf
```

* `NULL`

```r
vec <- c(vec, NULL) 
vec
```

```
##  [1]  2.3025243 -0.2552535         NA -0.6053567         NA  0.9591721
##  [7]  1.2998584  0.7039802  0.7709010 -0.1919022 -0.3071393  0.9991563
```

```r
length(vec)
```

```
## [1] 12
```

```r
a <- NULL
a + 7
```

```
## numeric(0)
```

```r
a[3, 4]
```

```
## NULL
```

```r
is.null(a)
```

```
## [1] TRUE
```

```r
myList <- list(a = 7, b = 5)
myList$a <- NULL  # works for data frames too
myList
```

```
## $b
## [1] 5
```

`NA` can hold a place but `NULL` cannot.
`NULL` is useful for having a function argument default to 'nothing'. See `help(crossprod)`, which can compute either $X^{\top}X$ or $X^{\top}Y$.  


# Logical vectors

```r
answers <- c(TRUE, TRUE, FALSE, FALSE)
update <- c(TRUE, FALSE, TRUE, FALSE)

answers & update
```

```
## [1]  TRUE FALSE FALSE FALSE
```

```r
answers | update
```

```
## [1]  TRUE  TRUE  TRUE FALSE
```

```r
# note the vectorized boolean arithmetic

# what am I doing here?
sum(answers)
```

```
## [1] 2
```

```r
mean(answers)
```

```
## [1] 0.5
```

```r
answers + update
```

```
## [1] 2 1 1 0
```

**Question**: What do you think R is doing to do arithmetic on logical vectors?

Tricks with logicals...


```r
identical(answers & update, as.logical(answers * update))
```

```
## [1] TRUE
```

```r
identical(answers | update, as.logical(answers + update))
```

```
## [1] TRUE
```

# Data frames

A review from Module 1...

- Data frames are combinations of vectors of the same length, but can be of different types
- Data frames are what is used for standard rectangular (record by field) datasets, similar to a spreadsheet
- Data frames are a functionality that both sets R aside from some languages (e.g., Matlab) and provides functionality similar to some statistical packages (e.g., Stata, SAS)


```r
air <- read.csv('../data/airline.csv')
class(air)
```

```
## [1] "data.frame"
```

```r
head(air)
```

```
##   Year Month DayOfMonth DayOfWeek DepTime CRSDepTime ArrTime CRSArrTime
## 1 2005     1          1         6    1211       1216    1451       1502
## 2 2005     1          2         7    1209       1216    1447       1502
## 3 2005     1          3         1    1213       1216    1454       1502
## 4 2005     1          4         2      NA       1216      NA       1502
## 5 2005     1          5         3    1211       1216    1504       1502
## 6 2005     1          6         4    1214       1215    1506       1505
##   UniqueCarrier FlightNum TailNum ActualElapsedTime CRSElapsedTime AirTime
## 1            UA       548  N341UA               100            106      81
## 2            UA       548  N398UA                98            106      79
## 3            UA       548  N303UA               101            106      83
## 4            UA       548  000000                NA            106      NA
## 5            UA       548  N917UA               113            106      85
## 6            UA       548  N924UA               112            110      95
##   ArrDelay DepDelay Origin Dest Distance TaxiIn TaxiOut Cancelled
## 1      -11       -5    SFO  SLC      599      2      17         0
## 2      -15       -7    SFO  SLC      599      2      17         0
## 3       -8       -3    SFO  SLC      599      3      15         0
## 4       NA       NA    SFO  SLC      599      0       0         1
## 5        2       -5    SFO  SLC      599      6      22         0
## 6        1       -1    SFO  SLC      599      6      11         0
##   CancellationCode Diverted CarrierDelay WeatherDelay NASDelay
## 1                         0            0            0        0
## 2                         0            0            0        0
## 3                         0            0            0        0
## 4                A        0            0            0        0
## 5                         0            0            0        0
## 6                         0            0            0        0
##   SecurityDelay LateAircraftDelay
## 1             0                 0
## 2             0                 0
## 3             0                 0
## 4             0                 0
## 5             0                 0
## 6             0                 0
```

```r
str(air)
```

```
## 'data.frame':	539895 obs. of  29 variables:
##  $ Year             : int  2005 2005 2005 2005 2005 2005 2005 2005 2005 2005 ...
##  $ Month            : int  1 1 1 1 1 1 1 1 1 1 ...
##  $ DayOfMonth       : int  1 2 3 4 5 6 7 8 9 10 ...
##  $ DayOfWeek        : int  6 7 1 2 3 4 5 6 7 1 ...
##  $ DepTime          : int  1211 1209 1213 NA 1211 1214 1330 1213 1206 1338 ...
##  $ CRSDepTime       : int  1216 1216 1216 1216 1216 1215 1215 1215 1215 1215 ...
##  $ ArrTime          : int  1451 1447 1454 NA 1504 1506 1620 1448 1443 1610 ...
##  $ CRSArrTime       : int  1502 1502 1502 1502 1502 1505 1505 1505 1505 1505 ...
##  $ UniqueCarrier    : Factor w/ 21 levels "AA","AS","B6",..: 17 17 17 17 17 17 17 17 17 17 ...
##  $ FlightNum        : int  548 548 548 548 548 548 548 548 548 548 ...
##  $ TailNum          : Factor w/ 3983 levels "","0","000000",..: 1033 1387 756 3 3777 3812 3840 1065 1387 3834 ...
##  $ ActualElapsedTime: int  100 98 101 NA 113 112 110 95 97 92 ...
##  $ CRSElapsedTime   : int  106 106 106 106 106 110 110 110 110 110 ...
##  $ AirTime          : int  81 79 83 NA 85 95 85 72 77 78 ...
##  $ ArrDelay         : int  -11 -15 -8 NA 2 1 75 -17 -22 65 ...
##  $ DepDelay         : int  -5 -7 -3 NA -5 -1 75 -2 -9 83 ...
##  $ Origin           : Factor w/ 1 level "SFO": 1 1 1 1 1 1 1 1 1 1 ...
##  $ Dest             : Factor w/ 82 levels "ABQ","ACV","ANC",..: 76 76 76 76 76 76 76 76 76 76 ...
##  $ Distance         : int  599 599 599 599 599 599 599 599 599 599 ...
##  $ TaxiIn           : int  2 2 3 0 6 6 6 3 3 3 ...
##  $ TaxiOut          : int  17 17 15 0 22 11 19 20 17 11 ...
##  $ Cancelled        : int  0 0 0 1 0 0 0 0 0 0 ...
##  $ CancellationCode : Factor w/ 5 levels "","A","B","C",..: 1 1 1 2 1 1 1 1 1 1 ...
##  $ Diverted         : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ CarrierDelay     : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ WeatherDelay     : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ NASDelay         : int  0 0 0 0 0 0 3 0 0 0 ...
##  $ SecurityDelay    : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ LateAircraftDelay: int  0 0 0 0 0 0 72 0 0 65 ...
```

# Data frames are (special) lists!


```r
is.list(air)
```

```
## [1] TRUE
```

```r
length(air)
```

```
## [1] 29
```

```r
air[[3]][1:5]
```

```
## [1] 1 2 3 4 5
```

```r
lapply(air, class) 
```

```
## $Year
## [1] "integer"
## 
## $Month
## [1] "integer"
## 
## $DayOfMonth
## [1] "integer"
## 
## $DayOfWeek
## [1] "integer"
## 
## $DepTime
## [1] "integer"
## 
## $CRSDepTime
## [1] "integer"
## 
## $ArrTime
## [1] "integer"
## 
## $CRSArrTime
## [1] "integer"
## 
## $UniqueCarrier
## [1] "factor"
## 
## $FlightNum
## [1] "integer"
## 
## $TailNum
## [1] "factor"
## 
## $ActualElapsedTime
## [1] "integer"
## 
## $CRSElapsedTime
## [1] "integer"
## 
## $AirTime
## [1] "integer"
## 
## $ArrDelay
## [1] "integer"
## 
## $DepDelay
## [1] "integer"
## 
## $Origin
## [1] "factor"
## 
## $Dest
## [1] "factor"
## 
## $Distance
## [1] "integer"
## 
## $TaxiIn
## [1] "integer"
## 
## $TaxiOut
## [1] "integer"
## 
## $Cancelled
## [1] "integer"
## 
## $CancellationCode
## [1] "factor"
## 
## $Diverted
## [1] "integer"
## 
## $CarrierDelay
## [1] "integer"
## 
## $WeatherDelay
## [1] "integer"
## 
## $NASDelay
## [1] "integer"
## 
## $SecurityDelay
## [1] "integer"
## 
## $LateAircraftDelay
## [1] "integer"
```

`lapply()` is a function used on lists; it works here to apply the `class()` function to each element of the list, which in this case is each field/column.

# But lists are also vectors!


```r
length(air)
```

```
## [1] 29
```

```r
someFields <- air[c(3,5)]
head(someFields)
```

```
##   DayOfMonth DepTime
## 1          1    1211
## 2          2    1209
## 3          3    1213
## 4          4      NA
## 5          5    1211
## 6          6    1214
```

```r
identical(air[c(3,5)], air[ , c(3,5)])
```

```
## [1] TRUE
```

In general the placement of commas in R is crucial, but here, two different operations give the same result because of the underlying structure of data frames.

# Factors
- A factor is a special data type in R used for categorical data. In some cases it works like magic and in others it is incredibly frustrating. 


```r
class(air$Dest)
```

```
## [1] "factor"
```

```r
head(air$Dest) # What order are the factors in?
```

```
## [1] SLC SLC SLC SLC SLC SLC
## 82 Levels: ABQ ACV ANC ASE ATL AUS BFL BIL BOI BOS BUR BWI BZN CEC ... TWF
```

```r
levels(air[["Dest"]])  # note alternate way to get the variable
```

```
##  [1] "ABQ" "ACV" "ANC" "ASE" "ATL" "AUS" "BFL" "BIL" "BOI" "BOS" "BUR"
## [12] "BWI" "BZN" "CEC" "CIC" "CLE" "CLT" "COS" "CVG" "DEN" "DFW" "DRO"
## [23] "DTW" "EGE" "ELP" "EUG" "EWR" "FAT" "FCA" "GJT" "HNL" "IAD" "IAH"
## [34] "IDA" "IND" "JFK" "KOA" "LAS" "LAX" "LGB" "LIH" "LMT" "MCO" "MDW"
## [45] "MEM" "MFR" "MIA" "MKE" "MOD" "MRY" "MSO" "MSP" "MSY" "OAK" "OGG"
## [56] "ONT" "ORD" "OTH" "PDX" "PHL" "PHX" "PIH" "PIT" "PMD" "PSC" "PSP"
## [67] "RDD" "RDM" "RNO" "SAN" "SAT" "SBA" "SBP" "SEA" "SJC" "SLC" "SMF"
## [78] "SMX" "SNA" "STL" "TUS" "TWF"
```

```r
summary(air$Dest)
```

```
##   ABQ   ACV   ANC   ASE   ATL   AUS   BFL   BIL   BOI   BOS   BUR   BWI 
##  2467 10402   613   166 13374  1705  3824    13  8338 10191  9052  1890 
##   BZN   CEC   CIC   CLE   CLT   COS   CVG   DEN   DFW   DRO   DTW   EGE 
##    43  2192  5102  1930  5313  1859  3957 20545 15694     2  4554    16 
##   ELP   EUG   EWR   FAT   FCA   GJT   HNL   IAD   IAH   IDA   IND   JFK 
##     1  8928 11741 10587    12     3 11644 11778 10582     2   752 24047 
##   KOA   LAS   LAX   LGB   LIH   LMT   MCO   MDW   MEM   MFR   MIA   MKE 
##  2741 24210 44265   209  2460   348  1593  2490   993 10829  4381   115 
##   MOD   MRY   MSO   MSP   MSY   OAK   OGG   ONT   ORD   OTH   PDX   PHL 
##  6334 10011    12  7935   251     1  4610  4409 22984   354 12707  9390 
##   PHX   PIH   PIT   PMD   PSC   PSP   RDD   RDM   RNO   SAN   SAT   SBA 
## 16313     2  1850  1119     1  5792  7382  4111  9114 18114  1367 12651 
##   SBP   SEA   SJC   SLC   SMF   SMX   SNA   STL   TUS   TWF 
##  6945 22283     2 15925 10806    13 15180  2797  1164    13
```

- What if we don't like the order these are in? Factor order is important for all kinds of things like plotting, analysis of variance, regression output, and more

# Ordering the Factor
- Ordered factors simply have an additional attribute explaining the order of the levels of a factor
- This is a useful shortcut when we want to preserve some of the meaning provided by the order
- Think ordinal data


```r
# first let's do some pre-processing to get set up for the example
air$DepDelayCens <- air$DepDelay
air$DepDelayCens[air$DepDelayCens > 60] <- 60
air$DepDelayCens[air$DepDelayCens < 0] <- 0
air$DayOfWeek <- as.factor(air$DayOfWeek)
air$DayOfWeekSun <- ordered(air$DayOfWeek, 
     levels = levels(air$DayOfWeek)[c(7,1,2,3,4,5,6)])

# alternative coding
# air <- within(air, 
#     DayOfWeekSun <- ordered(DayOfWeek, 
#        levels = levels(DayOfWeek)[c(7,1,2,3,4,5,6)])
#)

head(air$DayOfWeekSun)
```

```
## [1] 6 7 1 2 3 4
## Levels: 7 < 1 < 2 < 3 < 4 < 5 < 6
```

```r
levels(air$DayOfWeekSun)
```

```
## [1] "7" "1" "2" "3" "4" "5" "6"
```

```r
levels(air$DayOfWeekSun) <- c("Sun","Mon","Tue","Wed","Thu","Fri","Sat")
boxplot(DepDelayCens ~ DayOfWeekSun, data = air)
```

![](figure/orderedfac-1.png)

**Challenge**: Try to decipher what I just did with that complicated single line of code starting with `within`.

# Reclassifying Factors
- Turning factors into other data types can be tricky. All factors have an underlying numeric structure.


```r
students <- factor(c('basic','proficient','advanced','basic', 
      'advanced', 'minimal'))
levels(students)
```

```
## [1] "advanced"   "basic"      "minimal"    "proficient"
```

```r
unclass(students)
```

```
## [1] 2 4 1 2 1 3
## attr(,"levels")
## [1] "advanced"   "basic"      "minimal"    "proficient"
```

- Hmmm, what happened?
- Be careful! The best way to convert a factor is to convert it to a character first.


```r
students <- factor(c('basic','proficient','advanced','basic', 
      'advanced', 'minimal'))
score = c(minimal = 3, basic = 1, advanced = 13, proficient = 7) # a named vector
score["advanced"]  # look up by name
```

```
## advanced 
##       13
```

```r
students[3]
```

```
## [1] advanced
## Levels: advanced basic minimal proficient
```

```r
score[students[3]]
```

```
## minimal 
##       3
```

```r
score[as.character(students[3])]
```

```
## advanced 
##       13
```

What went wrong and how did we fix it?  Notice how easily this could be a big bug in your code.

# Subsetting

There are many ways to select subsets in R. The syntax below is useful for vectors, matrices, data frames, arrays and lists.


```r
vec <- rnorm(20)
mat <- matrix(vec, 4, 5)
rownames(mat) <- letters[1:4]
mat
```

```
##         [,1]       [,2]        [,3]        [,4]       [,5]
## a  0.9388063 -0.1578055 -0.63870678 -1.48048466 -0.3953748
## b -1.2129332 -0.6416316 -1.49569066  0.02969633 -0.3237404
## c  1.3533546 -0.5549230 -0.07178203  1.55374227  1.7648537
## d -0.5986032  0.1630123  0.21077843  1.58435976 -0.4144714
```
1) by direct indexing


```r
vec[c(3, 5, 12:14)]
```

```
## [1]  1.35335459 -0.15780554  0.21077843 -1.48048466  0.02969633
```

```r
vec[-c(3,5)]
```

```
##  [1]  0.93880627 -1.21293324 -0.59860321 -0.64163157 -0.55492299
##  [6]  0.16301234 -0.63870678 -1.49569066 -0.07178203  0.21077843
## [11] -1.48048466  0.02969633  1.55374227  1.58435976 -0.39537483
## [16] -0.32374035  1.76485365 -0.41447143
```

```r
mat[c(2,4), 5]
```

```
##          b          d 
## -0.3237404 -0.4144714
```

```r
rowInd <- c(1, 3, 4)
colInd <- c(2, 2, 1)
elemInd <- cbind(rowInd, colInd)
elemInd
```

```
##      rowInd colInd
## [1,]      1      2
## [2,]      3      2
## [3,]      4      1
```

```r
mat[elemInd]
```

```
## [1] -0.1578055 -0.5549230 -0.5986032
```

Note the last usage where we give it a 2-column matrix of indices

2) by a vector of logicals


```r
cond <- vec > 0
vec[cond]
```

```
## [1] 0.93880627 1.35335459 0.16301234 0.21077843 0.02969633 1.55374227
## [7] 1.58435976 1.76485365
```

```r
air[air$DepDelay > 60*12 & !is.na(air$DepDelay), ]
```

```
##        Year Month DayOfMonth DayOfWeek DepTime CRSDepTime ArrTime
## 71351  2005     7         23         6     633       1545    1212
## 80042  2005     8         20         6    2237       1030     122
## 115794 2005    11          2         3    1009        630    1526
## 168750 2006     4         11         2    1001       1543    1505
## 258029 2006    12         28         4    2232        845     216
## 258288 2006    12         14         4    1118       2156    1908
## 290635 2007     3         22         4     729       1545    1257
## 333067 2007     7         16         1    1200       2021    1522
## 337213 2007     7         16         1     858       1410    1805
## 372929 2007    10         28         7    2155        835      37
## 396990 2007    12         31         1     745       1225    1318
## 415550 2008     2          9         6    1003       1605    1302
## 449626 2008     5         10         6     735       1820    1005
## 454313 2008     5         15         4    2243        850      50
##        CRSArrTime UniqueCarrier FlightNum TailNum ActualElapsedTime
## 71351        2113            NW       354  N586NW               219
## 80042        1301            TZ      4603  N561TZ               345
## 115794       1207            NW       368  N552NW               197
## 168750       2112            NW       354  N541US               184
## 258029       1205            HA        11  N587HA               344
## 258288        527            NW       346  N539US               290
## 290635       2115            NW       354  N504US               208
## 333067       2359            UA       595  N535UA               262
## 337213       2245            AA       194  N5EVAA               367
## 372929       1045            HA        11  N585HA               162
## 396990       1809            NW       360  N525US               213
## 415550       1946            UA        77  N667UA               299
## 449626       2041            UA        39  N669UA               330
## 454313       1055            HA        11  N596HA               307
##        CRSElapsedTime AirTime ArrDelay DepDelay Origin Dest Distance
## 71351             208     184      899      888    SFO  MSP     1589
## 80042             331     323      741      727    SFO  LIH     2447
## 115794            217     175     1639     1659    SFO  MSP     1589
## 168750            209     165     1073     1098    SFO  MSP     1589
## 258029            320     189      851      827    SFO  HNL     2398
## 258288            271     228      821      802    SFO  DTW     2079
## 290635            210     188      942      944    SFO  MSP     1589
## 333067            278     235      923      939    SFO  ANC     2018
## 337213            335     323     1160     1128    SFO  BOS     2704
## 372929            130     123      832      800    SFO  HNL     2398
## 396990            224     191     1149     1160    SFO  MSP     1589
## 415550            341     272     1036     1078    SFO  HNL     2398
## 449626            321     299      804      795    SFO  OGG     2338
## 454313            305     286      835      833    SFO  HNL     2398
##        TaxiIn TaxiOut Cancelled CancellationCode Diverted CarrierDelay
## 71351      13      22         0                         0          888
## 80042       5      17         0                         0          652
## 115794      3      19         0                         0         1639
## 168750      5      14         0                         0         1048
## 258029      8      27         0                         0          827
## 258288     33      29         0                         0          802
## 290635      4      16         0                         0          884
## 333067      4      23         0                         0          844
## 337213     25      19         0                         0          993
## 372929      6      33         0                         0          800
## 396990      9      13         0                         0         1106
## 415550      2      25         0                         0          710
## 449626      6      25         0                         0          795
## 454313      7      14         0                         0          835
##        WeatherDelay NASDelay SecurityDelay LateAircraftDelay DepDelayCens
## 71351             0       11             0                 0           60
## 80042             0       14             0                75           60
## 115794            0        0             0                 0           60
## 168750            6        0             0                19           60
## 258029            0        0             0                24           60
## 258288            0       19             0                 0           60
## 290635            0        0             0                58           60
## 333067            0        0             0                79           60
## 337213            0       32             0               135           60
## 372929            0        0             0                32           60
## 396990            0        0             0                43           60
## 415550            0        0             0               326           60
## 449626            0        9             0                 0           60
## 454313            0        0             0                 0           60
##        DayOfWeekSun
## 71351           Sat
## 80042           Sat
## 115794          Wed
## 168750          Tue
## 258029          Thu
## 258288          Thu
## 290635          Thu
## 333067          Mon
## 337213          Mon
## 372929          Sun
## 396990          Mon
## 415550          Sat
## 449626          Sat
## 454313          Thu
```

What happened in the last subsetting operation?

3) by a vector of names

```r
mat[c('a', 'd', 'a'), ]
```

```
##         [,1]       [,2]       [,3]      [,4]       [,5]
## a  0.9388063 -0.1578055 -0.6387068 -1.480485 -0.3953748
## d -0.5986032  0.1630123  0.2107784  1.584360 -0.4144714
## a  0.9388063 -0.1578055 -0.6387068 -1.480485 -0.3953748
```
4) using *subset()*


```r
subset(air, DepDelay > 60*12)
```

```
##        Year Month DayOfMonth DayOfWeek DepTime CRSDepTime ArrTime
## 71351  2005     7         23         6     633       1545    1212
## 80042  2005     8         20         6    2237       1030     122
## 115794 2005    11          2         3    1009        630    1526
## 168750 2006     4         11         2    1001       1543    1505
## 258029 2006    12         28         4    2232        845     216
## 258288 2006    12         14         4    1118       2156    1908
## 290635 2007     3         22         4     729       1545    1257
## 333067 2007     7         16         1    1200       2021    1522
## 337213 2007     7         16         1     858       1410    1805
## 372929 2007    10         28         7    2155        835      37
## 396990 2007    12         31         1     745       1225    1318
## 415550 2008     2          9         6    1003       1605    1302
## 449626 2008     5         10         6     735       1820    1005
## 454313 2008     5         15         4    2243        850      50
##        CRSArrTime UniqueCarrier FlightNum TailNum ActualElapsedTime
## 71351        2113            NW       354  N586NW               219
## 80042        1301            TZ      4603  N561TZ               345
## 115794       1207            NW       368  N552NW               197
## 168750       2112            NW       354  N541US               184
## 258029       1205            HA        11  N587HA               344
## 258288        527            NW       346  N539US               290
## 290635       2115            NW       354  N504US               208
## 333067       2359            UA       595  N535UA               262
## 337213       2245            AA       194  N5EVAA               367
## 372929       1045            HA        11  N585HA               162
## 396990       1809            NW       360  N525US               213
## 415550       1946            UA        77  N667UA               299
## 449626       2041            UA        39  N669UA               330
## 454313       1055            HA        11  N596HA               307
##        CRSElapsedTime AirTime ArrDelay DepDelay Origin Dest Distance
## 71351             208     184      899      888    SFO  MSP     1589
## 80042             331     323      741      727    SFO  LIH     2447
## 115794            217     175     1639     1659    SFO  MSP     1589
## 168750            209     165     1073     1098    SFO  MSP     1589
## 258029            320     189      851      827    SFO  HNL     2398
## 258288            271     228      821      802    SFO  DTW     2079
## 290635            210     188      942      944    SFO  MSP     1589
## 333067            278     235      923      939    SFO  ANC     2018
## 337213            335     323     1160     1128    SFO  BOS     2704
## 372929            130     123      832      800    SFO  HNL     2398
## 396990            224     191     1149     1160    SFO  MSP     1589
## 415550            341     272     1036     1078    SFO  HNL     2398
## 449626            321     299      804      795    SFO  OGG     2338
## 454313            305     286      835      833    SFO  HNL     2398
##        TaxiIn TaxiOut Cancelled CancellationCode Diverted CarrierDelay
## 71351      13      22         0                         0          888
## 80042       5      17         0                         0          652
## 115794      3      19         0                         0         1639
## 168750      5      14         0                         0         1048
## 258029      8      27         0                         0          827
## 258288     33      29         0                         0          802
## 290635      4      16         0                         0          884
## 333067      4      23         0                         0          844
## 337213     25      19         0                         0          993
## 372929      6      33         0                         0          800
## 396990      9      13         0                         0         1106
## 415550      2      25         0                         0          710
## 449626      6      25         0                         0          795
## 454313      7      14         0                         0          835
##        WeatherDelay NASDelay SecurityDelay LateAircraftDelay DepDelayCens
## 71351             0       11             0                 0           60
## 80042             0       14             0                75           60
## 115794            0        0             0                 0           60
## 168750            6        0             0                19           60
## 258029            0        0             0                24           60
## 258288            0       19             0                 0           60
## 290635            0        0             0                58           60
## 333067            0        0             0                79           60
## 337213            0       32             0               135           60
## 372929            0        0             0                32           60
## 396990            0        0             0                43           60
## 415550            0        0             0               326           60
## 449626            0        9             0                 0           60
## 454313            0        0             0                 0           60
##        DayOfWeekSun
## 71351           Sat
## 80042           Sat
## 115794          Wed
## 168750          Tue
## 258029          Thu
## 258288          Thu
## 290635          Thu
## 333067          Mon
## 337213          Mon
## 372929          Sun
## 396990          Mon
## 415550          Sat
## 449626          Sat
## 454313          Thu
```

5) using *dplyr* tools such as *filter()* and *select()* -- more in Module 5

# Assignment into subsets

We can assign into subsets by using similar syntax, as we saw with vectors.


```r
vec[c(3, 5, 12:14)] <- 1:5
mat[2, 3:5] <- rnorm(3)
mat[mat[,1] > 0, ] <- -Inf
```

# Strings

R has lots of functionality for character strings. Usually these are stored as vectors of strings, each with arbitrary length.


```r
chars <- c('hi', 'hallo', "mother's", 'father\'s', "He said, \"hi\"" )
length(chars)
```

```
## [1] 5
```

```r
nchar(chars)
```

```
## [1]  2  5  8  8 13
```

```r
paste("bill", "clinton", sep = " ")  # paste together a set of strings
```

```
## [1] "bill clinton"
```

```r
paste(chars, collapse = ' ')  # paste together things from a vector
```

```
## [1] "hi hallo mother's father's He said, \"hi\""
```

```r
strsplit("This is the R bootcamp", split = " ")
```

```
## [[1]]
## [1] "This"     "is"       "the"      "R"        "bootcamp"
```

```r
substring(chars, 2, 3)
```

```
## [1] "i"  "al" "ot" "at" "e "
```

```r
chars2 <- chars
substring(chars2, 2, 3) <- "ZZ"
chars2
```

```
## [1] "hZ"              "hZZlo"           "mZZher's"        "fZZher's"       
## [5] "HZZsaid, \"hi\""
```
We can search for patterns in character vectors and replace patterns (both vectorized!)

```r
grep("ther", chars)
```

```
## [1] 3 4
```

```r
gsub("hi", "Hi", chars)
```

```
## [1] "Hi"              "hallo"           "mother's"        "father's"       
## [5] "He said, \"Hi\""
```

# Regular expressions (regex or regexp)

Some of you may be familiar with using *regular expressions*, which is functionality for doing sophisticated pattern matching and replacement with strings. *Python* and *Perl* are both used extensively for such text manipulation. 

R has a full set of regular expression capabilities available through the *grep()*, *gregexpr()*, and *gsub()* functions (among others - many R functions will work with regular expressions). A particularly nice way to make use of this functionality is to use the *stringr* package, which is more user-friendly than directly using the core R functions.

You can basically do any regular expression/string manipulations in R, though the syntax may be a bit clunky at times.

# More details on reading data into R

Remember that you'll need to know the current working directory so that you know where R is looking for files.

The workhorse for reading into a data frame is *read.table()*, which allows any separator (CSV, tab-delimited, etc.). *read.csv()* is a special case of *read.table()* for CSV files.

You've already seen a bit of this, but let's work through a more involved example, so you can see some of the steps and tricks involved in reading data into R.


```r
rta <- read.table("../data/RTAData.csv", sep = ",", head = TRUE)
rta[1:5, 1:5]
```

```
##               time X40010 X40015 X40020 X40025
## 1 2010-03-01 14:58    821    209    828    258
## 2 2010-03-01 15:01    804    209    804    248
## 3 2010-03-01 15:04    892    212    801    237
## 4 2010-03-01 15:07    857    214    821    243
## 5 2010-03-01 15:10    849    222    834    252
```

```r
dim(rta)
```

```
## [1] 120822     62
```

```r
# great, we're all set, right?
# Not so fast...
unlist(lapply(rta, class))[1:5]
```

```
##     time   X40010   X40015   X40020   X40025 
## "factor" "factor" "factor" "factor" "factor"
```

```r
# ?read.table
rta2 <- read.table("../data/RTAData.csv", sep = ",", 
  head = TRUE, stringsAsFactors = FALSE)
rta2[3,3]
```

```
## [1] "212"
```

```r
unlist(lapply(rta2, class))[1:5]
```

```
##        time      X40010      X40015      X40020      X40025 
## "character" "character" "character" "character" "character"
```

```r
# let's delve more deeply
levels(rta[ , 2])[c(1:5, 3041:3044)]
```

```
## [1] ""     "1000" "1001" "1002" "1003" "997"  "998"  "999"  "x"
```

```r
rta3 <- read.table("../data/RTAData.csv", sep = ",", head = TRUE, 
      stringsAsFactors = FALSE, na.strings = c('NA', 'x'))
unlist(lapply(rta3, class))[1:5]
```

```
##        time      X40010      X40015      X40020      X40025 
## "character"   "integer"   "integer"   "integer"   "integer"
```

```r
# checking...
missing <- which(rta[ , 2] == "")
missing[1:5]
```

```
## [1] 1167 1168 1169 1170 1171
```

```r
rta3[head(missing), ]
```

```
##                  time X40010 X40015 X40020 X40025 X40030 X40035 X40040
## 1167 2010-03-04 01:16     NA     NA     NA     NA     NA     NA     NA
## 1168 2010-03-04 01:19     NA     NA     NA     NA     NA     NA     NA
## 1169 2010-03-04 01:22     NA     NA     NA     NA     NA     NA     NA
## 1170 2010-03-04 01:25     NA     NA     NA     NA     NA     NA     NA
## 1171 2010-03-04 01:28     NA     NA     NA     NA     NA     NA     NA
## 1172 2010-03-04 01:31     NA     NA     NA     NA     NA     NA     NA
##      X40045 X40050 X40055 X40060 X40065 X40070 X40075 X40080 X40085 X40090
## 1167     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
## 1168     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
## 1169     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
## 1170     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
## 1171     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
## 1172     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
##      X40092 X40095 X40100 X40105 X40110 X40115 X40120 X40125 X40130 X40135
## 1167     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
## 1168     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
## 1169     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
## 1170     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
## 1171     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
## 1172     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
##      X40140 X40145 X40150 X41010 X41015 X41020 X41025 X41030 X41035 X41040
## 1167     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
## 1168     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
## 1169     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
## 1170     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
## 1171     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
## 1172     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
##      X41045 X41050 X41055 X41060 X41065 X41070 X41075 X41080 X41085 X41090
## 1167     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
## 1168     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
## 1169     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
## 1170     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
## 1171     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
## 1172     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
##      X41095 X41100 X41105 X41110 X41115 X41120 X41125 X41130 X41135 X41140
## 1167     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
## 1168     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
## 1169     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
## 1170     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
## 1171     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
## 1172     NA     NA     NA     NA     NA     NA     NA     NA     NA     NA
##      X41145 X41150 X41155 X41160
## 1167     NA     NA     NA     NA
## 1168     NA     NA     NA     NA
## 1169     NA     NA     NA     NA
## 1170     NA     NA     NA     NA
## 1171     NA     NA     NA     NA
## 1172     NA     NA     NA     NA
```

It's good to first look at your data in plain text format outside of R and then to check it after you've read it into R.

# Other ways to read data into R

The *read.table()* family of functions just skims the surface of things...

1) You can also read in a file as vector of characters, one character string per line of the file with *readLines()*, and then post-process it. 
2) You can read fixed width format (constant number of characters per field) with *read.fwf()*.
3) *read_csv()* (and *read_lines()*, *read_fwf()*, etc.) in the *readr* package is a faster, more helpful drop-in replacement for *read.csv()* that plays well with *dplyr* (see Module 5).
4) the *data.table* package is great for reading and manipulating large datasets (orders of gigabytes or 10s of gigabytes).


# Breakout

### Basics

1) Extract the 5th row from the airline dataset.

2) Extract the last row from the airline dataset.

3) Count the number of NAs in the departure delay field of the airline dataset.

4) Set all of the extreme delays (more than 300 minutes) to NA.

5) Consider the first row of the airline dataset, which has UA flight number 548. How do I create a string "UA-548" using `air$UniqueCarrier[1]` and `air$FlightNum[1]`? 

### Using the ideas

6) Create a character string using `paste()` that tells the user how many rows there are in the data frame - do this programmatically such that it would work for any data frame regardless of how many rows it has. The result should look like this: "There are 56,234,234 rows in the dataset"

7) If you didn't do it this way already, extract the last row from the airline dataset without typing the number '539895'.

8) Create a boolean vector indicating if flight on the carrier WN (Southwest) and distance is less than 1000 miles and calculate the proportion of all the flights these represent.

9) Use that vector to create a new data frame that is a subset of the original data frame.

10) Consider the attributes of the *air* dataset. What kind of R object is the set of attributes?

### Advanced

11) Make the destination variable a factor variable and reorder the levels by the number of arrivals from SFO at the destinations. Now create a boxplot of departure delay by destination for January, 2005.

12) Create row names for the data frame based on concatenating the Year, Month, DayOfMonth, UniqueCarrier and FlightNum fields. You may run into a problem with the data that you need to resolve before you're able to do this...
