---
title: Digital Data Collection - The Hard Way
author: Rolf Fredheim
date: 10/03/2015
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
- XML
- RCurl
- lubridate
- plyr
- stringr



Recap
================
**Week1**
- Basic principles of data collection
- Basics of text manipulation in R
- Simple scraping example

**Week2**
- Utility functions
- JSON and APIs

What's the problem with APIs?
===================
- Overkill
- Rate limiting
- Authentication
- Web content may be richer
- Deprecation
  - https://developers.google.com/youtube/2.0/developers_guide_protocol_deprecated
  - https://blog.twitter.com/2013/api-v1-is-retired

Today we will
================
Work with html to
- extract tables
- extract links
- extract information using class and id tags
- write a two-stage scraper to download newspaper articles

For that we will need
================
To use Google Chrome of Mozilla Firefox. 

No Internet Explorer or Safari please!



Load the packages
=================

```r
require(lubridate)
require(plyr)
require(stringr)
require(XML)
require(RCurl)
```


Getting to know HTML structure
==============================


- http://en.wikipedia.org/wiki/Euromaidan
- http://en.wikipedia.org/wiki/Boris_Nemtsov

Let's look at this webpage

- Headings
- Images
- links
- references
- tables

To look at the code (in Google Chrome), right-click somewhere on the page and select 'inspect element'

Tree-structure (parents, siblings)

Back to Wikipedia
====================
HTML tags.

They come in pairs and are surrounded by these guys:
<>

e.g. a heading might look like this:

\<h1\>MY HEADING\</h1\>
<h1>MY HEADING</h1>

Which others do you know or can you find?

Extracting a table
==============

can be a bit fiddly to format, but quite accessible:

```r
url='http://en.wikipedia.org/wiki/Elections_in_Russia'
tables<-readHTMLTable(url)
head(tables[[6]])
```

```
            Candidates       Nominating parties      Votes     %
1       Vladimir Putin            United Russia 45,513,001 63.64
2     Gennady Zyuganov          Communist Party 12,288,624 17.18
3    Mikhail Prokhorov           self-nominated  5,680,558  7.94
4 Vladimir Zhirinovsky Liberal Democratic Party  4,448,959  6.22
5       Sergey Mironov            A Just Russia  2,755,642  3.85
6          Valid votes               70,686,784      98.84  <NA>
```


Download html
=====================

Dont print a webpage like this to the console!

Download using readLines()
or getURL() (RCurl)

Rstudio will probably crash

use head(), str(), summary(), tail() etc. to inspect data.



```r
url  <- "http://en.wikipedia.org/wiki/Boris_Nemtsov"
raw <-  getURL(url,encoding="UTF-8") #Download the page
#this is a very very long line. Let's not print it. Instead:
substring (raw,1,200)
```

```
[1] "<!DOC
```

```r
PARSED <- htmlParse(raw) #Format the html code d
```




Accessing HTML elements in R with XPath
========


Is a language for querying XML

Reading and examples: 
- http://www.w3schools.com/xpath/xpath_intro.asp
- http://www.w3schools.com/xml/xml_xpath.asp


we can use XPath expressions to extract elements from HTML

The element in quotes below is an *XPath expression* that selects the indicated node


```r
xpathSApply(PARSED, "//h1")
```

```
[[1]]
<h1 id="firstHeading" class="firstHeading" lang="en">Boris Nemtsov</h1> 
```

Extract content
======
Not so pretty. But! Specifying xmlValue strips away the surrounding code and returns only the content of the tag

```r
xpathSApply(PARSED, "//h1",xmlValue)
```

```
[1] "Boris Nemtsov"
```





Fundamental XPath Syntax
===============

- /      Select from the root
- //     Select anywhere in document
- @      Select attributes. Use in square brackets


```r
xpathSApply(PARSED, "//h1",xmlValue)
```

```
[1] "Boris Nemtsov"
```




HTML tags
======================

- \<html>: starts html code
- \<head> : contains meta data etc
- \<script> : e.g. javascript to be loaded
- \<style> : css code
- \<meta> : denotes document properties, e.g. author, keywords
- \<title> : 
- \<body> : 

HTML tags2
======================

- \<div>, \<span> :these are used to break up a document into sections and boxes
- \<h1>,\<h2>,\<h3>,\<h4>,\<h5> Different levels of heading
- \<p> : paragraph
- \<br> : line break
- and others: \<a>, \<ul>, \<tbody>, \<th>, \<td>, \<ul>, \<ul>, <img>



What about other headings?
=====================




```r
xpathSApply(PARSED, "//h3",xmlValue)
```

```
 [1] "§Nemtsov's fears[edit]"                     
 [2] "§The assassination (27 February 2015)[edit]"
 [3] "§Aftermath, context and accusations[edit]"  
 [4] "§Reactions[edit]"                           
 [5] "Personal tools"                             
 [6] "Namespaces"                                 
 [7] "Variants"                                   
 [8] "Views"                                      
 [9] "More"                                       
[10] "\n\t\t\t\t\t\t\tSearch\n\t\t\t\t\t\t"       
[11] "Navigation"                                 
[12] "Interaction"                                
[13] "Tools"                                      
[14] "Print/export"                               
[15] "Languages"                                  
```

Line ten... 'search'. Remember the clever  str_trim() function?


```r
temp=xpathSApply(PARSED, "//h3",xmlValue)

str_trim(temp[10])
```

```
[1] "Search"
```



Extracting links
=========
and links

```r
length(xpathSApply(PARSED, "//a/@href"))
```

```
[1] 800
```
That's hundreds of links!!! That's hopeless. We want to be more selective. Back to the drawing board



CSS and XPath
===============

web-designers use Cascading Style Sheets to determine the way a webpage looks

Like variables: change the style, rather than the every item on a page

I use CSS for these slides, check out the code for this page

<strong>CSS allows us to make better selections, by latching onto tags</strong>

**Xpath allows us to move up and down the html tree structure**

CSS can be an html **attribute**


Principles of scraping
=============
- Identify the tag
- Download the web-page
- Extract content matching the tag
- Save the content
- Optional: repeat


Get references
=======================


Content of the references

