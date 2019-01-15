% R bootcamp, Module 2: Managing R and R resources
% January 2019, UC Berkeley
% Chris Paciorek



# Managing your objects

R has a number of functions for getting metadata about your objects. Some of this is built in to RStudio.


```r
v1 <- gap$year
v2 <- gap$continent
v3 <- gap$lifeExp

length(v1)
```

```
## [1] 1704
```

```r
str(v1)
```

```
##  int [1:1704] 1952 1957 1962 1967 1972 1977 1982 1987 1992 1997 ...
```

```r
class(v1)
```

```
## [1] "integer"
```

```r
typeof(v1)
```

```
## [1] "integer"
```

```r
class(v2)
```

```
## [1] "character"
```

```r
typeof(v2)
```

```
## [1] "character"
```

```r
class(v3)
```

```
## [1] "numeric"
```

```r
typeof(v3)
```

```
## [1] "double"
```

```r
is.vector(v1)
```

```
## [1] TRUE
```

```r
is.list(v1)
```

```
## [1] FALSE
```

```r
# only present if code from module 1 run in this R session
is.list(myList)
```

```
## Error in eval(expr, envir, enclos): object 'myList' not found
```

```r
is.vector(myList)
```

```
## Error in is.vector(myList): object 'myList' not found
```

```r
is.data.frame(gap)
```

```
## [1] TRUE
```

```r
is.list(gap)
```

```
## [1] TRUE
```

**Question**: What have you learned? Does it make sense? 

# Managing and saving the workspace

R has functions for learning about the collection of objects in your workspace. Some of this is built in to RStudio.


```r
x <- rnorm(5)
y <- c(5L, 2L, 7L)
z <- list(a = 3, b = c('sam', 'yang'))
ls()  # search the user workspace (global environment)
```

```
## [1] "gap" "v1"  "v2"  "v3"  "x"   "y"   "z"
```

```r
rm(x)    # delete a variable
ls()
```

```
## [1] "gap" "v1"  "v2"  "v3"  "y"   "z"
```

```r
ls.str() # list and describe variables
```

```
## gap : 'data.frame':	1704 obs. of  6 variables:
##  $ country  : chr  "Afghanistan" "Afghanistan" "Afghanistan" "Afghanistan" ...
##  $ year     : int  1952 1957 1962 1967 1972 1977 1982 1987 1992 1997 ...
##  $ pop      : num  8425333 9240934 10267083 11537966 13079460 ...
##  $ continent: chr  "Asia" "Asia" "Asia" "Asia" ...
##  $ lifeExp  : num  28.8 30.3 32 34 36.1 ...
##  $ gdpPercap: num  779 821 853 836 740 ...
## v1 :  int [1:1704] 1952 1957 1962 1967 1972 1977 1982 1987 1992 1997 ...
## v2 :  chr [1:1704] "Asia" "Asia" "Asia" "Asia" "Asia" "Asia" "Asia" "Asia" ...
## v3 :  num [1:1704] 28.8 30.3 32 34 36.1 ...
## y :  int [1:3] 5 2 7
## z : List of 2
##  $ a: num 3
##  $ b: chr [1:2] "sam" "yang"
```

Finally we can save the objects in our R session:

```r
ls()
```

```
## [1] "gap" "v1"  "v2"  "v3"  "y"   "z"
```

```r
save.image('module2.Rda')
rm(list = ls())
ls()
```

```
## character(0)
```

```r
load('module2.Rda') 
# the result of this may not be quite right in the slide version
ls()
```

```
##  [1] "a"                "D2R"              "deepExtract"     
##  [4] "denslines"        "densplot"         "dim2"            
##  [7] "ellipse.default"  "f.angdist"        "f.ciplot"        
## [10] "f.dplot"          "f.ess"            "f.ess.old"       
## [13] "f.flushplot"      "f.gm"             "f.grstat"        
## [16] "f.identity"       "f.invlogit"       "f.logit"         
## [19] "f.logmatern.euc"  "f.lonlat2eucl"    "f.matern.ang"    
## [22] "f.matern.ang.cov" "f.matern.euc"     "f.merge"         
## [25] "format_bytes"     "f.rdist.earth"    "f.sort"          
## [28] "f.sort2"          "f.squexp"         "f.trimat"        
## [31] "f.vecrep"         "getNcdf"          "im"              
## [34] "indices"          "ls_sizes"         "machineName"     
## [37] "makePoly"         "module"           "plot.ell"        
## [40] "pmap"             "pmap2"            "pointsInPoly"    
## [43] "pplot"            "pretty_size"      "print.closeR"    
## [46] "q"                "R2"               "R2D"             
## [49] "rcsv"             "rotate"           "sizes"           
## [52] "source"           "temp.colors"      "thresh"          
## [55] "tplot"            "tsplot"           "wcsv"
```



