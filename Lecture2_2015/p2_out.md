---
title: Digital Data Collection - Digging Deeper
author: Rolf Fredheim
date: 24/02/2015
output: 
  beamer_presentation:
      theme: "CambridgeUS"
      includes:
        in_header: ../header.tex
---

Logging on
========================================================


Before you sit down:
- Do you have your MCS password?
- Do you have your Raven password?
  - If you answered **'no'** to either then go to the University Computing Services (just outside the door) NOW!
- Are you registered? If not, see me!



Download these slides 
========================================================

Follow link from course description on the SSRMC pages or go directly to 
http://fredheir.github.io/WebScraping/

Download the R file to your computer



Install the following packages:
===============
ggplot2
lubridate
plyr
jsonlite
stringr

No class next week!!
=================


Recap
================

- Basic principles of data collection
- Basics of text manipulation in R
- Simple scraping example

Today we will scrape
================
More JSON!
- social share stats
- comments
- newspaper articles


For that we will need
================
- use paste to make urls
- jsonlite to convert json to lists and data.frames
- loops to iterate over urls
- functions to store code
- rbind, cbind, and c to collect data

This might seem a lot. But very little changes, and it's a powerful toolkit

Load the packages
=================

```r
require(ggplot2)
require(lubridate)
require(plyr)
require(stringr)
require(jsonlite)
```

Last week's example
==================

```r
url  <- "http://stats.grok.se/json/en/201201/web_scraping"
raw.data <- readLines(url, warn="F") 
rd  <- fromJSON(raw.data)
summary(rd)
```

```
            Length Class  Mode     
daily_views 31     -none- list     
project      1     -none- character
month        1     -none- character
rank         1     -none- numeric  
title        1     -none- character
```


Cont
=====

```r
rd.views <- unlist(rd$daily_views )
rd.views
```

```
2012-01-01 2012-01-02 2012-01-03 2012-01-04 2012-01-05 2012-01-06 
       283        573        578        666        673        626 
2012-01-07 2012-01-08 2012-01-09 2012-01-24 2012-01-25 2012-01-22 
       360        430        747        771        758        458 
2012-01-23 2012-01-20 2012-01-21 2012-01-17 2012-01-16 2012-01-15 
       673        739        536        730        669        568 
2012-01-14 2012-01-13 2012-01-12 2012-01-11 2012-01-10 2012-01-29 
       439        742        710        800        716        500 
2012-01-31 2012-01-30 2012-01-19 2012-01-18 2012-01-26 2012-01-27 
       753        838        726        734        739        738 
2012-01-28 
       490 
```

What are the moving parts?
=====================
in url:
- date
- language
- wiki page

in response:
- field name (daily_views)


Sorting a data frame
=================

- Use order()
- This will return ranks:
- These ranks can be applied using square bracket notation


```r
df <- data.frame(rd.views)
df$dates <-rownames(df)
order(rownames(df))
```

```
 [1]  1  2  3  4  5  6  7  8  9 23 22 21 20 19 18 17 16 28 27 14 15 12 13
[24] 10 11 29 30 31 24 26 25
```

```r
ord_df <-df[order(rownames(df)),]
ord_df
```

```
           rd.views      dates
2012-01-01      283 2012-01-01
2012-01-02      573 2012-01-02
2012-01-03      578 2012-01-03
2012-01-04      666 2012-01-04
2012-01-05      673 2012-01-05
2012-01-06      626 2012-01-06
2012-01-07      360 2012-01-07
2012-01-08      430 2012-01-08
2012-01-09      747 2012-01-09
2012-01-10      716 2012-01-10
2012-01-11      800 2012-01-11
2012-01-12      710 2012-01-12
2012-01-13      742 2012-01-13
2012-01-14      439 2012-01-14
2012-01-15      568 2012-01-15
2012-01-16      669 2012-01-16
2012-01-17      730 2012-01-17
2012-01-18      734 2012-01-18
2012-01-19      726 2012-01-19
2012-01-20      739 2012-01-20
2012-01-21      536 2012-01-21
2012-01-22      458 2012-01-22
2012-01-23      673 2012-01-23
2012-01-24      771 2012-01-24
2012-01-25      758 2012-01-25
2012-01-26      739 2012-01-26
2012-01-27      738 2012-01-27
2012-01-28      490 2012-01-28
2012-01-29      500 2012-01-29
2012-01-30      838 2012-01-30
2012-01-31      753 2012-01-31
```


Changing the date
=================


```r
target <- 201401
url <- paste("http://stats.grok.se/json/en/",
          target,"/web_scraping",sep="")

getData <- function(url){
	raw.data <- readLines(url, warn="F") 
	rd  <- fromJSON(raw.data)
	rd.views <- unlist(rd$daily_views )
	df <- data.frame(rd.views)
  #Because row names tend to get lost....
  df$dates <- rownames(df)
	return(df)
}

getData(url)
```