```r
head(xpathSApply(PARSED, "//span[@class='reference-text']",xmlValue))
```

```
[3] "Birnbaum, Michael; Branigan, William (28 February 2015). \"Putin critic, Russian opposition leader Boris Nemtsov killed in Moscow\". Washington Post. Retrieved 1 March 2015. "
[4] "http://www.foxnews.com/world/2015/03/04/crossing-kremlin-nemtsov-latest-in-long-line-putin-critics-to-wind-up-dead/"                                                           
[5] "Amos, Howard; Millward, David (27 February 2015). \"Leading Putin critic gunned down outside Kremlin\". The Telegraph (London). "                                              
[6] "Kramer, Andew E., \"Boris Nemtsov, Putin Foe, Is Shot Dead in Shadow of Kremlin,\" New York Times, 27 February 2015."                                                          
```
***
URLS

```r
head(as.character(xpathSApply(PARSED, "//span[@class=
  'reference-text']/span/a/@href")))
```

```
[1] "http://www.livelib.ru/author/208578"                                                                                                                                    
[2] "http://www.kommersant.ru/doc/2678842"                                                                                                                                   
[3] "/wiki/Kommersant"                                                                                                                                                       
[4] "http://www.washingtonpost.com/world/europe/russian-opposition-leader-boris-nemtsov-reported-killed-in-moscow/2015/02/27/972e15f0-becb-11e4-b274-e5209a3bc9a9_story.html"
[5] "http://www.telegraph.co.uk/news/worldnews/europe/russia/11441466/Veteran-Russian-opposition-politician-shot-dead-in-Moscow.html"                                        
[6] "http://www.rosbalt.ru/moscow/2015/02/28/1372887.html"                                                                                                                   
```

Sanity test
==========
Test that these work, using browseURL()


```r
links <- (xpathSApply(PARSED, "//span[@class='reference-text']/span/a/@href"))
browseURL(links[1])
```

So if you wanted to, you could scrape these links in turn.

tree-structure is navigated a bit like that on your computer (c:/windows/system)

How it works
======
In this example, we select all elements of 'span'
...Which have an **attribute** "class" of the value "citation news"
...then we select all links
...and return all attributes labeled "href" (the urls)



```r
head(xpathSApply(PARSED, "//span[@class='reference-text']/span/a/@href"))
```


XPath2
============


Like in R, we use square brackets to make selections

What does this select?

```r
head(xpathSApply(PARSED, "//span[@class='reference-text'][17]/span/a/@href"))
```

```
NULL
```



Wildcards
======================


We can also use wildcards:

- \* selects any node or tag
- @* selects any attribute (used to define nodes)

```r
(xpathSApply(PARSED, "//*[@class='citation news'][17]/a/@href"))
```

```
NULL
```

```r
(xpathSApply(PARSED, "//span[@class='citation news'][17]/a/@*"))
```

```
NULL
```

Scrape Telegraph
=======================



http://www.telegraph.co.uk/search/?queryText=nemtsov&sort=relevant

Note: this is a get request (see the ? and & characters?)


```r
url <- 'http://www.telegraph.co.uk/search/?queryText=nemtsov&sort=relevant'
raw <-  getURL(url)#,encoding="UTF-8") 
PARSED <- htmlParse(raw) #Format the html code d
links<-xpathSApply(PARSED, "//a/@href")
length(links)
```

```
[1] 211
```

```r
links<-xpathSApply(PARSED, "//div[@class='searchresults']//a/@href")
length(links)
```

```
[1] 73
```

```r
length(unique(links))
```

```
[1] 37
```

```r
links<-unique(links)

links<-links[grep('http',links)]
```

Daily Mail
======


```r
url <- 'http://www.dailymail.co.uk/home/search.html?sel=site&searchPhrase=nemtsov'
raw <-  getURL(url)#,encoding="UTF-8") 
PARSED <- htmlParse(raw) #Format the html code d
links<-xpathSApply(PARSED, "//a/@href")
length(links)
```

```
[1] 284
```

```r
links<-xpathSApply(PARSED, "//div[@class='sch-results']//a/@href")
length(links)
```

```
[1] 210
```

```r
length(unique(links))
```

```
[1] 51
```

```r
links<-unique(links)
```

dt2
=============


```r
head(links)
```

```
[1] "#"                                                                                                                                                   
[2] "/news/article-2987259/Litvinenko-s-killer-honoured-Putin-Russian-president-gives-medal-services-motherland-man-accused-killing-dissident-London.html"
[3] "/wires/reuters/article-2986593/Russias-Putin-honours-suspect-Litvinenko-poisoning.html"                                                              
[4] "/wires/ap/article-2986551/Putin-honors-Chechen-leader-praised-Nemtsov-suspect.html"                                                                  
[5] "/wires/reuters/article-2986264/Friend-says-Islamist-motive-killing-Nemtsov-nonsense.html"                                                            
[6] "/wires/ap/article-2986228/Putin-bestows-awards-Chechen-leader-Litvinenko-suspect.html"                                                               
```

```r
paste('http://www.dailymail.co.uk',links,sep='')
```