**Challenge**: how would I find all of my objects that have 'x' in their names?

# Packages (R's killer app)

Let's check out the [packages on CRAN](https://cran.r-project.org/web/packages/). In particular check out the [CRAN Task Views](https://cran.r-project.org/web/views/).

Essentially any well-established and many not-so-established statistical methods and other functionality is available in a package. 

If you want to sound like an R expert, make sure to call them *packages* and not *libraries*. A *library* is the location in the directory structure where the packages are installed/stored.

### Using packages

Two steps:

1. Install the package on your machine
2. Load the package

To install a package, in RStudio, just do `Packages->Install Packages`.

From the command line, you generally will just do

```r
install.packages('fields') 
```
That should work without specifying the repository from which to download the package (though sometimes you will be given a menu of repositories from which to select) but sometimes you'll get errors indicating a package is not available for your version of R if you don't include an explicit repository. In this case youmight need to use a repository that uses `http` rather than `https`, e.g.,


```r
install.packages('fields', repos = 'http://cran.cnr.berkeley.edu') 
```

If you're on a network and are not the administrator of the machine, you may need to explicitly tell R to install it in a directory you are able to write in:

```r
install.packages('fields', lib = file.path('~', 'R'))
```

If you're using R directly installed on your laptop (i.e., most of you), now (or at the break) would be a good point to install the various packages we need for the bootcamp, which can be done easily with the following command:


```r
install.packages(c('chron','colorspace','codetools', 'DBI','devtools',
                   'dichromat','digest','doParallel', 'dplyr', 'fields',
                   'foreach','ggplot2','gridExtra','gtable','inline',
                   'iterators','knitr','labeling','lattice','lme4',
                   'mapproj','maps','munsell','proftools','proto','purrr',
                   'rbenchmark','RColorBrewer','Rcpp','reshape2','rJava',
                   'RSQLite', 'scales','spam','stringr','tidyr','xlsx',
                   'xlsxjars','xtable'))
```

Note that packages often are dependent on other packages so these dependencies may be installed and loaded automatically. E.g., *fields* depends on *maps* and on *spam*.

You can also install directly from a package zip/tarball rather than from CRAN by giving a filename instead of a package name.

### General information about a package

You can use syntax as follows to get a list of the objects in a package and a brief description: `library(help = packageName)`. 

On CRAN there often *vignettes* that are an overview and describe usage of a package if you click on a specific package. The *reference manual* is just a single document with the help files for all of the objects/functions in a package, so may be helpful but often it's hard to get the big picture view from that.

# More on packages

### The search path

To see the packages that are loaded and the order in which packages are searched for functions/objects: `search()`.

To see what *libraries* (i.e., directory locations) R is retrieving packages from: `.libPaths()`.

And to see where R is getting specific packages, `searchpaths()`.

### Package namespaces

Namespaces are way to keep all the names for objects in a package together in a coherent way and allow R to look for objects in a principled way.

A few useful things to know:


```r
ls('package:stats')[1:20]
```

```
##  [1] "acf"                  "acf2AR"               "add1"                
##  [4] "addmargins"           "add.scope"            "aggregate"           
##  [7] "aggregate.data.frame" "aggregate.ts"         "AIC"                 
## [10] "alias"                "anova"                "ansari.test"         
## [13] "aov"                  "approx"               "approxfun"           
## [16] "ar"                   "ar.burg"              "arima"               
## [19] "arima0"               "arima0.diag"
```

```r
lm <- function(i) {
   print(i)
}
lm(7) 
```

```
## [1] 7
```

```r
lm(gap$lifeExp ~ gap$gdpPercap)
```

```
## gap$lifeExp ~ gap$gdpPercap
## <environment: 0x36cbb00>
```

```r
stats::lm(gap$lifeExp ~ gap$gdpPercap)
```

```
## 
## Call:
## stats::lm(formula = gap$lifeExp ~ gap$gdpPercap)
## 
## Coefficients:
##   (Intercept)  gap$gdpPercap  
##     5.396e+01      7.649e-04
```

```r
rm(lm)
```

Can you explain what is going on? Consider the results of `search()`.


### Looking inside a package

Packages are available as "Package source", namely the raw code and help files, and "binaries", where stuff is packaged up for R to use efficiently. 