```
           rd.views      dates
2014-01-15      779 2014-01-15
2014-01-14      806 2014-01-14
2014-01-17      827 2014-01-17
2014-01-16      981 2014-01-16
2014-01-11      489 2014-01-11
2014-01-10      782 2014-01-10
2014-01-13      756 2014-01-13
2014-01-12      476 2014-01-12
2014-01-19      507 2014-01-19
2014-01-18      473 2014-01-18
2014-01-28      789 2014-01-28
2014-01-29      799 2014-01-29
2014-01-20      816 2014-01-20
2014-01-21      857 2014-01-21
2014-01-22      899 2014-01-22
2014-01-23      792 2014-01-23
2014-01-24      749 2014-01-24
2014-01-25      508 2014-01-25
2014-01-26      488 2014-01-26
2014-01-27      769 2014-01-27
2014-01-06        0 2014-01-06
2014-01-07      786 2014-01-07
2014-01-04      456 2014-01-04
2014-01-05       77 2014-01-05
2014-01-02      674 2014-01-02
2014-01-03      586 2014-01-03
2014-01-01      348 2014-01-01
2014-01-08      765 2014-01-08
2014-01-09      787 2014-01-09
2014-01-31      874 2014-01-31
2014-01-30     1159 2014-01-30
```

Create urls for January -June
===========================

- ':' operator
- paste()


```r
5:10
```

```
[1]  5  6  7  8  9 10
```

```r
201401:201406
```

```
[1] 201401 201402 201403 201404 201405 201406
```

```r
targets <- 201401:201406
target_urls <- paste("http://stats.grok.se/json/en/",
                  targets,"/web_scraping",sep="")
target_urls
```

```
[1] "http://stats.grok.se/json/en/201401/web_scraping"
[2] "http://stats.grok.se/json/en/201402/web_scraping"
[3] "http://stats.grok.se/json/en/201403/web_scraping"
[4] "http://stats.grok.se/json/en/201404/web_scraping"
[5] "http://stats.grok.se/json/en/201405/web_scraping"
[6] "http://stats.grok.se/json/en/201406/web_scraping"
```


Download them one by one
==========================

```r
for (i in target_urls){
	print (i)
}
```

```
[1] "http://stats.grok.se/json/en/201401/web_scraping"
[1] "http://stats.grok.se/json/en/201402/web_scraping"
[1] "http://stats.grok.se/json/en/201403/web_scraping"
[1] "http://stats.grok.se/json/en/201404/web_scraping"
[1] "http://stats.grok.se/json/en/201405/web_scraping"
[1] "http://stats.grok.se/json/en/201406/web_scraping"
```

```r
for (i in target_urls){
	dat = getData(i)
}
```

Binding data together
=======
3 functions:
- c() 
- rbind()
- cbind()

Simple solution: bind object A together with new object, and overwrite object A. Repeat.

Loops: storing the data?
===========

- Usually we use the variable i
- Why variable? Because we can reuse 'i', even though the value it refers to 'varies'
- Why i? i is for index. But you could use anything you want

```r
hold <- NULL
for (i in 1:5){
  print(paste0('this is loop number ',i))
  hold <- c(hold,i)
  print(hold)
}
```

```
[1] "this is loop number 1"
[1] 1
[1] "this is loop number 2"
[1] 1 2
[1] "this is loop number 3"
[1] 1 2 3
[1] "this is loop number 4"
[1] 1 2 3 4
[1] "this is loop number 5"
[1] 1 2 3 4 5
```


Solution
========

use rbind()
create empty vector, and add data to the end of it:

```r
holder <- NULL
for (i in target_urls){
	dat <- getData(i)
	holder <- rbind(holder,dat)
}

holder
```