```
 [1] "http://www.dailymail.co.uk#"                                                                                                                                                   
 [2] "http://www.dailymail.co.uk/news/article-2987259/Litvinenko-s-killer-honoured-Putin-Russian-president-gives-medal-services-motherland-man-accused-killing-dissident-London.html"
 [3] "http://www.dailymail.co.uk/wires/reuters/article-2986593/Russias-Putin-honours-suspect-Litvinenko-poisoning.html"                                                              
 [4] "http://www.dailymail.co.uk/wires/ap/article-2986551/Putin-honors-Chechen-leader-praised-Nemtsov-suspect.html"                                                                  
 [5] "http://www.dailymail.co.uk/wires/reuters/article-2986264/Friend-says-Islamist-motive-killing-Nemtsov-nonsense.html"                                                            
 [6] "http://www.dailymail.co.uk/wires/ap/article-2986228/Putin-bestows-awards-Chechen-leader-Litvinenko-suspect.html"                                                               
 [7] "http://www.dailymail.co.uk/wires/reuters/article-2986027/Nemtsov-friend-says-Islamist-motive-Moscow-killing-nonsensical.html"                                                  
 [8] "http://www.dailymail.co.uk/wires/ap/article-2985778/10-Things-Know-Monday--9-March-2015.html"                                                                                  
 [9] "http://www.dailymail.co.uk/news/article-2985383/Suspect-Nemtsov-killing-devout-Muslim-shocked-Charlie-Hebdo-cartoons-Chechen-leader.html"                                      
[10] "http://www.dailymail.co.uk/video/news/video-1165510/Suspects-in-killing-of-Boris-Nemtsov-appear-in-Moscow-courthouse.html"                                                     
[11] "http://www.dailymail.co.uk/video/news/video-1165505/Boris-Nemtsov-shooting-suspects-seen-caged-in-court.html"                                                                  
[12] "http://www.dailymail.co.uk/wires/reuters/article-2985103/Russian-court-orders-three-Nemtsov-suspects-kept-custody.html"                                                        
[13] "http://www.dailymail.co.uk/news/article-2985051/Five-suspects-court-assassination-Vladimir-Putin-critic-Boris-Nemtsov-one-revealed-policeman-served-Chechnya.html"             
[14] "http://www.dailymail.co.uk/wires/reuters/article-2985011/Russian-authorities-charge-two-Nemtsov-shooting.html"                                                                 
[15] "http://www.dailymail.co.uk/wires/ap/article-2984912/Nemtsov-killing-suspects-whereabouts-unknown.html"                                                                         
[16] "http://www.dailymail.co.uk/wires/ap/article-2984876/Sunday-March-8-2015.html"                                                                                                  
[17] "http://www.dailymail.co.uk/wires/reuters/article-2984865/One-Nemtsov-killing-suspects-served-police-report.html"                                                               
[18] "http://www.dailymail.co.uk/wires/reuters/article-2984828/One-Nemtsov-killing-suspects-served-police-report.html"                                                               
[19] "http://www.dailymail.co.uk/video/news/video-1165340/Russia-announces-two-arrests-in-Boris-Nemtsov-murder.html"                                                                 
[20] "http://www.dailymail.co.uk/wires/ap/article-2983910/Saturday-March-7-2015.html"                                                                                                
[21] "http://www.dailymail.co.uk/wires/reuters/article-2983867/Two-suspects-detained-Nemtsov-murder--Russian-security-chief.html"                                                    
[22] "http://www.dailymail.co.uk/news/article-2983864/Russia-arrests-two-murder-Putin-opponent-gunned-outside-Kremlin.html"                                                          
[23] "http://www.dailymail.co.uk/wires/ap/article-2983856/Russia-2-suspects-detained-murder-Boris-Nemtsov.html"                                                                      
[24] "http://www.dailymail.co.uk/wires/reuters/article-2983240/Nemtsovs-friends-ask-police-shot.html"                                                                                
[25] "http://www.dailymail.co.uk/wires/reuters/article-2982521/Kremlin-critic-Navalny-jail-vows-fight-on.html"                                                                       
[26] "http://www.dailymail.co.uk/news/article-2982499/Ukraine-puts-murdered-Russian-opposition-chief-s-glamorous-model-girlfriend-secret-service-protection-death-threats.html"      
[27] "http://www.dailymail.co.uk/wires/ap/article-2982330/Kremlin-critic-Navalny-released-15-days-custody.html"                                                                      
[28] "http://www.dailymail.co.uk/wires/ap/article-2982306/Witness-Russian-politicians-slaying-gets-death-threats.html"                                                               
[29] "http://www.dailymail.co.uk/wires/reuters/article-2982305/Kremlin-critic-Navalny-walks-Moscow-jail.html"                                                                        
[30] "http://www.dailymail.co.uk/news/article-2982194/Hand-scrawled-notes-Russian-opposition-figure-written-day-shot-dead-trail-Moscow-backing-separatist-fighters.html"             
[31] "http://www.dailymail.co.uk/wires/reuters/article-2981498/Putins-suppression-blame-Nemtsovs-death--NATO-official.html"                                                          
[32] "http://www.dailymail.co.uk/wires/reuters/article-2981421/Scribbled-note-shows-Nemtsov-trail-Russian-deaths-Ukraine.html"                                                       
[33] "http://www.dailymail.co.uk/wires/reuters/article-2981085/Italys-Renzi-lays-flowers-Moscow-bridge-Nemtsov-killed.html"                                                          
[34] "http://www.dailymail.co.uk/wires/ap/article-2981048/Italian-PM-Moscow-discuss-Russia-EU-ties.html"                                                                             
[35] "http://www.dailymail.co.uk/video/news/video-1164466/Vladimir-Putin-calls-the-murder-of-Boris-Nemtsov-political.html"                                                           
[36] "http://www.dailymail.co.uk/wires/reuters/article-2979242/Britain-wont-Fridmans-North-Sea-deal-RWE-government-source.html"                                                      
[37] "http://www.dailymail.co.uk/news/article-2979206/Vladimir-Putin-calls-end-shameful-political-assassinations-Russia-shooting-Kremlin-critic-Boris-Nemtsov.html"                  
[38] "http://www.dailymail.co.uk/wires/reuters/article-2979068/Russias-Putin-calls-Nemtsovs-murder-political.html"                                                                   
[39] "http://www.dailymail.co.uk/wires/ap/article-2979019/Putin-says-opposition-politicians-murder-disgrace.html"                                                                    
[40] "http://www.dailymail.co.uk/wires/ap/article-2977725/Kremlin-critic-wants-international-probe-Nemtsov-death.html"                                                               
[41] "http://www.dailymail.co.uk/wires/reuters/article-2977688/Russia-bars-two-EU-politicians-Nemtsov-funeral.html"                                                                  
[42] "http://www.dailymail.co.uk/video/news/video-1164117/Crowds-pay-their-last-respects-by-the-coffin-of-Boris-Nemtsov.html"                                                        
[43] "http://www.dailymail.co.uk/wires/reuters/article-2977491/Russia-bars-two-EU-dignitaries-Nemtsov-funeral.html"                                                                  
[44] "http://www.dailymail.co.uk/wires/ap/article-2977522/EU-condemns-Russia-Nemtsov-funeral-bans.html"                                                                              
[45] "http://www.dailymail.co.uk/wires/reuters/article-2977465/Merkel-says-push-freedom-expression-Russia.html"                                                                      
[46] "http://www.dailymail.co.uk/wires/reuters/article-2977289/Russians-stand-line-mourn-coffin-murdered-Nemtsov.html"                                                               
[47] "http://www.dailymail.co.uk/news/article-2977093/Mourners-pay-respects-murdered-Russian-opposition-leader-laid-state-open-casket.html"                                          
[48] "http://www.dailymail.co.uk/money/markets/article-2977047/FTSE-LIVE-Footsie-rides-high-record-gains-overnight-Dow-Jones-S-P-500-hit-time-highs.html"                            
[49] "http://www.dailymail.co.uk/wires/reuters/article-2977048/Mourners-pay-respects-murdered-Kremlin-critic-Nemtsov.html"                                                           
[50] "http://www.dailymail.co.uk/wires/ap/article-2976976/Mourners-view-body-slain-opposition-leader-Boris-Nemtsov.html"                                                             
[51] "http://www.dailymail.co.uk/wires/reuters/article-2976581/President-Obama-Israels-Netanyahu-clash-Iran-diplomacy.html"                                                          
```

