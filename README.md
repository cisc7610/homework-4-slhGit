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
```
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
```
2. `analysis.RegexTokenizer() | analysis.LowercaseFilter()`
```
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
```
3. `analysis.RegexTokenizer() | analysis.LowercaseFilter() | analysis.StopFilter()`
```
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
```
List the terms that are present in all cases here
```
	If we take capitalizion in consideration no terms are present in all.
	Else:
		New
		York
```

List the terms that are only present in the last case
```
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
```

Paste here the "Top 20 terms" that result from indexing using the `analysis.StemmingAnalyzer()`.
```
	new     1.2403944269973532
	york    1.213203429589267
	photo   1.168302803218743
	citi    1.1624182021385305
	wa      1.1523694447648907
	all     1.1291666787263475
	de      1.127639903814849
	flickr  1.1198145166662379
	my      1.1125213438098644
	us      1.11074443481126
	art     1.0884187202677287
	but     1.075000584893044
	on      1.0716979780709746
	ha      1.0648387837662703
	bridg   1.0646040788654794
	la      1.0566267300421526
	view    1.0531003670595371
	orlean  1.0513038897754718
	time    1.0436137681325373
	world   1.0354997179539986
```

How are these results different from those using the `analysis.RegexTokenizer()`?
```
	The stemming analyzer has both a lower case and stop filter built into it. So its very similar to the
	`analysis.RegexTokenizer() | analysis.LowercaseFilter() | analysis.StopFilter()` But is also has stemming,
	which attempts to remove suffixes and sometimes prefixes of words to group them together. So for example 
	photos is reduceds to photo, and so on.
```

Which top terms seem to be the most descriptive of the content of the web pages?  Why?
```
	Currently it appears the third set of terms seems to be the best. It has much less unimportant words like 'the'
	and 'of', and while stemming is good in theory, the filter as applied also gave strang results, like changing
	'bridge' to 'bridg' and 'orleans' to 'orlean' which does affect their meaning.
```

Use the `analysis.RegexTokenizer() | analysis.LowercaseFilter() | analysis.StopFilter()` tokenizer for subsequent questions.


### Investigate the top term "de"

List the tokenizers under which "de" is a top 20 term.
	analysis.RegexTokenizer() | analysis.LowercaseFilter() | analysis.StopFilter()
	analysis.StemmingAnalyzer()
	