```
           rd.views      dates
2014-01-15      779 2014-01-15
2014-01-14      806 2014-01-14
2014-01-17      827 2014-01-17
2014-01-16      981 2014-01-16
2014-01-11      489 2014-01-11
2014-01-10      782 2014-01-10
2014-01-13      756 2014-01-13
2014-01-12      476 2014-01-12
2014-01-19      507 2014-01-19
2014-01-18      473 2014-01-18
2014-01-28      789 2014-01-28
2014-01-29      799 2014-01-29
2014-01-20      816 2014-01-20
2014-01-21      857 2014-01-21
2014-01-22      899 2014-01-22
2014-01-23      792 2014-01-23
2014-01-24      749 2014-01-24
2014-01-25      508 2014-01-25
2014-01-26      488 2014-01-26
2014-01-27      769 2014-01-27
2014-01-06        0 2014-01-06
2014-01-07      786 2014-01-07
2014-01-04      456 2014-01-04
2014-01-05       77 2014-01-05
2014-01-02      674 2014-01-02
2014-01-03      586 2014-01-03
2014-01-01      348 2014-01-01
2014-01-08      765 2014-01-08
2014-01-09      787 2014-01-09
2014-01-31      874 2014-01-31
2014-01-30     1159 2014-01-30
2014-02-23      537 2014-02-23
2014-02-07      831 2014-02-07
2014-02-06      776 2014-02-06
2014-02-05      882 2014-02-05
2014-02-04      750 2014-02-04
2014-02-03      765 2014-02-03
2014-02-02      448 2014-02-02
2014-02-01      507 2014-02-01
2014-02-25      871 2014-02-25
2014-02-24      830 2014-02-24
2014-02-27      834 2014-02-27
2014-02-26      907 2014-02-26
2014-02-21      968 2014-02-21
2014-02-20     1014 2014-02-20
2014-02-09      794 2014-02-09
2014-02-22      578 2014-02-22
2014-02-29        0 2014-02-29
2014-02-28      944 2014-02-28
2014-02-10     1204 2014-02-10
2014-02-11     2970 2014-02-11
2014-02-12      851 2014-02-12
2014-02-13      783 2014-02-13
2014-02-14      740 2014-02-14
2014-02-15      538 2014-02-15
2014-02-16      504 2014-02-16
2014-02-17      748 2014-02-17
2014-02-18      980 2014-02-18
2014-02-19      938 2014-02-19
2014-02-08      520 2014-02-08
2014-02-30        0 2014-02-30
2014-02-31        0 2014-02-31
2014-03-13      866 2014-03-13
2014-03-12      938 2014-03-12
2014-03-11      971 2014-03-11
2014-03-10      929 2014-03-10
2014-03-17      961 2014-03-17
2014-03-16      501 2014-03-16
2014-03-15      521 2014-03-15
2014-03-14      780 2014-03-14
2014-03-31      864 2014-03-31
2014-03-30      512 2014-03-30
2014-03-19      836 2014-03-19
2014-03-18      832 2014-03-18
2014-03-29      520 2014-03-29
2014-03-22      478 2014-03-22
2014-03-23      445 2014-03-23
2014-03-20      905 2014-03-20
2014-03-28      781 2014-03-28
2014-03-21      785 2014-03-21
2014-03-08      557 2014-03-08
2014-03-09      513 2014-03-09
2014-03-04      933 2014-03-04
2014-03-05      896 2014-03-05
2014-03-06      864 2014-03-06
2014-03-07      829 2014-03-07
2014-03-26      849 2014-03-26
2014-03-01      619 2014-03-01
2014-03-02      780 2014-03-02
2014-03-03      940 2014-03-03
2014-03-24      841 2014-03-24
2014-03-25      880 2014-03-25
2014-03-27      838 2014-03-27
2014-04-30      900 2014-04-30
2014-04-31        0 2014-04-31
2014-04-16      816 2014-04-16
2014-04-17      783 2014-04-17
2014-04-14      804 2014-04-14
2014-04-15      818 2014-04-15
2014-04-12      491 2014-04-12
2014-04-13      490 2014-04-13
2014-04-10      814 2014-04-10
2014-04-11      759 2014-04-11
2014-04-18      588 2014-04-18
2014-04-19      486 2014-04-19
2014-04-29      890 2014-04-29
2014-04-28     1002 2014-04-28
2014-04-27      890 2014-04-27
2014-04-26      970 2014-04-26
2014-04-25      921 2014-04-25
2014-04-24      787 2014-04-24
2014-04-23      904 2014-04-23
2014-04-22      763 2014-04-22
2014-04-21      726 2014-04-21
2014-04-20      417 2014-04-20
2014-04-05      492 2014-04-05
2014-04-04      790 2014-04-04
2014-04-07      844 2014-04-07
2014-04-06      502 2014-04-06
2014-04-01      857 2014-04-01
2014-04-03      845 2014-04-03
2014-04-02      848 2014-04-02
2014-04-09      823 2014-04-09
2014-04-08      807 2014-04-08
2014-05-27      911 2014-05-27
2014-05-06      955 2014-05-06
2014-05-21      775 2014-05-21
2014-05-22      766 2014-05-22
2014-05-23      843 2014-05-23
2014-05-19      823 2014-05-19
2014-05-18      494 2014-05-18
2014-05-31      437 2014-05-31
2014-05-30      759 2014-05-30
2014-05-11      447 2014-05-11
2014-05-10      588 2014-05-10
2014-05-13      799 2014-05-13
2014-05-12      865 2014-05-12
2014-05-15      806 2014-05-15
2014-05-14      766 2014-05-14
2014-05-17      432 2014-05-17
2014-05-16      604 2014-05-16
2014-05-08      938 2014-05-08
2014-05-09      769 2014-05-09
2014-05-28      821 2014-05-28
2014-05-29      854 2014-05-29
2014-05-02      807 2014-05-02
2014-05-03      523 2014-05-03
2014-05-26      704 2014-05-26
2014-05-01      793 2014-05-01
2014-05-20      787 2014-05-20
2014-05-07      874 2014-05-07
2014-05-04      498 2014-05-04
2014-05-05      738 2014-05-05
2014-05-24      586 2014-05-24
2014-05-25      541 2014-05-25
2014-06-14      494 2014-06-14
2014-06-15      496 2014-06-15
2014-06-16      716 2014-06-16
2014-06-17      924 2014-06-17
2014-06-10      697 2014-06-10
2014-06-11      789 2014-06-11
2014-06-12      865 2014-06-12
2014-06-13      867 2014-06-13
2014-06-18     1040 2014-06-18
2014-06-19     1058 2014-06-19
2014-06-31        0 2014-06-31
2014-06-09      658 2014-06-09
2014-06-08      448 2014-06-08
2014-06-03      745 2014-06-03
2014-06-02      697 2014-06-02
2014-06-01      475 2014-06-01
2014-06-07      411 2014-06-07
2014-06-06      775 2014-06-06
2014-06-05      790 2014-06-05
2014-06-04      873 2014-06-04
2014-06-21      538 2014-06-21
2014-06-20      978 2014-06-20
2014-06-23      777 2014-06-23
2014-06-22      488 2014-06-22
2014-06-25      905 2014-06-25
2014-06-24      852 2014-06-24
2014-06-27      727 2014-06-27
2014-06-26      748 2014-06-26
2014-06-29      414 2014-06-29
2014-06-28      439 2014-06-28
2014-06-30      694 2014-06-30
```

