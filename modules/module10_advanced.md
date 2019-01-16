% R bootcamp, Module 10: Advanced topics
% January 2019, UC Berkeley
% Chris Paciorek



# This purpose of this module

For some of the topics here, my goal is not to teach you how to fish, but merely to tell you that fish exist, they are delicious, that they can be caught, and where one might go to figure out how to catch them.

# Object-oriented programming (OOP) in R


Confusingly, R has three (well, four) different systems for OOP, and none of them are as elegant and powerful as in Python or other languages more focused on OOP. That said, they get the job done for a lot of tasks.

* S3: informal system used for `lm()`, `glm()`, and many other core features in R in the *stats* package
* S4: more formal system, used with *lme4* 
* Reference Classes and R6: new systems allowing for passing objects by reference

# Basics of object-oriented programming (OOP)

The basic idea is that coding is structured around *objects*, which belong to a *class*, and *methods* that operate on objects in the class.

Objects are like lists, but with methods that are specifically associated with particular classes, as we've seen with the `lm` class.

Objects have fields, analogous to the components of a list. For S4 and reference classes, the fields of the class are fixed and all objects of the class must contain those fields. 

# Working with S3 classes and methods

R has several approaches to object-oriented programming.  These are widely used, albeit a bit klunky. 

The most basic is 'S3' objects. These objects are generally built upon lists.


```r
mod <- lm(gap$lifeExp ~ log(gap$gdpPercap))
class(mod)
```

```
## [1] "lm"
```

```r
is.list(mod)
```

```
## [1] TRUE
```

```r
names(mod)
```

```
##  [1] "coefficients"  "residuals"     "effects"       "rank"         
##  [5] "fitted.values" "assign"        "qr"            "df.residual"  
##  [9] "xlevels"       "call"          "terms"         "model"
```

```r
mod$coefficients
```

```
##        (Intercept) log(gap$gdpPercap) 
##          -9.100889           8.405085
```

```r
mod[['coefficients']]
```

```
##        (Intercept) log(gap$gdpPercap) 
##          -9.100889           8.405085
```

```r
mod[[1]]
```

```
##        (Intercept) log(gap$gdpPercap) 
##          -9.100889           8.405085
```

The magic of OOP here is that methods (i.e., functions) can be tailored to work specifically with specific kinds of objects.


```r
summary(gap$lifeExp)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##   23.60   48.20   60.71   59.47   70.85   82.60
```

```r
summary(mod)
```

```
## 
## Call:
## lm(formula = gap$lifeExp ~ log(gap$gdpPercap))
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -32.778  -4.204   1.212   4.658  19.285 
## 
## Coefficients:
##                    Estimate Std. Error t value Pr(>|t|)    
## (Intercept)         -9.1009     1.2277  -7.413 1.93e-13 ***
## log(gap$gdpPercap)   8.4051     0.1488  56.500  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 7.62 on 1702 degrees of freedom
## Multiple R-squared:  0.6522,	Adjusted R-squared:  0.652 
## F-statistic:  3192 on 1 and 1702 DF,  p-value: < 2.2e-16
```

**Question**: What do you think R is doing behind the scenes?

Consider `summary.lm`.

# More on working with S3 classes and methods 


```r
library(methods)
yb <- gap$lifeExp > 75
yc <- gap$lifeExp
x <- log(gap$gdpPercap)
mod1 <- lm(yc ~ x)
mod2 <- glm(yb ~ x, family = binomial)
mod2$residuals[1:20] # access field with list-like syntax
```

```
##         1         2         3         4         5         6         7 
## -1.000026 -1.000031 -1.000035 -1.000033 -1.000022 -1.000027 -1.000054 
##         8         9        10        11        12        13        14 
## -1.000035 -1.000015 -1.000014 -1.000021 -1.000053 -1.000258 -1.000477 
##        15        16        17        18        19        20 
## -1.000832 -1.001461 -1.002613 -1.003204 -1.003496 -1.003838
```

```r
class(mod2)
```

```
## [1] "glm" "lm"
```

```r
is(mod2, "lm")
```

```
## [1] TRUE
```

```r
is.list(mod2)
```

```
## [1] TRUE
```

```r
names(mod2)
```