```r
links=paste('http://www.dailymail.co.uk',links,sep='')
```


Scrape BBC
=======================

Do the same for this page:

http://www.bbc.co.uk/search?q=nemtsov










Solution
============


```r
url <- 'http://www.bbc.co.uk/search?q=nemtsov'
raw <-  getURL(url,encoding="UTF-8") 
PARSED <- htmlParse(raw) #Format the html code d
links<-xpathSApply(PARSED, "//a/@href")
length(links)
```

```
[1] 114
```

```r
links<-xpathSApply(PARSED, '//ol[@class="search-results results"]//a/@href')
length(links)
```

```
[1] 30
```

```r
length(unique(links))
```

```
[1] 10
```

```r
links<-unique(links)
```

Make function to get links
===========


```r
getBBCLinks <- function(url){
  raw <-  getURL(url,encoding="UTF-8") 
  PARSED <- htmlParse(raw) #Format the html code d
  links<-unique(xpathSApply(PARSED, 
      '//ol[@class="search-results results"]//a/@href'))
  return (links)
}


url='http://www.bbc.co.uk/search?q=putin'
links <-getBBCLinks(url)
links
```

```
 [1] "http://www.bbc.co.uk/news/world-europe-31794742" 
 [2] "http://www.bbc.co.uk/news/world-europe-31796226" 
 [3] "http://www.bbc.co.uk/news/world-europe-31792991" 
 [4] "http://www.bbc.co.uk/news/world-europe-31794523" 
 [5] "http://www.bbc.co.uk/news/uk-politics-31787910"  
 [6] "http://www.bbc.co.uk/news/uk-politics-31787908"  
 [7] "http://www.bbc.co.uk/news/election-2015-31787285"
 [8] "http://www.bbc.co.uk/news/uk-politics-31786906"  
 [9] "http://www.bbc.co.uk/news/world-europe-31728623" 
[10] "http://www.bbc.co.uk/news/uk-31809453"           
```


BBC article scraper
================


```r
url <- "http://www.bbc.co.uk/news/world-europe-26333587"
# Specify encoding when dealing with non-latin characters
SOURCE <-  getURL(url,encoding="UTF-8") 
PARSED <- htmlParse(SOURCE)
```
Get the headline:

```r
(xpathSApply(PARSED, "//h1[@class='story-header']",xmlValue))
```

```
[1] "Ukraine crisis: Turchynov warns of 'separatism' risk"
```
Get the date:

```r
(xpathSApply(PARSED, "//span[@class='date']",xmlValue))
```

```
[1] "25 February 2014"
```



Make a scraper
===============


```r
bbcScraper <- function(url){
  date=''
  title=''
  SOURCE <-  getURL(url,encoding="UTF-8")
  PARSED <- htmlParse(SOURCE)
  title=xpathSApply(PARSED, "//title",xmlValue)
  date=xpathSApply(PARSED, "//meta[@name='OriginalPublicationDate']/@content")
  d=data.frame(url,title,date)
  return(d)
}

url='http://www.bbc.co.uk/search?q=putin'
links <-getBBCLinks(url)

dat <- ldply(links,bbcScraper)
```


Guardian
=======================

start with the bbc scraper as a base, then change the necessary fields

```r
url <- "http://www.theguardian.com/environment/2015/mar/08/
  how-will-everything-change-under-climate-change"
  SOURCE <-  getURL(url,encoding="UTF-8")
  PARSED <- htmlParse(SOURCE)
  xpathSApply(PARSED, "//h1[contains(@itemprop,'headline')]",xmlValue)
```

```
[1] "\nHow will everything change under climate change?\n"
```

```r
  xpathSApply(PARSED, "//a[@rel='author']",xmlValue)
```

```
[1] "Naomi Klein"
```

```r
  xpathSApply(PARSED, "//time[@itemprop='datePublished']",xmlValue)
```

```
[1] "\nSunday 8 March 2015 12.00 GMT\n"
```

Guardian continued
===========



```r
  xpathSApply(PARSED, "//time[@itemprop='datePublished']/@datetime")
```

```
                  datetime 
"2015-03-08T12:00:04+0000" 
```

```r
  xpathSApply(PARSED, "//li[@class='inline-list__item ']/a",xmlValue)
```

```
[1] "\nClimate change\n"           "\nGreenhouse gas emissions\n"
[3] "\nFossil fuels\n"             "\nEnergy\n"                  
```

```r
  xpathSApply(PARSED, "//div[@itemprop='articleBody']",xmlValue)
```