Is this efficient?
=========
Why (not)?

Parsimonious approach
========

Create a matrix and assign using square bracket notation - if you know the number of rows

You could also use ldply here:
> For each element of a list, apply function then combine results into a data frame.

Does the same thing as lapply + do.call(rbind)
- lapply: 'applies' a function to each item in a vector. Returns a list. 
- do.call: executes a function to each part of an item (here: the list)

Apply family: apply(), sapply(), lapply()
extensions from Hadley Wickham's plyr: ddply(), ldply() the most useful

```r
dat <- ldply(target_urls,getData)
```

Putting it together
==================


```r
targets <- 201401:201406
targets <- paste("http://stats.grok.se/json/en/",
              201401:201406,"/web_scraping",sep="")
dat <- ldply(targets,getData)
```

Task
========

Edit the code to download data for a different range of dates

Edit the second line to download a vector of pages, rather than dates:


```r
targets <- c("Barack_Obama","United_States_elections,_2014")
```


Walkthrough
=========

```r
targets <- c("Barack_Obama","United_States_elections,_2014")
target_urls <- paste("http://stats.grok.se/json/en/201401/",targets,sep="")
results <- ldply(target_urls,getData)

#find number of rows for each: 
t <- nrow(results)/length(targets)
t
```

```
[1] 31
```

```r
#apply ids:
results$id <- rep(targets,each=t)
```

Moving on
========
Comments to newspaper articles

http://www.dailymail.co.uk/news/article-2643770/Why-Americans-suckers-conspiracy-theories-The-country-founded-says-British-academic.html

http://www.dailymail.co.uk/reader-comments/p/asset/readcomments/2643770?max=10&order=desc

Why can't we use our getData function?


Download the page
=======


```r
url <- 'http://www.dailymail.co.uk/reader-comments/p/asset/readcomments/2643770?max=10&order=desc'
raw.data <- readLines(url, warn="F") 
rd  <- fromJSON(raw.data)

str(rd)
```