```
##  [1] "coefficients"      "residuals"         "fitted.values"    
##  [4] "effects"           "R"                 "rank"             
##  [7] "qr"                "family"            "linear.predictors"
## [10] "deviance"          "aic"               "null.deviance"    
## [13] "iter"              "weights"           "prior.weights"    
## [16] "df.residual"       "df.null"           "y"                
## [19] "converged"         "boundary"          "model"            
## [22] "call"              "formula"           "terms"            
## [25] "data"              "offset"            "control"          
## [28] "method"            "contrasts"         "xlevels"
```

```r
methods(class = "glm")
```

```
##  [1] add1           anova          coerce         confint       
##  [5] cooks.distance deviance       drop1          effects       
##  [9] extractAIC     family         formula        influence     
## [13] initialize     logLik         model.frame    nobs          
## [17] predict        print          profile        residuals     
## [21] rstandard      rstudent       show           slotsFromS3   
## [25] summary        vcov           weights       
## see '?methods' for accessing help and source code
```

```r
methods(predict)
```

```
##  [1] predict.ar*                predict.Arima*            
##  [3] predict.arima0*            predict.bs*               
##  [5] predict.bSpline*           predict.glm               
##  [7] predict.glmmPQL*           predict.gls*              
##  [9] predict.gnls*              predict.HoltWinters*      
## [11] predict.lda*               predict.lm                
## [13] predict.lme*               predict.lmList*           
## [15] predict.lmList4*           predict.loess*            
## [17] predict.lqs*               predict.mca*              
## [19] predict.merMod*            predict.mlm*              
## [21] predict.nbSpline*          predict.nlme*             
## [23] predict.nls*               predict.npolySpline*      
## [25] predict.ns*                predict.pbSpline*         
## [27] predict.polr*              predict.poly*             
## [29] predict.polySpline*        predict.ppolySpline*      
## [31] predict.ppr*               predict.prcomp*           
## [33] predict.princomp*          predict.qda*              
## [35] predict.rlm*               predict.smooth.spline*    
## [37] predict.smooth.spline.fit* predict.StructTS*         
## see '?methods' for accessing help and source code
```

```r
predict
```

```
## function (object, ...) 
## UseMethod("predict")
## <bytecode: 0x3f92288>
## <environment: namespace:stats>
```

```r
# predict.glm
```

When `predict()` is called on a GLM object, it first calls the generic `predict()`, which then recognizes that the first argument is of the class *glm* and immediately calls the right class-specific method, `predict.glm()` in this case.

# Making your own S3 class/object/method

Making an object and class-specific methods under S3 is simple. 


```r
rboot2018 <- list(month = 'August', year = 2015, 
  instructor = 'Paciorek', attendance = 100)
class(rboot2018) <- "workshop"

rboot2018
```

```
## $month
## [1] "August"
## 
## $year
## [1] 2015
## 
## $instructor
## [1] "Paciorek"
## 
## $attendance
## [1] 100
## 
## attr(,"class")
## [1] "workshop"
```

```r
is(rboot2018, "workshop")
```

```
## [1] TRUE
```

```r
rboot2018$instructor 
```

```
## [1] "Paciorek"
```

```r
print.workshop <- function(x) {
    with(x,
       cat("A workshop held in ", month, " ", year, "; taught by ", instructor, ".\nThe attendance was ", attendance, ".\n", sep = ""))
    invisible(x)
}

# doesn't execute correctly in the slide creation, so comment out here:
# rboot2018 
```
 
Note that we rely on the generic `print()` already existing in R. Otherwise we'd need to create it.

So what is happening behind the scenes here?

# Using S4 classes and methods

Unlike S4, S4 classes have a formal definition and objects in the class must have the specified fields. 

The fields of an S4 class are called 'slots'. Instead of `x$field` you do `x@field`.

Here's a bit of an example of an S4 class for a linear mixed effects model from *lme4*:


```r
library(lme4)
library(methods)
fm1 <- lmer(Reaction ~ Days + (Days|Subject), sleepstudy)
class(fm1)
```

```
## [1] "lmerMod"
## attr(,"package")
## [1] "lme4"
```

```r
methods(class = "lmerMod")
```

```
## [1] getL show
## see '?methods' for accessing help and source code
```

```r
slotNames(fm1)
```

```
##  [1] "resp"    "Gp"      "call"    "frame"   "flist"   "cnms"    "lower"  
##  [8] "theta"   "beta"    "u"       "devcomp" "pp"      "optinfo"
```

```r
fm1$theta
```

