# Homework 4: MongoDB database of image analysis from Google Cloud Vision API
### Due: December 10, 2018
### Prof Michael Mandel `mim@sci.brooklyn.cuny.edu`

## Introduction

For this assignment, you will be interacting with a set of text documents extracted from HTML pages using the Whoosh search engine in Python. The HTML pages are the pages that the Google Cloud Vision API identified as containing the images that we have been working with.

I have provided starter code in Python and because you will be using Whoosh, you must complete the assignment in Python.  You will submit your code, the output of your queries, and a brief report describing your approach.


### Install and setup Whoosh

Use `pip` to install the `whoosh` package:

```bash
pip install Whoosh
```

The following code should run from the commandline and print "Worked!" without generating any errors.  If it doesn't, then you have a problem with your python configuration or the installation of the `Whoosh` package.

```bash
python -c 'import whoosh; print("Worked!")'
```


### Introduction to the data (derived from data for homeworks 1, 2, and 3)

The data is derived from the data from homeworks 1, 2, and 3.  It is provided in this repository in the `data/` directory.  You will be working with the data in `data/pageText/` which is purely textual data.  It is derived from the HTML pages in `data/pages/` which is only provided for your reference.
   
### Introduction to starter code

All of the python code is contained in the file [`runWhoosh.py`](https://github.com/cisc7610/homework4/blob/master/runWhoosh.py).
If you have all of the necessary dependencies installed, you should be able to run the script as it is to populate the search engine and perform a basic search.

If it is working, it should print out (among other things):
```
************************************************************
Top 20 terms
************************************************************


new	1.238054351312873
york	1.2138877407557376
was	1.1528091018625797
city	1.1521471877477851
all	1.1282244248428333
photos	1.123346974800077
flickr	1.120673627748619
de	1.1151258493496314
my	1.1135350567601834
but	1.0760340756525266
one	1.0677393268888629
art	1.0618331867941
has	1.0591524293950991
orleans	1.052347558290222
bridge	1.0393962386859077
world	1.035395129299458
la	1.0321754629743232
there	1.0315826984389482
their	1.0279772967904774
out	1.0169593132866073



************************************************************
Query:   new york   returned 196 results
************************************************************


Url: indulgy.com/post/9oS6UVpli2/empire-state-building-at-night-new-york-city
Highlight: State Building, NEW YORK City
Collect this now...State Building, NEW YORK City, 1957
Collect this...State Building ~ NEW YORK City, NEW YORK ✅
Collect this now for


Url: www.flickr.com/photos/tags/New_York
Highlight: guide possible to NEW YORK City!
Time Square : NEW YORK City
Time Square : NEW YORK City
Time Square has...questions. Thank you
NEW YORK - The City of Gold
NEW YORK Skyline Panorama
NEW YORK at Twilight, a Panorama...State Building
NEW YORK Public Library
NEW YORK from Brooklyn
NEW YORK depuis Brooklyn au


Url: indulgy.com/post/nGdsQo4VJ2/george-rodger-new-york-city-the-empire-stat
Highlight: State Building ~ NEW YORK City, NEW YORK ✅
Collect this now for...State Building in NEW YORK City 8/18/14
Collect...Empire State - NEW YORK City
Collect this now


Url: www.etsy.com/listing/200815622/nyc-photography-coney-island-boardwalk
Highlight: street scene gold
                NEW YORK, NYC, state print, NEW YORK City, NYC art print, NEW YORK Map, I Love NY, NEW YORK I Love You
        Eligible...Storm Photography, NEW YORK Photography, NEW YORK Wall Art Prints, NEW YORK City Prints, NYC Wall...Flowers, Florals, NEW YORK City Photography
                NEW YORK Skyline, NEW YORK, NEW YORK City, NEW YORK Art, Manhattan, Manhattan


Url: www.localdatabase.com/newyork/New_York
Highlight: Showroom
Photos of NEW YORK, NY
NEW YORK Population, Income...the English named it NEW YORK after the Duke of YORK.
NEW YORK City was once called...lost all their teeth
NEW YORK College Maps
NEW YORK Police Station Maps





************************************************************
Query:   empire state building   returned 47 results
************************************************************


Url: indulgy.com/post/9oS6UVpli2/empire-state-building-at-night-new-york-city
Highlight: this now for later
	                EMPIRE STATE BUILDING at night - New York...Bridge, Central Park, EMPIRE STATE BUILDING, Chrysler BUILDING and other popular...this now for later
                EMPIRE STATE BUILDING, New York BUILDING
Collect this now


Url: indulgy.com/post/nGdsQo4VJ2/george-rodger-new-york-city-the-empire-stat
Highlight: New York City. The EMPIRE STATE BUILDING. The observatory...City skyline and the EMPIRE STATE BUILDING
Collect this now...Chrysler BUILDING  The EMPIRE STATE BUILDING isnt the only New


Url: www.flickr.com/photos/danielmennerich/5443157673
Highlight: New York City USA - EMPIRE STATE BUILDING South Manhatten...A nice view from EMPIRE STATE BUILDING South Manhatten


Url: www.flickr.com/photos/danielmennerich/5205390702
Highlight: New York City USA - EMPIRE STATE BUILDING 01-TiltShift
The...EMPIRE STATE BUILDING rises to 381 m at...443.09 m. The BUILDING has 85 stories


Url: www.haikudeck.com/copy-of-richardson-stern-2-uncategorized-presentation-AUxDx8RhJJ
Highlight: up space between the EMPIRE STATE BUILDING ( nucleus ) and the...because the EMPIRE STATE BUILDING ( nucleolus ) is in...AND RNA
We chose the EMPIRE STATE BUILDING because it's the
```

### Introduction to Whoosh

[Whoosh](https://whoosh.readthedocs.io/en/latest/index.html) is a search engine implemented in pure python.  It contains a everything you need for indexing documents and searching them, with several options for each feature.  Specifically, for this assignment we will be exploring the effect of different tokenizers during indexing, different weighting methods during search, and different queries.

We covered an overview of search engines in [lecture 7](http://m.mr-pc.org/t/cisc7610/2018fa/lecture07.pdf).  Whoosh indexes documents, processing different *fields* separately.  In the case of this homework, we are only indexing two fields, `url` and `body` with all of the action taking place in `body`.

Tokenization in Whoosh is accomplished by the `analyzer` used to analyze a field, for `body` this is defined on [Line 30 of runWhoosh.py](https://github.com/cisc7610/homework4/blob/master/runWhoosh.py#L30).  Documentation for analyzers can be found [here](https://whoosh.readthedocs.io/en/latest/analysis.html).  The standard analyzer is the `analysis.RegexTokenizer`.  Multiple analyzers can be chained together using the pipe operator `|`.

Weighting methods determine how documents that are relevant to the query are ranked.  In lecture 7 we discussed (term) frequency and TF-IDF.  The weighting is selected on [Line 79 of runWhoosh.py](https://github.com/cisc7610/homework4/blob/master/runWhoosh.py#L79).  The default weighting is BM25F, which is like TF-IDF but with several extra parameters to adjust its performance in various ways.  Documentation about weightings can be found [here](https://whoosh.readthedocs.io/en/latest/api/scoring.html) listing the standard classes for weightings [here](https://whoosh.readthedocs.io/en/latest/api/scoring.html#scoring-algorithm-classes).


## Tasks

To complete the assignment, perform the following tasks and write the results in the file README.md in this repository.


### Compare different tokenizers

For this part, we will only use the "Top 20 terms" returned after indexing the documents.

Paste here the "Top 20 terms" that result from indexing using each of the following analysis pipelines:
1. `analysis.RegexTokenizer()`
	the     1.3037791555841314
	of      1.2856720639792742
	and     1.285067654715793
	to      1.2846583021453661
	a       1.2730243401508674
	in      1.2703124093026643
	for     1.229095837292915
	you     1.2265749480085957
	is      1.2240995859845578
	on      1.215081249611279
	The     1.2146130510907465
	s       1.2133323980298831
	New     1.199515673705514
	by      1.1939935683353056
	York    1.1896696480958109
	I       1.1887369757029826
	with    1.1879591065884456
	that    1.1747113648649536
	your    1.1661691222580117
	at      1.159907004477876
2. `analysis.RegexTokenizer() | analysis.LowercaseFilter()`
	the     1.3067382932505052
	to      1.28554472925298
	of      1.2854610557193074
	and     1.2853020763716763
	a       1.2764304825577468
	in      1.2736417509116345
	you     1.2378651162720165
	for     1.2350573948756958
	is      1.225883807680983
	on      1.2186198018337329
	new     1.2180882023914608
	s       1.2148533176190297
	by      1.199931671144417
	york    1.1942713168368804
	i       1.1938181978819107
	with    1.192645627507902
	that    1.182916340074976
	it      1.1786306069049397
	this    1.1752068654362056
	your    1.1751407020872995
3. `analysis.RegexTokenizer() | analysis.LowercaseFilter() | analysis.StopFilter()`
	new     1.2379017736230968
	york    1.2136974797038964
	was     1.152531571847837
	city    1.1518687769955114
	all     1.1279151309280322
	photos  1.1230316110126444
	flickr  1.1203549696182569
	de      1.1148004283120685
	my      1.1132077148105946
	but     1.0756638074292622
	one     1.0673601719636745
	art     1.0614478377108494
	has     1.0587643053765545
	orleans 1.051952492615239
	bridge  1.0389883654681298
	world   1.0349834061343315
	la      1.0317606782540332
	there   1.03116735360441
	their   1.027558568831997
	out     1.0165304981304573

List the terms that are present in all cases here
	If we take capitalizion in consideration no terms are present in all.
	Else:
		New
		York

List the terms that are only present in the last case
	was 
	city 
	all 
	photos 
	flickr 
	de 
	my 
	but 
	one 
	art 
	has 
	orleans 
	bridge 
	world 
	la 
	there 
	their 
	out 

Paste here the "Top 20 terms" that result from indexing using the `analysis.StemmingAnalyzer()`.

How are these results different from those using the `analysis.RegexTokenizer()`?

Which top terms seem to be the most descriptive of the content of the web pages?  Why?

Use the `analysis.RegexTokenizer() | analysis.LowercaseFilter() | analysis.StopFilter()` tokenizer for subsequent questions.


### Investigate the top term "de"

List the tokenizers under which "de" is a top 20 term.

One of the top terms is "de".  List the URLs of the documents in which this term occurs (by running a query for it, see [Line 82 of runWhoosh.py](https://github.com/cisc7610/homework4/blob/master/runWhoosh.py#L82)).

What do these pages have in common?


### Compare different weightings

For this part, we will use the two queries, "new york" and "empire state building".

List the number of documents returned for each query and the URLs of the top 20 results for each query using the default weighting, `scoring.BM25F()`.

List the number of documents returned for each query and the URLs of the top 20 results for each query using the weightings:
1. `scoring.TF_IDF()`
1. `scoring.Frequency()`

List the URLs that are present for all weightings.

Which URLs seem to be the most relevant to the queries?  Why?

Use the `scoring.BM25F()` weighting for subsequent questions.


### Measure the precision of several queries

Recall that *precision* is a measure of retrieval performance calculated as the proportion of correct results returned to the total number of results returned.  Normally the correct results are determined by hand, but in this case we will use a more specific query as the target correct results.

Measure the precision of the top 10 results returned by the query "empire state" with respect to retrieving the results of the query "empire state building".  This means that correct results are those that are returned by the query "empire state building" and we are measuring the performance of the query "empire state" in retrieving those pages.  Because the *highlights* might be different between the two queries, make sure to compare URLs, which are unique.

Measure the precision of the top 20 results returned by the query "empire state" with respect to retrieving the results of the query "empire state building".

Measure the precision of the top 20 results returned by the query "new york" with respect to retrieving the results of the query "new york city".


### Describe any problems that you ran into in the course of this project

Describe any problems that you ran into in the course of this project