```
List of 3
 $ status : chr "success"
 $ code   : chr "200"
 $ payload:List of 9
  ..$ total                : int 373
  ..$ parentCommentsCount  : int 166
  ..$ offset               : chr "0"
  ..$ max                  : int 10
  ..$ page                 :'data.frame':	10 obs. of  14 variables:
  .. ..$ id                  : int [1:10] 55921963 55864818 55863015 55860458 55859344 55856766 55851783 55850891 55849779 55848206
  .. ..$ dateCreated         : chr [1:10] "2014-06-01T16:41:36.960Z" "2014-05-31T17:45:45.763Z" "2014-05-31T17:10:40.931Z" "2014-05-31T16:21:58.323Z" ...
  .. ..$ message             : chr [1:10] "Step 1: Throw a bunch of \"conspiracy theories\" together, carefully lumping in absurd stories about aliens and fake moon landi"| __truncated__ "Remember Hillary Clinton's \"vast right wing consipiracy\" -- and she's a favorite for president among half of the country.  Wh"| __truncated__ "Better to ask why they abound!! because Governmental malfeasance is rampant!" "If the mainstream media did its job, many conspiracy theories could be put to rest.  But the media serve as shields for the gov"| __truncated__ ...
  .. ..$ assetId             : int [1:10] 2643770 2643770 2643770 2643770 2643770 2643770 2643770 2643770 2643770 2643770
  .. ..$ assetUrl            : chr [1:10] "/news/article-2643770/Why-Americans-suckers-conspiracy-theories-The-country-founded-says-British-academic.html" "/news/article-2643770/Why-Americans-suckers-conspiracy-theories-The-country-founded-says-British-academic.html" "/news/article-2643770/Why-Americans-suckers-conspiracy-theories-The-country-founded-says-British-academic.html" "/news/article-2643770/Why-Americans-suckers-conspiracy-theories-The-country-founded-says-British-academic.html" ...
  .. ..$ assetHeadline       : chr [1:10] "America 'founded on conspiracy theories', says British academic" "America 'founded on conspiracy theories', says British academic" "America 'founded on conspiracy theories', says British academic" "America 'founded on conspiracy theories', says British academic" ...
  .. ..$ assetCommentCount   : int [1:10] 375 375 375 375 375 375 375 375 375 375
  .. ..$ userAlias           : chr [1:10] "Wasabista" "Brian" "david" "Cloudy" ...
  .. ..$ userLocation        : chr [1:10] "Saitama, Japan" "Denver, United States" "St helier" "San Francisco" ...
  .. ..$ userIdentifier      : chr [1:10] "4064792" "1392341264378632" "4361368" "4419281" ...
  .. ..$ voteRating          : int [1:10] 3 4 11 11 17 3 19 -4 13 -6
  .. ..$ voteCount           : int [1:10] 5 14 15 19 19 3 45 16 15 24
  .. ..$ replies             :'data.frame':	10 obs. of  2 variables:
  .. .. ..$ totalCount: int [1:10] 0 0 0 0 0 0 0 1 0 2
  .. .. ..$ comments  :List of 10
  .. .. .. ..$ :'data.frame':	0 obs. of  0 variables
  .. .. .. ..$ :'data.frame':	0 obs. of  0 variables
  .. .. .. ..$ :'data.frame':	0 obs. of  0 variables
  .. .. .. ..$ :'data.frame':	0 obs. of  0 variables
  .. .. .. ..$ :'data.frame':	0 obs. of  0 variables
  .. .. .. ..$ :'data.frame':	0 obs. of  0 variables
  .. .. .. ..$ :'data.frame':	0 obs. of  0 variables
  .. .. .. ..$ :'data.frame':	1 obs. of  14 variables:
  .. .. .. .. ..$ id                  : int 55857464
  .. .. .. .. ..$ dateCreated         : chr "2014-05-31T15:22:19.139Z"
  .. .. .. .. ..$ message             : chr "No these are our politicians. "
  .. .. .. .. ..$ assetId             : int 2643770
  .. .. .. .. ..$ assetUrl            : chr "/news/article-2643770/Why-Americans-suckers-conspiracy-theories-The-country-founded-says-British-academic.html"
  .. .. .. .. ..$ assetHeadline       : chr "America 'founded on conspiracy theories', says British academic"
  .. .. .. .. ..$ assetCommentCount   : int 375
  .. .. .. .. ..$ userAlias           : chr "Just call me queen."
  .. .. .. .. ..$ userLocation        : chr "Over the hill and far away, Monaco"
  .. .. .. .. ..$ userIdentifier      : chr "4752815"
  .. .. .. .. ..$ voteRating          : int 10
  .. .. .. .. ..$ voteCount           : int 12
  .. .. .. .. ..$ replies             :'data.frame':	1 obs. of  2 variables:
  .. .. .. .. .. ..$ totalCount: int 0
  .. .. .. .. .. ..$ comments  :List of 1
  .. .. .. .. .. .. ..$ : list()
  .. .. .. .. ..$ formattedDateAndTime: chr "8 months ago"
  .. .. .. ..$ :'data.frame':	0 obs. of  0 variables
  .. .. .. ..$ :'data.frame':	2 obs. of  14 variables:
  .. .. .. .. ..$ id                  : int [1:2] 55851451 55857057
  .. .. .. .. ..$ dateCreated         : chr [1:2] "2014-05-31T13:27:53.306Z" "2014-05-31T15:14:11.612Z"
  .. .. .. .. ..$ message             : chr [1:2] "I believe them all and the more outlandish, the better." "@os, isn't that the definition of people ho read the tabloids for their news?  "
  .. .. .. .. ..$ assetId             : int [1:2] 2643770 2643770
  .. .. .. .. ..$ assetUrl            : chr [1:2] "/news/article-2643770/Why-Americans-suckers-conspiracy-theories-The-country-founded-says-British-academic.html" "/news/article-2643770/Why-Americans-suckers-conspiracy-theories-The-country-founded-says-British-academic.html"
  .. .. .. .. ..$ assetHeadline       : chr [1:2] "America 'founded on conspiracy theories', says British academic" "America 'founded on conspiracy theories', says British academic"
  .. .. .. .. ..$ assetCommentCount   : int [1:2] 375 375
  .. .. .. .. ..$ userAlias           : chr [1:2] "Bill" "Mortie"
  .. .. .. .. ..$ userLocation        : chr [1:2] "Wasp in the South, United Kingdom" "Birmingham, United States"
  .. .. .. .. ..$ userIdentifier      : chr [1:2] "132686" "1392234183908892"
  .. .. .. .. ..$ voteRating          : int [1:2] 2 0
  .. .. .. .. ..$ voteCount           : int [1:2] 4 8
  .. .. .. .. ..$ replies             :'data.frame':	2 obs. of  2 variables:
  .. .. .. .. .. ..$ totalCount: int [1:2] 0 0
  .. .. .. .. .. ..$ comments  :List of 2
  .. .. .. .. .. .. ..$ : list()
  .. .. .. .. .. .. ..$ : list()
  .. .. .. .. ..$ formattedDateAndTime: chr [1:2] "8 months ago" "8 months ago"
  .. ..$ formattedDateAndTime: chr [1:10] "8 months ago" "8 months ago" "8 months ago" "8 months ago" ...
  ..$ assetId              : chr "2643770"
  ..$ assetStatusId        : int 2
  ..$ isOldArticle         : logi TRUE
  ..$ shoutDisabledChannels: chr "{ \"disabledMolShout\": false, \"disabledChannels\": [] }"
```