```
## Error in fm1$theta: $ operator not defined for this S4 class
```

```r
fm1@theta
```

```
## [1] 0.96673279 0.01516906 0.23090960
```

# A brief mention of Reference Classes

Reference classes are new in the last few years; for more information, see `?ReferenceClasses`.

Reference classes are somewhat like S4 in that a class is formally defined and R is careful about the fields of the class.

Methods for reference classes can modify the contents of fields in an object invisibly without the need to copy over the entire object.



# Error and warning messages

When you write your own functions, and particularly for distributing to others, it's a good idea to:

* Check for possible errors (particularly in the input arguments) and give the user an informative error message
* Warn them if you're doing something they might not have anticipated

We can use `stop()` and `warning()` to do this. They're the same functions that are being called when you see an error message or a warning in reaction to your own work in R.


```r
mysqrt <- function(x) {
  if(is.list(x)) {
    warning("x is a list; converting to a vector")
    x <- unlist(x)
  }
  if(!is.numeric(x)) {
    stop("What is the square root of 'bob'?")
  } else {
      if(any(x < 0)) {
        warning("mysqrt: found negative values; proceeding anyway")
        x[x >= 0] <- (x[x >= 0])^(1/2)
        x[x < 0] <- NaN
        return(x)
      } else return(x^(1/2))
  }
}

mysqrt(c(1, 2, 3))
```

```
## [1] 1.000000 1.414214 1.732051
```

```r
mysqrt(c(5, -7))
```

```
## Warning in mysqrt(c(5, -7)): mysqrt: found negative values; proceeding
## anyway
```

```
## [1] 2.236068      NaN
```

```r
mysqrt(c('asdf', 'sdf'))
```

```
## Error in mysqrt(c("asdf", "sdf")): What is the square root of 'bob'?
```

```r
mysqrt(list(5, 3, 'ab'))
```

```
## Warning in mysqrt(list(5, 3, "ab")): x is a list; converting to a vector
```

```
## Error in mysqrt(list(5, 3, "ab")): What is the square root of 'bob'?
```

```r
sqrt(c(5, -7))
```

```
## Warning in sqrt(c(5, -7)): NaNs produced
```

```
## [1] 2.236068      NaN
```

```r
sqrt('asdf')
```

```
## Error in sqrt("asdf"): non-numeric argument to mathematical function
```

```r
sqrt(list(5, 3, 2))
```

```
## Error in sqrt(list(5, 3, 2)): non-numeric argument to mathematical function
```

So we've done something similar to what `sqrt()` actually does in R.

# 'Catching' errors

When you automate analyses, sometimes an R function call will fail. But you don't want all of your analyses to grind to a halt because one failed. Rather, you want to catch the error, record that it failed, and move on.

For me this is most critical when I'm doing stratified analyses or sequential operations.

The `try()` function is a powerful tool here.

# Why we need to `try()`

Suppose we tried to do a stratified analysis of life expectancy on GDP within continents, for 2007. I'm going to do this as a for loop for pedagogical reasons, but again, it would be better to do this with dplyr/lapply/by type tools.

For the purpose of illustration, I'm going to monkey a bit with the data such that there is an error in fitting Oceania. This is artificial, but when you stratify data into smaller groups it's not uncommon that the analysis can fail for one of the groups (often because of small sample size or missing data).



```r
mod <- list()
fakedat <- gap[gap$year == 2007, ]
fakedat$gdpPercap[fakedat$continent == 'Oceania'] <- NA

for(cont in c('Asia', 'Oceania', 'Europe', 'Americas', 'Africa')) {
            cat("Fitting model for continent ", cont, ".\n")
            tmp <- subset(fakedat, continent == cont)
            mod[[cont]] <- lm(lifeExp ~ log(gdpPercap), data = tmp)
}
```

```
## Fitting model for continent  Asia .
## Fitting model for continent  Oceania .
```

```
## Error in lm.fit(x, y, offset = offset, singular.ok = singular.ok, ...): 0 (non-NA) cases
```

What happened?


# How we can `try()` harder


```r
mod <- list()

for(cont in c('Asia', 'Oceania', 'Europe', 'Americas', 'Africa')) {
            cat("Fitting model for continent ", cont, ".\n")
            tmp <- subset(fakedat, continent == cont)
            curMod <- try(lm(lifeExp ~ log(gdpPercap), data = tmp))
            if(is(curMod, "try-error")) mod[[cont]] <- NA 
                       else mod[[cont]] <- curMod            
}
```