To look at the raw R code (and possibly C/C++/Fortran code included in some packages), download and unzip the package source tarball. From the command line of a Linux/Mac terminal (note this won't look right in the slides version of the HTML):


```bash
curl https://cran.r-project.org/src/contrib/fields_9.6.tar.gz \
     -o fields_9.6.tar.gz
tar -xvzf fields_9.6.tar.gz
cd fields
ls R
ls src
ls man
ls data
```

### Creating your own R package

R is do-it-yourself - you can write your own package. At its most basic this is just some R scripts that are packaged together in a convenient format. And if giving it to someone else, it's best to have some documentation in the form of function help files. 

Why make a package?

* It's an easy way to share code with collaborators
* It's a good way to create self-contained code for code you commonly use yourself
* It's how you can share your code and methods with the outside world
* It helps make your work reproducible
* It forces you to be more formal about your coding, which will improve your code

See the *devtools* package and `package.skeleton()` for some useful tools to help you create a package. And there are lots of tips/tutorials online, in particular [Hadley Wickham's R packages book](https://r-pkgs.had.co.nz/).

# The working directory

To read and write from R, you need to have a firm grasp of where in the computer's filesystem you are reading and writing from. 


```r
getwd()  # what directory will R look in?
# Linux/Mac specific
setwd('~/Desktop/r-bootcamp-winter-2019') # change the working directory
setwd('/Users/paciorek/Desktop') # absolute path
getwd()
setwd('r-bootcamp-winter-2019/modules') # relative path
setwd('../tmp') # relative path, up and back down the tree

# Windows - use either \\ or / to indicate directories
# setwd('C:\\Users\\Your_username\\Desktop\\r-bootcamp-winter-2019')
# setwd('..\\r-bootcamp-winter-2019')

# platform-agnostic
setwd(file.path('~', 'Desktop', 'r-bootcamp-winter-2019', 'modules')) # change the working directory
setwd(file.path('/', 'Users', 'paciorek', 'Desktop', 'r-bootcamp-winter-2019', 'modules')) # absolute path
getwd()
setwd(file.path('..', 'data')) # relative path
```
Many errors and much confusion result from you and R not being on the same page in terms of where in the directory structure you are.

# Reading text files into R

The workhorse for reading into a data frame is `read.table()`, which allows any separator (CSV, tab-delimited, etc.). `read.csv()` is a special case of `read.table()` for CSV files.

Here's a simple example where R is able to read the data in using the default arguments to `read.csv()`.


```r
cpds <- read.csv(file.path('..', 'data', 'cpds.csv'))
head(cpds)
```

```
##   year   country vturn outlays realgdpgr unemp
## 1 1960 Australia  95.5      NA        NA  1.42
## 2 1961 Australia  95.3      NA     -0.07  2.79
## 3 1962 Australia  95.3   23.17      5.71  2.63
## 4 1963 Australia  95.7   23.01      6.10  2.12
## 5 1964 Australia  95.7   22.88      6.28  1.15
## 6 1965 Australia  95.7   24.90      4.97  1.15
```

It's good to first look at your data in plain text format outside of R and then to check it after you've read it into R.

# Reading 'foreign' format data

Here's an example of reading data produced by another statistical package (Stata) with `read.dta()`. 


```r
library(foreign)
vote <- read.dta(file.path('..', 'data', '2004_labeled_processed_race.dta'))
head(vote)
```

```
##   state pres04    sex  race  age9 partyid income relign8 age60 age65
## 1     2      1 female white 25-29    <NA>   <NA>    <NA> 18-29 25-29
## 2     2      2   male white 18-24    <NA>   <NA>    <NA> 18-29 18-24
## 3     2      1 female black 30-39    <NA>   <NA>    <NA> 30-44 30-39
## 4     2      1 female black 30-39    <NA>   <NA>    <NA> 30-44 30-39
## 5     2      1 female white 40-44    <NA>   <NA>    <NA> 30-44 40-49
## 6     2      1 female white 30-39    <NA>   <NA>    <NA> 30-44 30-39
##   geocode sizeplac brnagain attend year region y
## 1       3    rural     <NA>   <NA> 2004      4 0
## 2       3    rural     <NA>   <NA> 2004      4 1
## 3       3    rural     <NA>   <NA> 2004      4 0
## 4       3    rural     <NA>   <NA> 2004      4 0
## 5       3    rural     <NA>   <NA> 2004      4 0
## 6       3    rural     <NA>   <NA> 2004      4 0
```

There are a number of other formats that we can handle for either reading or writing. Let's see `library(help = foreign)`.

R can also read in (and write out) Excel files, netCDF files, HDF5 files, etc., in many cases through add-on packages from CRAN. 

A pause for a (gentle) diatribe:

Please try to avoid using Excel files as a data storage format. It's proprietary, complicated (can have multiple sheets), allows a limited number of rows/columns, and files are not easily readable/viewable (unlike simple text files).

# Writing data out from R

Here you have a number of options. 

1) You can write out R objects to an R Data file, as we've seen, using `save()` and `save.image()`.
2) You can use `write.csv()` and `write.table()` to write data frames/matrices to flat text files with delimiters such as comma and tab.
3) You can use `write()` to write out matrices in a simple flat text format.
4) You can use `cat()` to write to a file, while controlling the formatting to a fine degree.
5) You can write out in the various file formats mentioned on the previous slide