Digging in
========

find the list called 'payload'

here we get stats about the number of comments
rd$payload$total

Dig furter into 
'page'
Here's a reasonably well formatted dataframe. We'll get rid of replies, though:


```r
dat <- rd$payload$page
dat$replies <- NULL
head(dat)
```

```
        id              dateCreated
1 55921963 2014-06-01T16:41:36.960Z
2 55864818 2014-05-31T17:45:45.763Z
3 55863015 2014-05-31T17:10:40.931Z
4 55860458 2014-05-31T16:21:58.323Z
5 55859344 2014-05-31T15:59:16.771Z
6 55856766 2014-05-31T15:07:57.775Z
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       message
1                                                                                                                                                                                                                                                                                                                                                                                                                                                                              Step 1: Throw a bunch of "conspiracy theories" together, carefully lumping in absurd stories about aliens and fake moon landings with issues that deserve serious consideration, like the Kennedy assassination and 9/11.\n\nStep 2: Enjoy the satisfaction of having smeared a growing section of the public that doesn't trust the politicians' lies.\n\nStep 3: Invoice enclosed. Cha-ching!
2                                                                                       Remember Hillary Clinton's "vast right wing consipiracy" -- and she's a favorite for president among half of the country.  When you have public Senate hearings confirming CIA projects like MKULTRA  and Project Mockingbird, and with the CIA admitting to Congress that it destroyed most of the related documents, and you have the current NSA scandal, and people like Obama in charge who can't tell the truth if it could save him from Hell, and my personal eyewitness account of the media distorting truth or sometimes outright lying to fit their "story", yeah....I don't believe the vast most of the conspiracy theory loons, but I also don't believe a large portion of what our government and media says, and their non-collective "agenda" is often transparent.
3                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 Better to ask why they abound!! because Governmental malfeasance is rampant!
4                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            If the mainstream media did its job, many conspiracy theories could be put to rest.  But the media serve as shields for the government, instead of investigating, and then act very surprised when the truth comes out years later.  Why did the BBC announce on 9/11 that Tower 7 had collapsed half an hour BEFORE it actually did so?  Whoops!
5 I always thought my grandparents were just conspiracy theory loons when they talked about the CIA dosing citizens with LSD or drugging enlisted men without their knowledge...or that our own government was secretly listening in on our private phone calls or had a vast spying network in place; it seemed too crazy to be true! I guess they showed me; MKULTRA definitely DID happen and as we all know now, the NSA absolutely was/is spying on us. How can we ever be sure of what's true and what's not if we keep finding out, years after the fact, that all these people that were denounced as "crazy" or "out there" by the government and authorities were the only ones who actually were aware of the truth? And that most of us dismissed them as well, believing in our government(s) because the alternative just seemed too far-fetched to be possible?
6                                                                                                                                                                                                                                                                                                                                                               He's got a muddled definition of "conspiracy theory." What differentiates 9/11 and Apollo conspiracy theories from possible real conspiracies is the utter perfection of the imaginary conspiracies and their outlandishly complicated procedures to do things a rational person could do more simply. Imaginary conspiracies are Rube Goldberg operations that nevertheless operate without a hitch. Real conspiracies leak like sieves. They always recruit people who blab in bars trying to impress women.
  assetId
1 2643770
2 2643770
3 2643770
4 2643770
5 2643770
6 2643770
                                                                                                        assetUrl
1 /news/article-2643770/Why-Americans-suckers-conspiracy-theories-The-country-founded-says-British-academic.html
2 /news/article-2643770/Why-Americans-suckers-conspiracy-theories-The-country-founded-says-British-academic.html
3 /news/article-2643770/Why-Americans-suckers-conspiracy-theories-The-country-founded-says-British-academic.html
4 /news/article-2643770/Why-Americans-suckers-conspiracy-theories-The-country-founded-says-British-academic.html
5 /news/article-2643770/Why-Americans-suckers-conspiracy-theories-The-country-founded-says-British-academic.html
6 /news/article-2643770/Why-Americans-suckers-conspiracy-theories-The-country-founded-says-British-academic.html
                                                    assetHeadline
1 America 'founded on conspiracy theories', says British academic
2 America 'founded on conspiracy theories', says British academic
3 America 'founded on conspiracy theories', says British academic
4 America 'founded on conspiracy theories', says British academic
5 America 'founded on conspiracy theories', says British academic
6 America 'founded on conspiracy theories', says British academic
  assetCommentCount userAlias              userLocation   userIdentifier
1               375 Wasabista            Saitama, Japan          4064792
2               375     Brian     Denver, United States 1392341264378632
3               375     david                 St helier          4361368
4               375    Cloudy             San Francisco          4419281
5               375 Way Lucas Pittsburgh, United States 1379523501240711
6               375   Steve D          Green Bay WI USA          4113189
  voteRating voteCount formattedDateAndTime
1          3         5         8 months ago
2          4        14         8 months ago
3         11        15         8 months ago
4         11        19         8 months ago
5         17        19         8 months ago
6          3         3         8 months ago
```