```
[1] "\nThe alarm bells of the climate crisis have been ringing in our ears for years and are getting louder all the time - yet humanity has failed to change course. What is wrong with us?\nMany answers to that question have been offered, ranging from the extreme difficulty of getting all the governments in the world to agree on anything, to an absence of real technological solutions, to something deep in our human nature that keeps us from acting in the face of seemingly remote threats, to – more recently – the claim that we have blown it anyway and there is no point in even trying to do much more than enjoy the scenery on the way down.\nSome of these explanations are valid, but all are ultimately inadequate. Take the claim that it’s just too hard for so many countries to agree on a course of action. It is hard. But many times in the past, the United Nations has helped governments to come together to tackle tough cross-border challenges, from ozone depletion to nuclear proliferation. The deals produced weren’t perfect, but they represented real progress. Moreover, during the same years that our governments failed to enact a tough and binding legal architecture requiring emission reductions, supposedly because cooperation was too complex, they managed to create the World Trade Organisation – an intricate global system that regulates the flow of goods and services around the planet, under which the rules are clear and violations are harshly penalised.\nThe assertion that we have been held back by a lack of technological solutions is no more compelling. Power from renewable sources like wind and water predates the use of fossil fuels and is becoming cheaper, more efficient, and easier to store every year. The past two decades have seen an explosion of ingenious zero-waste design, as well as green urban planning. Not only do we have the technical tools to get off fossil fuels, we also have no end of small pockets where these low carbon lifestyles have been tested with tremendous success. And yet the kind of large-scale transition that would give us a collective chance of averting catastrophe eludes us.\nIs it just human nature that holds us back then? In fact we humans have shown ourselves willing to collectively sacrifice in the face of threats many times, most famously in the embrace of rationing, victory gardens, and victory bonds during world wars one and two. Indeed to support fuel conservation during world war two, pleasure driving was virtually eliminated in the UK, and between 1938 and 1944, use of public transit went up by 87% in the US and by 95% in Canada. Twenty million US households – representing three fifths of the population – were growing victory gardens in 1943, and their yields accounted for 42% of the fresh vegetables consumed that year. Interestingly, all of these activities together dramatically reduce carbon emissions. \nYes, the threat of war seemed immediate and concrete but so too is the threat posed by the climate crisis that has already likely been a substantial contributor to massive disasters in some of the world’s major cities. Still, we’ve gone soft since those days of wartime sacrifice, haven’t we? Contemporary humans are too self-centered, too addicted to gratification to live without the full freedom to satisfy our every whim – or so our culture tells us every day. And yet the truth is that we continue to make collective sacrifices in the name of an abstract greater good all the time. We sacrifice our pensions, our hard-won labour rights, our arts and after-school programmes. We accept that we have to pay dramatically more for the destructive energy sources that power our transportation and our lives. We accept that bus and subway fares go up and up while service fails to improve or degenerates. We accept that a public university education should result in a debt that will take half a lifetime to pay off when such a thing was unheard of a generation ago. \nThe past 30 years have been a steady process of getting less and less in the public sphere. This is all defended in the name of austerity, the current justification for these never-ending demands for collective sacrifice. In the past, calls for balanced budgets, greater efficiency, and faster economic growth have served the same role. \nEmail\nIt seems to me that if humans are capable of sacrificing this much collective benefit in the name of stabilising an economic system that makes daily life so much more expensive and precarious, then surely humans should be capable of making some important lifestyle changes in the interest of stabilising the physical systems upon which all of life depends. Especially because many of the changes that need to be made to dramatically cut emissions would also materially improve the quality of life for the majority of people on the planet – from allowing kids in Beijing to play outside without wearing pollution masks to creating good jobs in clean energy sectors for millions. \nTime is tight, to be sure. But we could commit ourselves, tomorrow, to radically cutting our fossil fuel emissions and beginning the shift to zero-carbon sources of energy based on renewable technology, with a full-blown transition underway within the decade. We have the tools to do that. And if we did, the seas would still rise and the storms would still come, but we would stand a much greater chance of preventing truly catastrophic warming. Indeed, entire nations could be saved from the waves.\nSo my mind keeps coming back to the question: what is wrong with us? I think the answer is far more simple than many have led us to believe: we have not done the things that are necessary to lower emissions because those things fundamentally conflict with deregulated capitalism, the reigning ideology for the entire period we have been struggling to find a way out of this crisis. We are stuck because the actions that would give us the best chance of averting catastrophe – and would benefit the vast majority – are extremely threatening to an elite minority that has a stranglehold over our economy, our political process, and most of our major media outlets. That problem might not have been insurmountable had it presented itself at another point in our history. But it is our great collective misfortune that the scientific community made its decisive diagnosis of the climate threat at the precise moment when those elites were enjoying more unfettered political, cultural, and intellectual power than at any point since the 1920s. Indeed, governments and scientists began talking seriously about radical cuts to greenhouse gas emissions in 1988 – the exact year that marked the dawning of what came to be called “globalisation,” with the signing of the agreement representing the world’s largest bilateral trade relationship between Canada and the US, later to be expanded into the North American Free Trade Agreement (Nafta) with the inclusion of Mexico. \nThe three policy pillars of this new era are familiar to us all: privatisation of the public sphere, deregulation of the corporate sector, and lower corporate taxation, paid for with cuts to public spending. Much has been written about the real-world costs of these policies – the instability of financial markets, the excesses of the super-rich, and the desperation of the increasingly disposable poor, as well as the failing state of public infrastructure and services. Very little, however, has been written about how market fundamentalism has, from the very first moments, systematically sabotaged our collective response to climate change. \nThe core problem was that the stranglehold that market logic secured over public life in this period made the most direct and obvious climate responses seem politically heretical. How, for instance, could societies invest massively in zero-carbon public services and infrastructure at a time when the public sphere was being systematically dismantled and auctioned off? How could governments heavily regulate, tax, and penalise fossil fuel companies when all such measures were being dismissed as relics of “command and control” communism? And how could the renewable energy sector receive the supports and protections it needed to replace fossil fuels when “protectionism” had been made a dirty word?\nEven more directly, the policies that so successfully freed multinational corporations from virtually all constraints also contributed significantly to the underlying cause of global warming – rising greenhouse gas emissions. The numbers are striking: In the 1990s, as the market integration project ramped up, global emissions were going up an average of one percent a year; by the 2000s, with “emerging markets” like China now fully integrated into the world economy, emissions growth had sped up disastrously, with the annual rate of increase reaching 3.4% a year for much of the decade. That rapid growth rate continues to this day, interrupted only briefly in 2009 by the world financial crisis. Emissions rebounded with a vengeance in 2010, which saw the largest absolute increase since the Industrial Revolution.\n\n  Sorry, your browser is unable to play this video. \n\n\n Facebook \n Twitter \n Pinterest \n\n\nThis Changes Everything - excerpt from work-in-progress feature documentary\nWith hindsight, it’s hard to see how it could have turned out otherwise. The twin signatures of this era have been the mass export of products across vast distances (relentlessly burning carbon all the way), and the import of a uniquely wasteful model of production, consumption, and agriculture to every corner of the world (also based on the profligate burning of fossil fuels). Put differently, the liberation of world markets, a process powered by the liberation of unprecedented amounts of fossil fuels from the earth, has dramatically sped up the same process that is liberating Arctic ice from existence.\nAs a result, we now find ourselves in a very difficult and slightly ironic position. Because of those decades of hardcore emitting, exactly when we were supposed to be cutting back, the things we must do to avoid catastrophic warming are no longer just in conflict with the particular strain of deregulated capitalism that triumphed in the 1980s. They are now in conflict with the fundamental imperative at the heart of our economic model: grow or die.\nOnce carbon has been emitted into the atmosphere, it sticks around for hundreds of years, some of it even longer, trapping heat. The effects are cumulative, growing more severe with time. And according to emissions specialists like the Tyndall Centre’s Kevin Anderson (as well as others), so much carbon has been allowed to accumulate in the atmosphere over the past two decades that now our only hope of keeping warming below the internationally agreed-upon target of 2C is for wealthy countries to cut their emissions by somewhere in the neighbourhood of eight to 10% a year. The “free” market simply cannot accomplish this task. Indeed, this level of emission reduction has happened only in the context of economic collapse or deep depressions.\nWhat those numbers mean is that our economic system and our planetary system are now at war. Or, more accurately, our economy is at war with many forms of life on earth, including human life. What the climate needs to avoid collapse is a contraction in humanity’s use of resources; what our economic model demands to avoid collapse is unfettered expansion. Only one of these sets of rules can be changed, and it’s not the laws of nature.\nFortunately, it is eminently possible to transform our economy so that it is less resource-intensive, and to do it in ways that are equitable, with the most vulnerable protected and the most responsible bearing the bulk of the burden. Low-carbon sectors of our economies can be encouraged to expand and create jobs, while high-carbon sectors are encouraged to contract. The problem, however, is that this scale of economic planning and management is entirely outside the boundaries of our reigning ideology. The only kind of contraction our current system can manage is a brutal crash, in which the most vulnerable will suffer most of all.\n\n\n\n\n Facebook \n Twitter \n Pinterest \n\nPacific Ocean VIII (Gladstone 4 Fish), Santa Monica, USA, 2001 © Nadav Kander. Courtesy Flowers Gallery, London.\nSo we are left with a stark choice: allow climate disruption to change everything about our world, or change pretty much everything about our economy to avoid that fate. But we need to be very clear: because of our decades of collective denial, no gradual, incremental options are now available to us. Gentle tweaks to the status quo stopped being a climate option when we supersized the American Dream in the 1990s, and then proceeded to take it global. And it’s no longer just radicals who see the need for radical change. In 2012, 21 past winners of the prestigious Blue Planet Prize – a group that includes James Hansen, former director of Nasa’s Goddard Institute for Space Studies, and Gro Harlem Brundtland, former prime minister of Norway – authored a landmark report. It stated that, “in the face of an absolutely unprecedented emergency, society has no choice but to take dramatic action to avert a collapse of civilization. Either we will change our ways and build an entirely new kind of global society, or they will be changed for us.”\nThat’s tough for a lot of people in important positions to accept, since it challenges something that might be even more powerful than capitalism, and that is the fetish of centrism – of reasonableness, seriousness, splitting the difference, and generally not getting overly excited about anything. This is the habit of thought that truly rules our era, far more among the liberals who concern themselves with matters of climate policy than among conservatives, many of whom simply deny the existence of the crisis. Climate change presents a profound challenge to this cautious centrism because half measures won’t cut it: “all of the above energy” program, as US president Barack Obama describes his approach, has about as much chance of success as an all-of-the-above diet, and the firm deadlines imposed by science require that we get very worked up indeed.\nThe challenge, then, is not simply that we need to spend a lot of money and change a lot of policies; it’s that we need to think differently, radically differently, for those changes to be remotely possible. A worldview will need to rise to the fore that sees nature, other nations, and our own neighbours not as adversaries, but rather as partners in a grand project of mutual reinvention.\nThat’s a big ask. But it gets bigger. Because of our endless  procrastination, we also have to pull off this massive transformation without delay. The International Energy Agency (IEA) warns that if we do not get our emissions under control by a rather terrifying 2017, our fossil fuel economy will “lock-in” extremely dangerous warming. “The energy-related infrastructure then in place will generate all the CO2 emissions allowed” in our carbon budget for limiting warming to 2C – “leaving no room for additional power plants, factories and other infrastructure unless they are zero-carbon, which would be extremely costly”. This assumes, probably accurately, that governments would be unwilling to force the closure of still profitable power plants and factories. As Fatih Birol, the IEA’s chief economist, bluntly put it: “The door to reach two degrees is about to close. In 2017 it will be closed forever.” In short, we have reached what some activists have started calling “Decade Zero” of the climate crisis: we either change now or we lose our chance. All this means that the usual free market assurances – A techno-fix is around the corner! Dirty development is just a phase on the way to a clean environment, look at 19th-century London! – simply don’t add up. We don’t have a century to spare for China and India to move past their Dickensian phases. Because of our lost decades, it is time to turn this around now. Is it possible? Absolutely. Is it possible without challenging the fundamental logic of deregulated capitalism? Not a chance. \n\n\nI was struck recently by a mea culpa of sorts, written by Gary Stix, a senior editor of Scientific American. Back in 2006, he edited a special issue on responses to climate change and, like most such efforts, the articles were narrowly focused on showcasing exciting low-carbon technologies. \n\nBut in 2012 Stix wrote that he had overlooked a much larger and more important part of the story – the need to create the social and political context in which these technological shifts stand a chance of displacing the all too profitable status quo. “If we are ever to cope with climate change in any fundamental way, radical solutions on the social side are where we must focus, though. The relative efficiency of the next generation of solar cells is trivial by comparison.”\nIn other words, our problem has a lot less to do with the mechanics of solar power than the politics of human power – specifically whether there can be a shift in who wields it, a shift away from corporations and toward communities, which in turn depends on whether or not the great many people who are getting a rotten deal under our current system can build a determined and diverse enough social force to change the balance of power. Such a shift would require rethinking the very nature of humanity’s power – our right to extract ever more without facing consequences, our capacity to bend complex natural systems to our will. This is a shift that challenges not only capitalism, but also the building blocks of materialism that preceded modern capitalism, a mentality some call “extractivism”.\nBecause, underneath all of this is the real truth we have been avoiding: climate change isn’t an “issue” to add to the list of things to worry about, next to healthcare and taxes. It is a civilisational wake-up call. A powerful message – spoken in the language of fires, floods, droughts, and extinctions – telling us that we need an entirely new economic model and a new way of sharing this planet. Telling us that we need to evolve.\nSome say there is no time for this transformation; the crisis is too pressing and the clock is ticking. I agree that it would be reckless to claim that the only solution to this crisis is to revolutionise our economy and revamp our worldview from the bottom up – and anything short of that is not worth doing. There are all kinds of measures that would lower emissions substantively that could and should be done right now. But we aren’t taking those measures, are we? The reason is that by failing to fight these big battles that stand to shift our ideological direction and change the balance of who holds power in our societies, a context has been slowly created in which any muscular response to climate change seems politically impossible, especially during times of economic crisis. \nOn the other hand, if we can shift the cultural context even a little, then there will be some breathing room for those sensible reformist policies that will at least get the atmospheric carbon numbers moving in the right direction. And winning is contagious so, who knows? \nFor a quarter of a century, we have tried the approach of polite incremental change, attempting to bend the physical needs of the planet to our economic model’s need for constant growth and new profit-making opportunities. The results have been disastrous, leaving us all in a great deal more danger than when the experiment began.\n\n\nLooking for a Moose is one of my two-year-old son’s favourite books. It’s about a bunch of kids that really, really, really want to see a moose. They search high and low – through a forest, a swamp, in brambly bushes and up a mountain, for “a long legged, bulgy nosed, branchy antlered moose”. \n\nThe joke is that there are moose hiding on each page. In the end, the animals all come out of hiding and the ecstatic kids proclaim: “We’ve never ever seen so many moose!”\nOn about the 75th reading, it suddenly hit me: he might never see a moose. I tried to hold it together. I went back to my computer and began to write about my time in northern Alberta, tar sands country, where members of the Beaver Lake Cree Nation told me about how the moose had changed – one woman described killing a moose on a hunting trip only to find that the flesh had already turned green. I heard a lot about strange tumors too, which locals assumed had to do with the animals drinking water contaminated by tar sands toxins. But mostly I heard about how the moose were simply gone.\nAnd not just in Alberta. “Rapid Climate Changes Turn North Woods into Moose Graveyard,” reads a May 2012 headline in Scientific American. A year and a half later, The New York Times was reporting that one of Minnesota’s two moose populations had declined from four thousand in the 1990s to just one hundred today. Will he ever see a moose?\nThen, the other day, I was slain by a miniature board book called Snuggle Wuggle. It involves different animals cuddling, with each posture given a ridiculously silly name: “How does a bat hug?” it asks. “Topsy turvy, topsy turvy.” For some reason my son reliably cracks up at this page. I explain that it means upside down, because that’s the way bats sleep.\nBut all I could think about was the report of some 100,000 dead and dying bats raining down from the sky in the midst of record-breaking heat across part of Queensland, Australia. Whole colonies devastated. Will he ever see a bat?\nWhen fear like that used to creep through my armour of climate change denial, I would do my utmost to stuff it away, change the channel, click past it. Now I try to feel it. It seems to me that I owe it to my son, just as we all owe it to ourselves and one another.\nBut what should we do with this fear that comes from living on a planet that is dying, made less alive every day? First, accept that it won’t go away. That it is a fully rational response to the unbearable reality that we are living in a dying world, a world that a great many of us are helping to kill, by doing things like making tea and driving to the grocery store and yes, okay, having kids.\nNext, use it. Fear is a survival response. Fear makes us run, it makes us leap, it can make us act superhuman. But we need somewhere to run to. Without that, the fear is only paralysing. So the real trick, the only hope, really, is to allow the terror of an unlivable future to be balanced and soothed by the prospect of building something much better than many of us have previously dared hope.\nYes, there will be things we will lose, luxuries some of us will have to give up, whole industries that will disappear. Climate change is already here, and increasingly brutal disasters are headed our way no matter what we do. But it’s not too late to avert the worst, and there is still time to change ourselves so that we are far less brutal to one another when those disasters strike. And that, it seems to me, is worth a great deal.\nBecause the thing about a crisis this big, this all-encompassing, is that it changes everything. It changes what we can do, what we can hope for, what we can demand from ourselves and our leaders. It means there is a whole lot of stuff that we have been told is inevitable that simply cannot stand. And it means that a whole lot of stuff we have been told is impossible has to start happening right away.\nCan we pull it off? All I know is that nothing is inevitable. Nothing except that climate change changes everything. And for a very brief time, the nature of that change is still up to us.\nExtracted from THIS CHANGES EVERYTHING: Capitalism vs the Climate by Naomi Klein published this week in paperback by Penguin, £8.99 \n"
```