```
## Fitting model for continent  Asia .
## Fitting model for continent  Oceania .
## Fitting model for continent  Europe .
## Fitting model for continent  Americas .
## Fitting model for continent  Africa .
```

```r
mod[[1]]
```

```
## 
## Call:
## lm(formula = lifeExp ~ log(gdpPercap), data = tmp)
## 
## Coefficients:
##    (Intercept)  log(gdpPercap)  
##         25.650           5.157
```

```r
mod[[2]]
```

```
## [1] NA
```

# Computing on the language

One of the powerful capabilities you have in R is the ability to use R to modify and create R code. 

First we need to understand a bit about how R code is stored and manipulated when we don't want to immediately evaluate it.

When you send some code to R to execute, it has to 'parse' the input; i.e., to process it so that it know how to evaluate it. The parsed input can then be evaluated in the proper context (i.e., the right frame, holding the objects to be operated on and created). 

We can capture parsed code before it is evaluated, manipulate it, and execute the modified result.

# Capturing and evaluating parsed code


```r
code <- quote(n <- 100)
code
```

```
## n <- 100
```

```r
class(code)
```

```
## [1] "<-"
```

```r
n
```

```
## Error in eval(expr, envir, enclos): object 'n' not found
```

```r
eval(code)
n
```

```
## [1] 100
```

```r
results <- rep(0, n)
moreCode <- quote(for(i in 1:n) {
    tmp <- rnorm(30)
    results[i] <- min(tmp)
})
class(moreCode)
```

```
## [1] "for"
```

```r
as.list(moreCode)
```

```
## [[1]]
## `for`
## 
## [[2]]
## i
## 
## [[3]]
## 1:n
## 
## [[4]]
## {
##     tmp <- rnorm(30)
##     results[i] <- min(tmp)
## }
```

```r
newN <- 200
codeText <- paste("n", "<-", newN)
codeText
```

```
## [1] "n <- 200"
```

```r
codeFromText <- parse(text = codeText)
codeFromText
```

```
## expression(n <- 200)
```

```r
eval(codeFromText)
n
```

```
## [1] 200
```

So you could use R's string manipulation capabilities to write and then evaluate R code. Meta.

# Using R to automate working with object names 

Suppose you were given a bunch of objects named "x1", "x2", "x3", ... and you wanted to write code to automatically do some computation on them. Here I'll just demonstrate with three, but this is obviously more compelling if there are many of them.


```r
# assume these objects were provided to you
x1 <- rnorm(5)
x2 <- rgamma(10, 1)
x3 <- runif(20)
# now you want to work with them
nVals <- 3
results <- rep(0, nVals)
for(i in 1:nVals) { 
  varName <- paste("x", i, sep = "")
  tmp <- eval(as.name(varName))
  # tmp <- get(varName) # an alternative
   results[i] <- mean(tmp)
}
results
```

```
## [1] 0.1351357 0.8783309 0.4669116
```

```r
varName
```

```
## [1] "x3"
```

```r
tmp
```

```
##  [1] 0.52971958 0.78935623 0.02333120 0.47723007 0.73231374 0.69273156
##  [7] 0.47761962 0.86120948 0.43809711 0.24479728 0.07067905 0.09946616
## [13] 0.31627171 0.51863426 0.66200508 0.40683019 0.91287592 0.29360337
## [19] 0.45906573 0.33239467
```

Or suppose you needed to create "x1", "x2", "x3", automatically.


```r
nVals <- 3
results <- rep(0, nVals)
for(i in 1:nVals) {  
   varName <- paste("x", i, sep = "")
   assign(varName, rnorm(10))
}
x2
```

```
##  [1]  0.6969634  0.5566632 -0.6887557 -0.7074952  0.3645820  0.7685329
##  [7] -0.1123462  0.8811077  0.3981059 -0.6120264
```

Can you think of any uses of this ability for R to self-generate?

# File encodings