Movable parts
===========

url <- 'http://www.dailymail.co.uk/reader-comments/p/asset/readcomments/2643770?max=10&order=desc'

- id <- 2643770
- max <- 10
- order <- desc

Try repeating the process to download 100 comments


APIs
================


> When used in the context of web development, an API is typically defined as a set of Hypertext Transfer Protocol (HTTP) request messages, along with a definition of the structure of response messages, which is usually in an Extensible Markup Language (XML) or JavaScript Object Notation (JSON) format.

> The practice of publishing APIs has allowed web communities to create an open architecture for sharing content and data between communities and applications. In this way, content that is created in one place can be dynamically posted and updated in multiple locations on the web

-Wikipedia


Social shares
===============
Most newssites will give stats about social shares. 
E.g: http://www.bbc.co.uk/sport/0/football/31583092

-> as of writing this up it had been shared 92 times

Uses Twitter and Facebook apis
Work with a 'get' request


Types of request
==============
> GET requests a representation of the specified resource. Note that GET should not be used for operations that cause side-effects, such as using it for taking actions in web applications. One reason for this is that GET may be used arbitrarily by robots or crawlers, which should not need to consider the side effects that a request should cause.

> POST submits data to be processed (e.g., from an HTML form) to the identified resource. The data is included in the body of the request. This may result in the creation of a new resource or the updates of existing resources or both.

we use 'get' for scraping, 
'post' is more complicated. Use it to navigate logins, popups, etc. 


Constructing a query
======

You might have seen urls with these signs in them:
- ?
- &
The question mark indicates the start of a query, while & is used to separate fields. 

Get requests to social shares are very simple:

http://graph.facebook.com/?id=http://www.bbc.co.uk/sport/0/football/31583092

http://urls.api.twitter.com/1/urls/count.json?url=http://www.bbc.co.uk/sport/0/football/31583092


Download these into R!
=========

Facebook

```r
url <- 'http://graph.facebook.com/?id=http://www.bbc.co.uk/sport/0/football/31583092'
raw.data <- readLines(url, warn="F") 
rd  <- fromJSON(raw.data)
df <- data.frame(rd)
```

Repeat for Twitter


Task
=====

1) Download the number of Twitter shares for each of these these pages:

- http://www.huffingtonpost.com/2015/02/22/wisconsin-right-to-work_n_6731064.html
- http://www.dailymail.co.uk/news/article-2643770/Why-Americans-suckers-conspiracy-theories-The-country-founded-says-British-academic.html