Guardian scraper
======================


```r
guardianScraper <- function(url){
  SOURCE <-  getURL(url,encoding="UTF-8")
  PARSED <- htmlParse(SOURCE)
  headline = author =date = tags = body =''
  headline<-xpathSApply(PARSED, "//h1[contains(@itemprop,'headline')]",xmlValue)
  author<-xpathSApply(PARSED, "//a[@rel='author']",xmlValue)[1]
  date<-as.character(xpathSApply(PARSED, 
      "//time[@itemprop='datePublished']/@datetime"))
  tags<-xpathSApply(PARSED, "//li[@class='inline-list__item ']/a",xmlValue)
  tags<-paste(tags,collapse=',')
  body<-xpathSApply(PARSED, "//div[@itemprop='articleBody']",xmlValue)
  d<- tryCatch(
    {
      d=data.frame(url,headline,author,date,tags,body)      
    },error=function(cond){
      print (paste('failed for page',url))
      return(NULL)
    }
    )
}
```

Get Guardian links
=====================

scraping is getting harder


```r
url='http://www.theguardian.com/world/russia?page=7'
getGuardianLinks <- function(url){
  raw <-  getURL(url,encoding="UTF-8") 
  PARSED <- htmlParse(raw) #Format the html code d
  links<-unique(xpathSApply(PARSED, '//div[@class="fc-item__container"]//a/@href'))
  return (links)
}

links <- getGuardianLinks(url)
dat<-ldply(links[1:8],guardianScraper)
```