One of the top terms is "de".  List the URLs of the documents in which this term occurs (by running a query for it, see [Line 82 of runWhoosh.py](https://github.com/cisc7610/homework4/blob/master/runWhoosh.py#L82)).
```
	Url: www.hdfondos.eu\imagen\223881\wtc-world-trade-center-ciudades-de-rascacielos-de-la-ciudad-de-nueva-york-edificio
	Url: www.flickr.com\photos\tags\new york pennsilvania station\
	Url: www.actualitix.com\villes-les-plus-visitees-au-monde.txt
	Url: nayruscatwalk.blogspot.com\2013\02\diario-de-nueva-york-day-6-tiendas.txt
	Url: generationvoyage.fr\loger-new-york\
	Url: neoyorkinos.com\2015\12\31\buenos-dias-nyc-un-feliz-2016\
	Url: www.flickr.com\photos\tags\Leon Moisseiff\
	Url: www.meetup.com\es\tokyojazz\members\156441832\
	Url: aroundguides.com\fr\22570354
	Url: mapcarta.com\es\25031690
	Url: www.tumblr.com\tagged\wcl-x100
	Url: noleggiocamper.rent.it\noleggiocamper\stati_uniti_d_america-253\new_york-1243\
	Url: plus.google.com\111165959052757347968
	Url: www.flickr.com\people\anieto2k\
	Url: www.wordnik.com\words\graphy
	Url: flickriver-lb-1710691658.us-east-1.elb.amazonaws.com\groups\allpeople\pool\random\
	Url: www.flickr.com\photos\tags\Port Authority of New York and New Jersey\
	Url: www.flickr.com\photos\tags\fuji wcl-x100\
	Url: www.flickr.com\photos\tags\Statue\
	Url: flickriver.com\groups\614844@N20\pool\interesting\
	Url: www.depositagift.com\_\blog\page\12\
	Url: www.flickr.com\photos\tags\daniel walker\
	Url: www.flickr.com\photos\tags\New_York
	Url: jenare.com\beautiful-churches-cathedrals-around-world\
	Url: mashable.com\2015\03\21\new-york-city-apps-2\
	Url: mail.asadal.com\compds_file\tumblr-fisheye
	Url: www.flickr.com\photos\tags\Michael S. Walker\
	Url: www.mappingmegan.com\things-you-should-know-about-staying-in-new-york-city-hotels\
	Url: www.flickr.com\photos\tags\disregarded
	Url: www.redhotsunglasses.co.uk\blog\fashion-lovers-guide-new-york\
	Url: nabewise.com\nyc\city-island\
	Url: www.flickr.com\photos\tags\ataferner
	Url: www.flickr.com\photos\tags\axeltaferner
	Url: www.localdatabase.com\newyork\New_York
	Url: www.flickr.com\photos\tags\statue_of_liberty\
	Url: www.timeout.com\newyork\attractions\new-york-attractions
```

What do these pages have in common?
	They mostly appear to be in spanish, or at least another non english language.

### Compare different weightings

For this part, we will use the two queries, "new york" and "empire state building".

List the number of documents returned for each query and the URLs of the top 20 results for each query using the default weighting, `scoring.BM25F()`.
```
	************************************************************
	Query:   new york   returned 196 results
	************************************************************
	Url: indulgy.com\post\9oS6UVpli2\empire-state-building-at-night-new-york-city
	Url: www.flickr.com\photos\tags\New_York
	Url: indulgy.com\post\nGdsQo4VJ2\george-rodger-new-york-city-the-empire-stat
	Url: www.etsy.com\listing\200815622\nyc-photography-coney-island-boardwalk
	Url: www.localdatabase.com\newyork\New_York
	Url: www.nyny.com\neighborhoods\manhattan
	Url: www.desktopimages.org\wallpaper\223881\wtc-world-trade-center-skyscraper-city-cities-building-new-york
	Url: lv.advisor.travel\city\Nujorka-2306488\photos
	Url: susangospelmusic.blogspot.com\2015\09\autumn-in-new-york.txt
	Url: generationvoyage.fr\loger-new-york\
	Url: www.nyny.com\neighborhoods\dumbo
	Url: flipboard.com\@shermar54\nyc-5j29vomfz
	Url: mapcarta.com\22602858
	Url: www.businessinsider.com\hacker-takes-down-new-york-magazine-due-to-nyc-grudge-2015-7
	Url: www.rent.it\autonoleggio\stati_uniti_d_america-253\new_york-1243\
	Url: www.etsy.com\shop\CityThatNeverSleeps
	Url: www.npr.org\sections\alltechconsidered\2014\10\16\356728035\airbnb-new-york-state-spar-over-legality-of-rentals
	Url: triptaptoe.com\tour\new-york-breaks\
	Url: www.onlyinyourstate.com\new-york\things-to-avoid-in-nyc\
	Url: triptaptoe.com\tour\new-york-vacation\


	************************************************************
	Query:   empire state building   returned 47 results
	************************************************************
	Url: indulgy.com\post\9oS6UVpli2\empire-state-building-at-night-new-york-city
	Url: indulgy.com\post\nGdsQo4VJ2\george-rodger-new-york-city-the-empire-stat
	Url: www.flickr.com\photos\danielmennerich\5443157673
	Url: www.flickr.com\photos\danielmennerich\5205390702
	Url: www.haikudeck.com\copy-of-richardson-stern-2-uncategorized-presentation-AUxDx8RhJJ
	Url: www.linkedin.com\pulse\top-10-things-do-new-york-kate-moswood
	Url: www.flyfar.ca\blog\things-to-do-in-new-york\
	Url: elevation.maplogs.com\poi\hillcrest_queens_ny_usa.36360.txt
	Url: elevation.maplogs.com\poi\riverside_dr_new_york_ny_usa.231154.txt
	Url: elevation.maplogs.com\poi\e_14th_st_new_york_ny_usa.231190.txt
	Url: www.flickr.com\photos\25582125@N04\36344754535
	Url: www.flickr.com\photos\andreas_komodromos\36343385791
	Url: www.haikudeck.com\richardson-stern-2-uncategorized-presentation-7Nlp3sq4Bq
	Url: www.flickr.com\photos\nikiteenrico\galleries\72157645655220179\
	Url: www.thehotelguru.com\best-hotels-in\united-states-of-america\new-york
	Url: www.flickr.com\photos\tags\Port Authority of New York and New Jersey\
	Url: www.myadventuresacrosstheworld.com\things-to-do-in-new-york\
	Url: www.flickr.com\photos\hickatee\34643516810
	Url: www.flickr.com\photos\hickatee\35024953080
	Url: mapcarta.com\25005378
```

List the number of documents returned for each query and the URLs of the top 20 results for each query using the weightings:
1. `scoring.TF_IDF()`
```
	************************************************************
	Query:   new york   returned 196 results
	************************************************************
	Url: www.localdatabase.com\newyork\New_York
	Url: www.flickr.com\photos\tags\Port Authority of New York and New Jersey\
	Url: www.travelalphas.com\new-york-city-bucket-list-things-to-do\
	Url: www.flickr.com\photos\tags\daniel walker\
	Url: www.etsy.com\listing\200815622\nyc-photography-coney-island-boardwalk
	Url: www.flickr.com\photos\tags\New_York
	Url: generationvoyage.fr\loger-new-york\
	Url: www.flickr.com\photos\tags\Leon Moisseiff\
	Url: www.thehotelguru.com\best-hotels-in\united-states-of-america\new-york
	Url: www.flickr.com\photos\tags\statue_of_liberty\
	Url: www.flickr.com\photos\tags\Michael S. Walker\
	Url: www.travelwitharchie.com\destinations-usa-nyc-10-reasons-love-new-york\
	Url: www.etsy.com\shop\CityThatNeverSleeps
	Url: www.timeout.com\newyork\attractions\new-york-attractions
	Url: www.myadventuresacrosstheworld.com\things-to-do-in-new-york\
	Url: indulgy.com\post\9oS6UVpli2\empire-state-building-at-night-new-york-city
	Url: www.nyny.com\neighborhoods\manhattan
	Url: indulgy.com\post\nGdsQo4VJ2\george-rodger-new-york-city-the-empire-stat
	Url: www.rent.it\autonoleggio\stati_uniti_d_america-253\new_york-1243\
	Url: dopesontheroad.com\travel\lesbian-travel-guide-new-york-city\



	************************************************************
	Query:   empire state building   returned 47 results
	************************************************************
	Url: www.flickr.com\photos\tags\Port Authority of New York and New Jersey\
	Url: indulgy.com\post\9oS6UVpli2\empire-state-building-at-night-new-york-city
	Url: indulgy.com\post\nGdsQo4VJ2\george-rodger-new-york-city-the-empire-stat
	Url: www.thehotelguru.com\best-hotels-in\united-states-of-america\new-york
	Url: www.flickr.com\photos\tags\daniel walker\
	Url: www.timeout.com\newyork\attractions\new-york-attractions
	Url: www.flickr.com\photos\tags\Leon Moisseiff\
	Url: www.travelalphas.com\new-york-city-bucket-list-things-to-do\
	Url: www.myadventuresacrosstheworld.com\things-to-do-in-new-york\
	Url: www.flyfar.ca\blog\things-to-do-in-new-york\
	Url: www.linkedin.com\pulse\top-10-things-do-new-york-kate-moswood
	Url: generationvoyage.fr\loger-new-york\
	Url: www.flickr.com\photos\tags\New_York
	Url: www.haikudeck.com\copy-of-richardson-stern-2-uncategorized-presentation-AUxDx8RhJJ
	Url: www.haikudeck.com\richardson-stern-2-uncategorized-presentation-7Nlp3sq4Bq
	Url: www.etsy.com\listing\200815622\nyc-photography-coney-island-boardwalk
	Url: elevation.maplogs.com\poi\hillcrest_queens_ny_usa.36360.txt
	Url: www.seduniatravel.com\tours\americas\usa\eastern-highlights
	Url: elevation.maplogs.com\poi\e_14th_st_new_york_ny_usa.231190.txt
	Url: elevation.maplogs.com\poi\riverside_dr_new_york_ny_usa.231154.txt
```

2. `scoring.Frequency()`
```
	************************************************************
	Query:   new york   returned 196 results
	************************************************************
	Url: www.localdatabase.com\newyork\New_York
	Url: www.flickr.com\photos\tags\Port Authority of New York and New Jersey\
	Url: www.travelalphas.com\new-york-city-bucket-list-things-to-do\
	Url: www.flickr.com\photos\tags\daniel walker\
	Url: www.etsy.com\listing\200815622\nyc-photography-coney-island-boardwalk
	Url: www.flickr.com\photos\tags\New_York
	Url: generationvoyage.fr\loger-new-york\
	Url: www.flickr.com\photos\tags\Leon Moisseiff\
	Url: www.thehotelguru.com\best-hotels-in\united-states-of-america\new-york
	Url: www.flickr.com\photos\tags\statue_of_liberty\
	Url: www.flickr.com\photos\tags\Michael S. Walker\
	Url: www.travelwitharchie.com\destinations-usa-nyc-10-reasons-love-new-york\
	Url: www.etsy.com\shop\CityThatNeverSleeps
	Url: www.timeout.com\newyork\attractions\new-york-attractions
	Url: www.myadventuresacrosstheworld.com\things-to-do-in-new-york\
	Url: indulgy.com\post\9oS6UVpli2\empire-state-building-at-night-new-york-city
	Url: www.nyny.com\neighborhoods\manhattan
	Url: dopesontheroad.com\travel\lesbian-travel-guide-new-york-city\
	Url: indulgy.com\post\nGdsQo4VJ2\george-rodger-new-york-city-the-empire-stat
	Url: www.rent.it\autonoleggio\stati_uniti_d_america-253\new_york-1243\


	************************************************************
	Query:   empire state building   returned 47 results
	************************************************************
	Url: www.flickr.com\photos\tags\Port Authority of New York and New Jersey\
	Url: indulgy.com\post\9oS6UVpli2\empire-state-building-at-night-new-york-city
	Url: indulgy.com\post\nGdsQo4VJ2\george-rodger-new-york-city-the-empire-stat
	Url: www.flickr.com\photos\tags\daniel walker\
	Url: www.thehotelguru.com\best-hotels-in\united-states-of-america\new-york
	Url: www.timeout.com\newyork\attractions\new-york-attractions
	Url: www.flickr.com\photos\tags\Leon Moisseiff\
	Url: www.travelalphas.com\new-york-city-bucket-list-things-to-do\
	Url: www.myadventuresacrosstheworld.com\things-to-do-in-new-york\
	Url: www.flyfar.ca\blog\things-to-do-in-new-york\
	Url: www.linkedin.com\pulse\top-10-things-do-new-york-kate-moswood
	Url: generationvoyage.fr\loger-new-york\
	Url: www.flickr.com\photos\tags\New_York
	Url: www.etsy.com\listing\200815622\nyc-photography-coney-island-boardwalk
	Url: www.haikudeck.com\copy-of-richardson-stern-2-uncategorized-presentation-AUxDx8RhJJ
	Url: www.haikudeck.com\richardson-stern-2-uncategorized-presentation-7Nlp3sq4Bq
	Url: elevation.maplogs.com\poi\hillcrest_queens_ny_usa.36360.txt
	Url: www.seduniatravel.com\tours\americas\usa\eastern-highlights
	Url: elevation.maplogs.com\poi\e_14th_st_new_york_ny_usa.231190.txt
	Url: elevation.maplogs.com\poi\riverside_dr_new_york_ny_usa.231154.txt
```

List the URLs that are present for all weightings.
```
	************************************************************
	Query:   new york
	************************************************************
	Url: indulgy.com\post\9oS6UVpli2\empire-state-building-at-night-new-york-city
	Url: www.flickr.com\photos\tags\New_York
	Url: indulgy.com\post\nGdsQo4VJ2\george-rodger-new-york-city-the-empire-stat
	Url: www.etsy.com\listing\200815622\nyc-photography-coney-island-boardwalk
	Url: www.localdatabase.com\newyork\New_York
	Url: www.nyny.com\neighborhoods\manhattan
	Url: generationvoyage.fr\loger-new-york\


	************************************************************
	Query:   empire state building
	************************************************************
	Url: indulgy.com\post\9oS6UVpli2\empire-state-building-at-night-new-york-city
	Url: indulgy.com\post\nGdsQo4VJ2\george-rodger-new-york-city-the-empire-stat
	Url: www.haikudeck.com\copy-of-richardson-stern-2-uncategorized-presentation-AUxDx8RhJJ
	Url: www.linkedin.com\pulse\top-10-things-do-new-york-kate-moswood
	Url: www.flyfar.ca\blog\things-to-do-in-new-york\
	Url: elevation.maplogs.com\poi\hillcrest_queens_ny_usa.36360.txt
	Url: elevation.maplogs.com\poi\riverside_dr_new_york_ny_usa.231154.txt
	Url: elevation.maplogs.com\poi\e_14th_st_new_york_ny_usa.231190.txt
	Url: www.haikudeck.com\richardson-stern-2-uncategorized-presentation-7Nlp3sq4Bq
	Url: www.thehotelguru.com\best-hotels-in\united-states-of-america\new-york
	Url: www.flickr.com\photos\tags\Port Authority of New York and New Jersey\
	Url: www.myadventuresacrosstheworld.com\things-to-do-in-new-york\
```

Which URLs seem to be the most relevant to the queries?  Why?
```
	The URLs from BM25F appear most relevant, as the other ones produced a lot more photo, while this seems to have returned more informative results.
```


Use the `scoring.BM25F()` weighting for subsequent questions.


### Measure the precision of several queries

Recall that *precision* is a measure of retrieval performance calculated as the proportion of correct results returned to the total number of results returned.  Normally the correct results are determined by hand, but in this case we will use a more specific query as the target correct results.

Measure the precision of the top 10 results returned by the query "empire state" with respect to retrieving the results of the query "empire state building".  This means that correct results are those that are returned by the query "empire state building" and we are measuring the performance of the query "empire state" in retrieving those pages.  Because the *highlights* might be different between the two queries, make sure to compare URLs, which are unique.
	```0.8```

Measure the precision of the top 20 results returned by the query "empire state" with respect to retrieving the results of the query "empire state building".
	```0.9````

Measure the precision of the top 20 results returned by the query "new york" with respect to retrieving the results of the query "new york city".
	```0.6```

### Describe any problems that you ran into in the course of this project

Describe any problems that you ran into in the course of this project
```
	As far as I can tell, a lot of the data had symbols in its names that couldnt be download onto windows with get.
	For example '?=' in urls are invalid in file names. So I think I got all the data downloaded and working, but I
	Cant be sure.
```