Text (either in the form of a file with regular language in it or a data file with fields of character strings) will often contain characters that are not part of the [limited ASCII set of characters](http://en.wikipedia.org/wiki/ASCII), which has 128 characters and control codes; basically what you see on a standard US keyboard.

UTF-8 is an encoding for the Unicode characters that include more than 110,000 characters from 100 different alphabets/scripts. It's widely used on the web.

Latin-1 encodes a small subset of Unicode and contains the characters used in many European languages (e.g., letters with accents).

# Dealing with encodings in R

To read files with other characters correctly into R, you may need to tell R what encoding the file is in. E.g., see help on `read.table()` for the *fileEncoding* and *encoding* arguments. 

With strings already in R, you can convert between encodings with `iconv()`:

```r
text <- "Melhore sua seguran\xe7a"
iconv(text, from = "latin1", to = "UTF-8")
```

```
## [1] "Melhore sua segurança"
```

```r
iconv(text, from = "latin1", to = "ASCII", sub = "???")
```

```
## [1] "Melhore sua seguran???a"
```

You can mark a string with an encoding so R can display it correctly:

```r
x <- "fa\xE7ile"
Encoding(x) <- "latin1"
x
```

```
## [1] "façile"
```

```r
# playing around...
x <- "\xa1 \xa2 \xa3 \xf1 \xf2"
Encoding(x) <- "latin1"
x
```

```
## [1] "¡ ¢ £ ñ ò"
```

# Line endings in text files

Windows, Mac, and Linux handle line endings in text files somewhat differently. So if you read a text file into R that was created in a different operating system you can run into difficulties.

* In Windows lines end in both a newline (the ASCII character `\n`) and a carriage return (`\r`). 
* In UNIX and Mac OS X, lines end in only a newline.


So in UNIX you might see `^M` at the end of lines when you open a Windows file in a text editor. The *dos2unix* or *fromdos* in UNIX commands can do the necessary conversion

In Windows you might have a UNIX text file appear to be all one line. The *unix2dos* or *todos* commands in UNIX can do the conversion. 

There may also be Windows tools to deal with this. 

As a side note, before Mac OS X, lines ended only in a carriage return.
There is a UNIX utility call *mac2unix* that can convert such text files.

# Working with databases

R has the capability to read and write from a variety of relational database management systems (DBMS). Basically a database is a collection of rectangular format datasets (tables). Some of these tables have fields in common so it makes sense to merge (i.e., join) information from multiple tables. E.g., you might have a database with a table of student information, a table of teacher information and a table of school information. 

The *DBI* package provides a front-end for manipulating databases from a variety of DBMS (MySQL, SQLite, Oracle, among others)

Basically, you tell the package what DBMS is being used on the back-end, link to the actual database, and then you can use the standard functions in the package regardless of the back-end. 

# Database example

You can get an example database of information about Stack Overflow questions and answers from [http://www.stat.berkeley.edu/share/paciorek/stackoverflow-2016.db](http://www.stat.berkeley.edu/share/paciorek/stackoverflow-2016.db). Stack Overflow is a website where programmers and software users can pose questions and get answers to technical problems.



```r
library(RSQLite)  # DBI is a dependency
db <- dbConnect(SQLite(), dbname = "../data/stackoverflow-2016.db") 
## stackoverflow-2016.db is an SQLite database

## metadata
dbListTables(db)
```

```
## [1] "answers"          "maxRepByQuestion" "questions"       
## [4] "questionsAugment" "questions_tags"   "users"
```

```r
dbListFields(db, "questions")
```

```
## [1] "questionid"   "creationdate" "score"        "viewcount"   
## [5] "title"        "ownerid"
```

```r
## simple filter operation
popular <- dbGetQuery(db, "select * from questions 
   where viewcount > 10000")
## a join followed by a filter operation
popularR <- dbGetQuery(db, "select * from questions join questions_tags
   on questions.questionid = questions_tags.questionid
   where viewcount > 10000 and
   tag = 'r'")
```

# Computer architecture

Note to participants: I'm having trouble with parallelization in RStudio, so we'll just run the demo code in this module in a command line R session. You can open the basic R GUI for Mac or Windows, or, on a Mac, start R in a terminal window.

* Modern computers have multiple processors and clusters/supercomputers have multiple networked machines, each with multiple processors.
* The key to increasing computational efficiency in these contexts is breaking up the work amongst the processors.
* Processors on a single machine (or 'node') share memory and don't need to carry out explicit communication (shared memory computation)
* Processors on separate machines need to pass data across a network, often using the MPI protocol (distributed memory computation)

We'll focus on shared memory computation here.

# How do I know how many cores a computer has?

* Linux - count the processors listed in */proc/cpuinfo* or use `nproc`
* Mac - in a terminal: `system_profiler | grep -i 'Cores'`
* Windows - count the number of graphs shown for CPU Usage (or CPU Usage History) under "Task Manager->Performance", or [try this program](http://www.cpuid.com/cpuz.php) 
 
To see if multiple cores are being used by your job, you can do:

* Mac/Linux - use *top* or *ps*
* Windows - see the "Task Manager->Performance->CPU Usage"

# How can we make use of multiple cores?

Some basic approaches are:

* Use a linear algebra package that distributes computations across 'threads'
* Spread independent calculations (embarrassingly parallel problems) across multiple cores
    - *for* loops with independent calculations
    - parallelizing `apply()` and its variants


# Threaded linear algebra

R comes with a default BLAS (basic linear algebra subroutines) and LAPACK (linear algebra package) that carry out the core linear algebra computations. However, you can generally improve performance (sometimes by an order of magnitude) by using a different BLAS. Furthermore a threaded BLAS will allow you to use multiple cores.

A 'thread' is a lightweight process, and the operating system sees multiple threads as part of a single process.

* For Linux, *openBLAS*, Intel's *MKL* and AMD's *ACML* are both fast and threaded. On the SCF we have openBLAS on the compute servers and ACML on the Linux cluster and R uses these for linear algebra.
* For Mac, Apple's *vecLib* (in a library called *libRblas.vecLib.dylib*) is fast and threaded.
* For Windows, you're probably out of luck.

We'll show by demonstration that my Mac laptop is using multiple cores for linear algebra operations.


```r
# note to CJP: don't run in RStudio
n <- 5000
x <- matrix(rnorm(n^2), n)
U <- chol(crossprod(x))
```

You should see that your R process is using more than 100% of CPU. Inconceivable!

# More details on the BLAS

You can talk with your systems administrator about linking R to a fast BLAS or you can look into it yourself for your personal machine; see the [R Installation and Administration manual](http://www.cran.r-project.org/manuals.html).

Note that in some cases, in particular for small matrix operations, using multiple threads may actually slow down computation, so you may want to experiment, particularly with Linux. You can force the linear algebra to use only a single core by doing (assuming you're using the bash shell) `export OMP_NUM_THREADS=1` in the terminal window *before* starting R in the same terminal. Or see the *RhpcBLASctl* package to do it from within R.
 
Finally, note that threaded BLAS and either `foreach` or parallel versions of `apply()` can conflict and cause R to hang, so you're likely to want to set the number of threads to 1 as above if you're doing explicit parallelization. 

# What is an embarrassingly parallel (EP) problem?

Do you think you should be asking? 

An EP problem is one that can be solved by doing independent computations as separate processes without communication between the processes. You can get the answer by doing separate tasks and then collecting the results. 

Examples in statistics include

1. stratified analyses
2. cross-validation
4. simulations with many independent replicates
5. bootstrapping
3. random forests models

Can you think of others in your work?

Some things that are not EP (at least not in a basic formulation):

1. Markov chain Monte Carlo for fitting Bayesian models
2. optimization

# Using multiple cores for EP problems: *foreach*

First, make sure your iterations are independent and don't involve sequential calculations!

The *foreach* package provides a way to do a for loop using multiple cores. It can use a variety of 'back-ends' that handle the nitty-gritty of the parallelization. 

To use multiple cores on a single machine, use the *parallel* back-end from the *doParallel* package.

We'll use a new dataset here, which is a dataset of airline departure times (in particular delays) for all flights from SFO over a period of several years. We'll do a stratified analysis, fitting a GAM (see Unit 7) for each of the destination airports.


```r
# note to CJP: don't run in RStudio
library(parallel)
library(doParallel)
```

```
## Loading required package: foreach
```

```
## Loading required package: iterators
```

```r
library(foreach)

nCores <- 4  # actually only 2 on my laptop, but appears hyperthreaded
registerDoParallel(nCores)

fitFun <- function(curDest) {
            library(mgcv)
            tmp <- subset(air, Dest == curDest)
            tmp$Hour <- tmp$CRSDepTime %/% 100
            curMod <- try(gam(DepDelay ~ Year + s(Month) + s(Hour) + 
                 as.factor(DayOfWeek), data = tmp))
            if(is(tmp, "try-error")) curMod <- NA 
            return(curMod)
}

out <- foreach(dest = unique(air$Dest)) %dopar% {
    cat("Starting job for ", dest, ".\n", sep = "")
    outSub <- fitFun(dest)
    cat("Finishing job for ", dest, ".\n", sep = "")
    outSub # this will become part of the out objec
}
out[1:5]
```

```
## [[1]]
## 
## Family: gaussian 
## Link function: identity 
## 
## Formula:
## DepDelay ~ Year + s(Month) + s(Hour) + as.factor(DayOfWeek)
## 
## Estimated degrees of freedom:
## 7.51 7.49  total = 23 
## 
## GCV score: 1051.858     
## 
## [[2]]
## 
## Family: gaussian 
## Link function: identity 
## 
## Formula:
## DepDelay ~ Year + s(Month) + s(Hour) + as.factor(DayOfWeek)
## 
## Estimated degrees of freedom:
## 7.46 8.64  total = 24.1 
## 
## GCV score: 1325.755     
## 
## [[3]]
## 
## Family: gaussian 
## Link function: identity 
## 
## Formula:
## DepDelay ~ Year + s(Month) + s(Hour) + as.factor(DayOfWeek)
## 
## Estimated degrees of freedom:
## 5.93 8.70  total = 22.63 
## 
## GCV score: 639.5956     
## 
## [[4]]
## 
## Family: gaussian 
## Link function: identity 
## 
## Formula:
## DepDelay ~ Year + s(Month) + s(Hour) + as.factor(DayOfWeek)
## 
## Estimated degrees of freedom:
## 7.84 8.51  total = 24.35 
## 
## GCV score: 880.4363     
## 
## [[5]]
## 
## Family: gaussian 
## Link function: identity 
## 
## Formula:
## DepDelay ~ Year + s(Month) + s(Hour) + as.factor(DayOfWeek)
## 
## Estimated degrees of freedom:
## 8.00 5.68  total = 21.69 
## 
## GCV score: 1367.66
```

**Question**: What do you think are the advantages and disadvantages of having many small tasks vs. a few large tasks?

# Using multiple cores for EP problems: parallel *apply* variants

Note there may be  errors in the HTML output from the code below, but the code should work when running it directly.

`help(clusterApply)` shows the wide variety of parallel versions of the *apply* family of functions.

Here's `parLapply`:



```r
library(parallel)
nCores <- 4
cluster <- makeCluster(nCores)
cluster
```

```
## socket cluster with 4 nodes on host 'localhost'
```

```r
# you may need to load all required libraries within
# the function being applied
# note this produces an error when run during creation of this HTML,
# but should be fine in a standard R session
clusterExport(cluster, 'air')
```

```
## Error in get(name, envir = envir): object 'air' not found
```

```r
out <- parLapply(cluster, unique(air$Dest), fitFun)
out[1:5]
```

```
## [[1]]
## 
## Family: gaussian 
## Link function: identity 
## 
## Formula:
## DepDelay ~ Year + s(Month) + s(Hour) + as.factor(DayOfWeek)
## 
## Estimated degrees of freedom:
## 7.51 7.49  total = 23 
## 
## GCV score: 1051.858     
## 
## [[2]]
## 
## Family: gaussian 
## Link function: identity 
## 
## Formula:
## DepDelay ~ Year + s(Month) + s(Hour) + as.factor(DayOfWeek)
## 
## Estimated degrees of freedom:
## 7.46 8.64  total = 24.1 
## 
## GCV score: 1325.755     
## 
## [[3]]
## 
## Family: gaussian 
## Link function: identity 
## 
## Formula:
## DepDelay ~ Year + s(Month) + s(Hour) + as.factor(DayOfWeek)
## 
## Estimated degrees of freedom:
## 5.93 8.70  total = 22.63 
## 
## GCV score: 639.5956     
## 
## [[4]]
## 
## Family: gaussian 
## Link function: identity 
## 
## Formula:
## DepDelay ~ Year + s(Month) + s(Hour) + as.factor(DayOfWeek)
## 
## Estimated degrees of freedom:
## 7.84 8.51  total = 24.35 
## 
## GCV score: 880.4363     
## 
## [[5]]
## 
## Family: gaussian 
## Link function: identity 
## 
## Formula:
## DepDelay ~ Year + s(Month) + s(Hour) + as.factor(DayOfWeek)
## 
## Estimated degrees of freedom:
## 8.00 5.68  total = 21.69 
## 
## GCV score: 1367.66
```

And here is `mclapply`, which is similar:


```r
library(parallel)
nCores <- 4
out2 <- mclapply(unique(air$Dest), fitFun, mc.cores = nCores)
out2[1:5]
```

```
## [[1]]
## 
## Family: gaussian 
## Link function: identity 
## 
## Formula:
## DepDelay ~ Year + s(Month) + s(Hour) + as.factor(DayOfWeek)
## 
## Estimated degrees of freedom:
## 7.51 7.49  total = 23 
## 
## GCV score: 1051.858     
## 
## [[2]]
## 
## Family: gaussian 
## Link function: identity 
## 
## Formula:
## DepDelay ~ Year + s(Month) + s(Hour) + as.factor(DayOfWeek)
## 
## Estimated degrees of freedom:
## 7.46 8.64  total = 24.1 
## 
## GCV score: 1325.755     
## 
## [[3]]
## 
## Family: gaussian 
## Link function: identity 
## 
## Formula:
## DepDelay ~ Year + s(Month) + s(Hour) + as.factor(DayOfWeek)
## 
## Estimated degrees of freedom:
## 5.93 8.70  total = 22.63 
## 
## GCV score: 639.5956     
## 
## [[4]]
## 
## Family: gaussian 
## Link function: identity 
## 
## Formula:
## DepDelay ~ Year + s(Month) + s(Hour) + as.factor(DayOfWeek)
## 
## Estimated degrees of freedom:
## 7.84 8.51  total = 24.35 
## 
## GCV score: 880.4363     
## 
## [[5]]
## 
## Family: gaussian 
## Link function: identity 
## 
## Formula:
## DepDelay ~ Year + s(Month) + s(Hour) + as.factor(DayOfWeek)
## 
## Estimated degrees of freedom:
## 8.00 5.68  total = 21.69 
## 
## GCV score: 1367.66
```

One thing to keep in mind is whether the different tasks all take about the same amount of time or widely different times. In the latter case, one wants to sequentially dispatch tasks as earlier tasks finish, rather than dispatching a block of tasks to each core. Some of these parallel *apply* variants allow you to control this. 

# Parallelization and Random Number Generation

A tale of the good, the bad, and the ugly

Random numbers on a computer are [not truly random](http://dilbert.com/strips/comic/2001-10-25) but are generated as a sequence of pseudo-random numbers. The sequence is finite (but very, very, very, very long) and eventally repeats itself. 

A random number seed determines where in the sequence one starts when generating random numbers.

* The ugly: Make sure you do not use the same seed for each task

```r
set.seed(1)
rnorm(5)
```

```
## [1] -0.6264538  0.1836433 -0.8356286  1.5952808  0.3295078
```

```r
set.seed(1)
rnorm(5)
```

```
## [1] -0.6264538  0.1836433 -0.8356286  1.5952808  0.3295078
```
* The (not so) bad: Use a different seed for each task or each process. It's possible the subsequences will overlap but quite unlikely.

* The good: Use the L'Ecuyer algorithm (`library(lecuyer)`) to ensure distinct subsequences
    - with `foreach` you can use `%dorng%` from the *doRNG* package in place of `%dopar%`
    - with `mclapply()`, use the `mc.set.seed` argument (see `help(mcparallel)`) 

* The ugly but good: Generate all your random numbers in the master process and distribute them to the tasks if feasible.

The syntax for using L'Ecuyer is available in my [parallel computing tutorial](https://github.com/berkeley-scf/tutorial-parallel-basics).

# A brief note on distributed computing for advanced users

If you have access to multiple machines within a networked environment, such as the compute servers in the Statistics Department or the campus-wide Savio cluster or machines on Amazon's EC2 service, there are a few (sometimes) straightforward ways to parallelize EP jobs across machines.

1. Use `foreach` with *doMPI* as the back-end. You'll need *MPI* and *Rmpi* installed on all machines.
2. Use `foreach` with *doSNOW* as the backend and start a cluster of type "SOCK" to avoid needing MPI/Rmpi. 
3. Use sockets to make a cluster in R and then use `parLapply()`, `parSapply()`, `mclapply()`, etc.
4. Use the *future* package.

See my [tutorial on distributed computing](https://github.com/berkeley-scf/tutorial-parallel-distributed) for syntax and more details. 