```
[1] "failed for page http://get.adobe.com/flashplayer/"
[1] "failed for page http://whatbrowser.org/"
[1] "failed for page http://www.theguardian.com/world/video/2015/feb/10/
    rockets-hit-residential-area-kramatorsk-ukraine-video"
[1] "failed for page http://www.theguardian.com/sport/2015/feb/10/
    marat-safin-denies-tax-evasion-allegations"
```

```r
head(dat[,1:5])
```

```
                                                                                                    url
1                     http://www.theguardian.com/world/2015/feb/10/minsk-summit-talks-ukraine-ceasefire
2   http://www.theguardian.com/politics/2015/feb/10/uk-government-defends-role-in-ukraine-russia-crisis
3 http://www.theguardian.com/world/2015/feb/10/litvinenko-inquiry-suspect-drank-porn-star-dmitry-kovtun
4    http://www.theguardian.com/world/2015/feb/10/ukraine-draft-dodgers-jail-kiev-struggle-new-fighters
                                                                     headline
1                      \nLeaders locked in Minsk talks on Ukraine ceasefire\n
2                     \nUK government defends role in Ukraine-Russia crisis\n
3    \nLitvinenko inquiry: suspect 'drank a lot and wanted to be porn star'\n
4 \nUkraine: draft dodgers face jail as Kiev struggles to find new fighters\n
             author                     date
1       Ian Traynor 2015-02-10T17:17:43+0000
2 Frances Perraudin 2015-02-10T16:55:55+0000
3      Luke Harding 2015-02-10T16:55:47+0000
4      Shaun Walker 2015-02-10T15:28:35+0000
                                                                                           tags
1                                                 \nUkraine\n,\nBelarus\n,\nRussia\n,\nEurope\n
2 \nForeign policy\n,\nUkraine\n,\nPhilip Hammond\n,\nDouglas Alexander\n,\nRussia\n,\nEurope\n
3                                                           \nAlexander Litvinenko\n,\nRussia\n
4                     \nUkraine\n,\nRussia\n,\nPetro Poroshenko\n,\nVladimir Putin\n,\nEurope\n
```