### Writing out plots and tables


```r
pdf('myplot.pdf', width = 7, height = 7)
x <- rnorm(10); y <- rnorm(10)
plot(x, y)
dev.off()
```

```
## png 
##   2
```
`xtable()` formats tables for HTML and Latex (the default).


```r
library(xtable)
print(xtable(table(gap$year, gap$continent)), type = "html")
```

```
## <!-- html table generated in R 3.4.3 by xtable 1.8-2 package -->
## <!-- Mon Jan 14 17:24:52 2019 -->
## <table border=1>
## <tr> <th>  </th> <th> Africa </th> <th> Americas </th> <th> Asia </th> <th> Europe </th> <th> Oceania </th>  </tr>
##   <tr> <td align="right"> 1952 </td> <td align="right">  52 </td> <td align="right">  25 </td> <td align="right">  33 </td> <td align="right">  30 </td> <td align="right">   2 </td> </tr>
##   <tr> <td align="right"> 1957 </td> <td align="right">  52 </td> <td align="right">  25 </td> <td align="right">  33 </td> <td align="right">  30 </td> <td align="right">   2 </td> </tr>
##   <tr> <td align="right"> 1962 </td> <td align="right">  52 </td> <td align="right">  25 </td> <td align="right">  33 </td> <td align="right">  30 </td> <td align="right">   2 </td> </tr>
##   <tr> <td align="right"> 1967 </td> <td align="right">  52 </td> <td align="right">  25 </td> <td align="right">  33 </td> <td align="right">  30 </td> <td align="right">   2 </td> </tr>
##   <tr> <td align="right"> 1972 </td> <td align="right">  52 </td> <td align="right">  25 </td> <td align="right">  33 </td> <td align="right">  30 </td> <td align="right">   2 </td> </tr>
##   <tr> <td align="right"> 1977 </td> <td align="right">  52 </td> <td align="right">  25 </td> <td align="right">  33 </td> <td align="right">  30 </td> <td align="right">   2 </td> </tr>
##   <tr> <td align="right"> 1982 </td> <td align="right">  52 </td> <td align="right">  25 </td> <td align="right">  33 </td> <td align="right">  30 </td> <td align="right">   2 </td> </tr>
##   <tr> <td align="right"> 1987 </td> <td align="right">  52 </td> <td align="right">  25 </td> <td align="right">  33 </td> <td align="right">  30 </td> <td align="right">   2 </td> </tr>
##   <tr> <td align="right"> 1992 </td> <td align="right">  52 </td> <td align="right">  25 </td> <td align="right">  33 </td> <td align="right">  30 </td> <td align="right">   2 </td> </tr>
##   <tr> <td align="right"> 1997 </td> <td align="right">  52 </td> <td align="right">  25 </td> <td align="right">  33 </td> <td align="right">  30 </td> <td align="right">   2 </td> </tr>
##   <tr> <td align="right"> 2002 </td> <td align="right">  52 </td> <td align="right">  25 </td> <td align="right">  33 </td> <td align="right">  30 </td> <td align="right">   2 </td> </tr>
##   <tr> <td align="right"> 2007 </td> <td align="right">  52 </td> <td align="right">  25 </td> <td align="right">  33 </td> <td align="right">  30 </td> <td align="right">   2 </td> </tr>
##    </table>
```

Recall our discussion of the `summary()` function used on either a vector or a regression object. What do you think is going on with call to `print()` above? 

# Version control

### Overview

At a basic level, a simple principle is to have version numbers for all your work: code, datasets, manuscripts. Whenever you make a change to a dataset, increment the version number. For code and manuscripts, increment when you make substantial changes or have obvious breakpoints in your workflow. 