2) Use rbind to combine these responses into a single data.frame

3) write a function that takes an input url to scrape Twitter

4) Write a function that takes an input url to scrape both Twitter and Facebook (hard!)

5) use ldply to make a scraper (copy code from slide 14 - parsimonious approach - above)


Walkthrough
=========

```r
#1) 
url <- 'http://www.dailymail.co.uk/news/article-2643770/Why-Americans-suckers-conspiracy-theories-The-country-founded-says-British-academic.html'
target <- paste('http://urls.api.twitter.com/1/urls/count.json?url=',url,sep="")
raw.data <- readLines(target, warn="F") 
rd  <- fromJSON(raw.data)
tw1 <- data.frame(rd)

url2 <- 'http://www.huffingtonpost.com/2015/02/22/wisconsin-right-to-work_n_6731064.html'
target <- paste('http://urls.api.twitter.com/1/urls/count.json?url=',url2,sep="")
raw.data <- readLines(target, warn="F") 
rd  <- fromJSON(raw.data)
tw2 <- data.frame(rd)
```

Walkthrough 2 and 3
=========


```r
#2)
df <- rbind(tw1,tw2)

#3)
getTweetCount <-function(url){
	target <- paste('http://urls.api.twitter.com/1/urls/count.json?url=',url,sep="")
	raw.data <- readLines(target, warn="F") 
	rd  <- fromJSON(raw.data)
	tw1 <- data.frame(rd)
	return(tw1)
}
getTweetCount(url2)
```

```
  count
1   250
                                                                               url
1 http://www.huffingtonpost.com/2015/02/22/wisconsin-right-to-work_n_6731064.html/
```

Walkthrough 4
==========

```r
#4)
getBoth <-function(url){
	target <- paste('http://urls.api.twitter.com/1/urls/count.json?url=',url,sep="")
	raw.data <- readLines(target, warn="F") 
	rd  <- fromJSON(raw.data)
	tw1 <- data.frame(rd)

	target <- paste('http://graph.facebook.com/?id=',url,sep='')
	raw.data <- readLines(target, warn="F") 
	rd  <- fromJSON(raw.data)
	fb1 <- data.frame(rd)
  
	df <- cbind(fb1[,1:2],tw1$count)
	colnames(df) <- c('id','fb_shares','tw_shares')
	return(df)
}
```

Walkthrough 5
===========


```r
#5)
targets <- c(
'http://www.dailymail.co.uk/news/article-2643770/Why-Americans-suckers-conspiracy-theories-The-country-founded-says-British-academic.html',
'http://www.huffingtonpost.com/2015/02/22/wisconsin-right-to-work_n_6731064.html'
)

dat <- ldply(targets,getBoth)
```

Comments
=========================
It's almost the same thing

```r
url <- 'http://www.huffingtonpost.com/2015/02/22/wisconsin-right-to-work_n_6731064.html'
api <- 'http://graph.facebook.com/comments?id='
target <- paste(api,url,sep="")
raw.data <- readLines(target, warn="F") 
rd  <- fromJSON(raw.data)
head(rd$data)
```

Getting article content
===================
In TWO WEEKS TIME (10 March) we will do scraping proper. But check out this awesome API:
http://juicer.herokuapp.com/

http://juicer.herokuapp.com/api/article?url=http://www.huffingtonpost.com/2015/02/22/wisconsin-right-to-work_n_6731064.html


Download the page
==========

```r
url <- 'http://www.huffingtonpost.com/2015/02/22/wisconsin-right-to-work_n_6731064.html'
api <- 'http://juicer.herokuapp.com/api/article?url='

target <- paste(api,url,sep="")
target

raw.data <- readLines(target, warn="F") 
rd  <- fromJSON(raw.data)

dat <- rd$article
dat$entities <-NULL

dat <-data.frame(dat)
dat
```

=====

```r
ent <- rd$article$entities
ent
```

What does the last frame give us?
========
Named entity recognition!


```r
#use square bracket notation to navigate these data:
ent[ent$type=='Location',]
ent[ent$type=='Person',]
```


Summing up
========
given URLs of target pages, we can now:
- download raw JSON data
- extract fields of interest
- put this in a function
- apply the function to a list of targets


No class next week!!
=================
So that's it for APIs and JSON
But for those who are keen, more advanced stuff involving APIs and JSON sources (maps? YouTube?) can be found in last year's slides:

- http://fredheir.github.io/WebScraping/Lecture4/p4.html

Too many loops, variables, and functions?
============

If this has all been a bit much, below is a link to some extra material on all things variables, functions and loops


- http://fredheir.github.io/WebScraping/Lecture2_2015/extra.html
- http://fredheir.github.io/WebScraping/Lecture2_2015/extra.R