Tidy the data
=====================


```r
gsub('\n','',dat$tags)
as.Date(dat$date)
gsub('\n','',dat$headline)

dat$tags <-gsub('\n','',dat$tags)
dat$date <-as.Date(dat$date)
dat$headline <-gsub('\n','',dat$headline)
```

lubridate for dates
============


```r
time="2015-02-09T14:59:32+0000"
ymd_hms(time)
```

```
[1] "2015-02-09 14:59:32 UTC"
```

```r
time="\nMonday 23 February 2015\n"
dmy(time)
```

```
[1] "2015-02-23 UTC"
```

```r
time='06/12/2013'
dmy(time)
```

```
[1] "2013-12-06 UTC"
```

```r
mdy(time)
```

```
[1] "2013-06-12 UTC"
```


Practice time
================


write a scraper for

- http://www.mirror.co.uk/
- http://www.telegraph.co.uk/
- http://www.independent.co.uk




Solutions (1: Mirror)
==================


Can you turn these into scraper functions?
Mirror

```r
#MIRROR
url <- "http://www.mirror.co.uk/news/world-news/
    oscar-pistorius-trial-murder-reeva-3181393"
SOURCE <-  getURL(url,encoding="UTF-8") 
PARSED <- htmlParse(SOURCE)
title <- xpathSApply(PARSED, "//h1",xmlValue)
author <- xpathSApply(PARSED, "//li[@class='author']",xmlValue)
time  <- xpathSApply(PARSED, "//time[@itemprop='datePublished']/@datetime")
```

Telegraph
============



```r
#Telegraph
url <- "http://www.telegraph.co.uk/news/uknews/terrorism-in-the-uk/10659904/
    Former-Guantanamo-detainee-Moazzam-Begg-one-of-four-arrested-on-suspicion
    -of-terrorism.html"
SOURCE <-  getURL(url,encoding="UTF-8") 
PARSED <- htmlParse(SOURCE)
title <- xpathSApply(PARSED, "//h1[@itemprop='headline name']",xmlValue)
author <- xpathSApply(PARSED, "//p[@class='bylineBody']",xmlValue)
time  <- xpathSApply(PARSED, "//p[@class='publishedDate']",xmlValue)
```

Independent
==============


```r
#Independent
url <- "http://www.independent.co.uk/news/people/emma-watson-issues-feminist-
    response-to-prince-harry-speculation-marrying-a-prince-is-not-prerequisite-
    to-being-a-princess-10064025.html"
SOURCE <-  getURL(url,encoding="UTF-8")
PARSED <- htmlParse(SOURCE)
title <- xpathSApply(PARSED, "//h1",xmlValue)
author <- xpathSApply(PARSED, "//span[@class='authorName']",xmlValue)
time  <- xpathSApply(PARSED, "//p[@class='dateline']",xmlValue)
as.Date(str_trim(time),"%d %B %Y")
```


Where do we go from here
=======================
Scale up, store data, retrieve:

- R + sql + data.table
- python + mongodb
- python + selenium -> javascript and logons. 

R for tables, vector operations, numbers.

Python for (irregular) objects, loops, text processing.