However, this is a hassle to do manually. Instead of manually trying to keep track of what changes you've made to code, data, documents, you use software to help you manage the process. This has several benefits:

* easily allowing you to go back to earlier versions
* allowing you to have multiple versions you can switch between
* allowing you to share work easily without worrying about conflicts
* providing built-in backup


### Git and Github

Git is a popular tool for version control. Git is based around the notion of a repository, which is basically a version-controlled project directory. Many people use it with the Github or Bitbucket online hosting services for repositories.

In the introductory material, we've already seen how to get a copy of a Github repository on your local machine. 

As you're gathering by now, I've used Git and Github to manage all the content for this workshop.

### Making changes to a repository

We'll go through a short example of making changes to the r-bootcamp-winter-2019 repository. In this case you don't have permission to make changes so you'll just have to follow along as I do it. However, you could start your own repository and then you'd be able to do similar things.

Note that there are [graphical interfaces to Git](https://git-scm.com/downloads/guis) that you might want to check out, but here I'm just going to do it from the command line on my Mac.

The basic notion we need is a *commit*. As we make changes to our files, we want to commit those changes to the repository regularly. A *commit* is a set of changes recorded with Git. We will often then *push* those changes to a remote copy of the repository, such as on Github.

Here's a basic workflow:

1. Add a file to, or make changes to a file in, a repository
2. Commit the changes
3. Push the changes to the remote version of the repository

Here's how this would look from the command line (this won't look right in the slides version of the HTML):

```bash
git add myfile
# make changes to mycode.R
git commit -am'added myfile and fixed bug in mycode.R'
git push
```

The changes are then available to anyone to *pull* from the remote repository, a using `git pull` or graphical interfaces, such as using RStudio's tools to pull the changes to your machine, discussed in the Github slide in module 0.


# Getting R help online

### Mailing lists

There are several mailing lists that have lots of useful postings. In general if you have an error, others have already posted about it.

- R help: [R mailing lists archive](https://tolstoy.newcastle.edu.au/R/)
- [Stack Overflow](https://stackoverflow.com) (R stuff will be tagged with [R])
- R help special interest groups (SIG) such as *r-sig-hpc* (high performance computing), *r-sig-mac* (R on Macs), etc. (not easily searchable)
- Simple Google searches 
    - You may want to include "in R", with the quotes in the search
    - To search a SIG you might include the name of the SIG in the search string
    - [Rseek.org](https://Rseek.org) for Google searches restricted to sites that have information on R

If you are searching you often want to search for a specific error message. Remember to use double quotes around your error message so it is not broken into individual words by the search engine. 

### Posting your own questions

The main rule of thumb is to do your homework first to make sure the answer is not already available on the mailing list or in other documentation. Some of the folks who respond to mailing list questions are not the friendliest so it helps to have a thick skin, even if you have done your homework. On the plus side, they are very knowledgeable and include the world's foremost R experts/developers.

Here are some guidelines when posting to one of the R mailing lists [https://www.r-project.org/posting-guide.html](https://www.r-project.org/posting-guide.html)

`sessionInfo()` is a function that will give information about your R version, OS, etc., that you can include in your posting.

You also want to include a short, focused, [reproducible](https://adv-r.had.co.nz/Reproducibility.html) example of your problem that others can run. 


# Breakout

### Basics

1) Make sure you are able to install packages from CRAN. E.g., try to install *lmtest*.

2) Figure out what your current working directory is.

### Using the ideas

3) Put the *data/cpds.csv* file in some other directory on your computer, such as *Downloads*. Use `setwd()` to set your working directory to be that directory. Read the file in using `read.csv()`.  Now use `setwd()` to point to a different directory such as *Desktop*. Write the data frame out to a file without any row names and without quotes on the character strings.

4) Make a plot with the gapminder data. Save it as a PDF in *Desktop*. Now see what happens if you set the *width* and *height* arguments to be very small and see how it affects the resulting PDF. Do the same but setting width and height to be very large.

5) Figure out where (what directory) the *graphics* package is stored on your machine. Is it the same as where the *fields* package is stored?

### Advanced

6) Load the *spam* package. Note the message about `backsolve()` being masked from package:base. Now if you enter `backsolve`, you'll see the code associated with the version of `backsolve()` provided by the *spam* package. Now enter `base::backsolve` and you'll see the code for the version of `backsolve()` provided by base R. Explain why typing `backsolve` shows the *spam* version rather than the *base* version. 
