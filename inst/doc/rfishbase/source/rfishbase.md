`ro warning=FALSE, message=FALSE, comment=NA, tidy=FALSE, cache=TRUE or`

% Rfishbase
% Carl Boettiger[^\*][^a] and Peter Wainwright[^a]
% 

[^a]: Center for Population Biology, University of California, Davis, United  States
  

# Abstract 
We introduce a package that provides interactive and programmatic
access to the FishBase repository. This package allows us to
interact with data on over 30,000 fish species in the rich
statistical computing environment, `R`. We illustrate how this
direct, scriptable interface to FishBase data enables better
discovery and integration essential for large-scale comparative
analyses. We provide several examples to illustrate how the package
works, and how it can be integrated into such as phylogenetics
packages `ape` and `geiger`.


###### keywords 
R | vignette | fishbase

# Introduction


``` {r libraries, echo=FALSE }
library(rfishbase) 
library(xtable) 
library(ggplot2)
data(fishbase)
````


Informatics
Stuff about Machine access to data, role of large scale data in ecology [@jones2006d], [@hanson2011a], [@reichman2011a]

FishBase ([fishbase.org](http://fishbase.org)) is an award-winning
online database of information about the morphology, trophic ecology,
physiology, ecotoxicology, reproduction, economic relevance of the
world’s fish, organized by species [@fishbase2012]. FishBase was developed in
collaboration with the United Nations Food and Agriculture Organization
and is supported by a consortium of nine research institutions. In
addition to its web-based interface, FishBase provides machine readable
XML files for `ri I(sprintf("%d", length(fish.data))) ir` of its species entries.

To facilitate the extraction, visualization, and integration of this
data in research, we have written the `rfishbase` package for the R
language for statistical computing and graphics [@rteam2012]. R is a freely
available open source computing environment that is used extensively in
ecological research, with a large library of packages built explicitly
for this purpose [@kneib2007].  

In this paper, we illustrate how the `rfishbase` package is synchronized 
with the FishBase database, describe its functions for extracting, 
manipulating and visualizing data, and then illustrate how these 
functions can be combined for more complicated analyses.  Lastly we 
illustrate how having access to FishBase data through R allows 
a user to interface with other kinds of analyses, such as 
comparative phylogenetics software.  


# A programmatic interface

The `rfishbase` package works by creating a cached copy of all data on
FishBase currently available in XML format. Caching increases the speed
of queries and places minimal demands on the fishbase server, which in
it’s present form is not built to support direct access to application
programming interfaces (APIs). A cached copy is included in the package 
and can be loaded in to R using the command:

``` {r loaddata }
data(fishbase) 
````

This loads a copy of all available data from FishBase into the R list, 
`fish.data`, which can be passed to the various functions of `rfishbase`
to extraction, manipulation and visualization.  The online repository
is frequently updated as new information is uploaded.  To get the most 
recent copy of fishbase, update the cache instead. The
update may take up to 24 hours. This copy is stored in the working
directory with the current date and can be loaded when finished.

``` {r update, eval=FALSE }
updateCache()
loadCache("2012-03-26fishdata.Rdat")
````
Loading the database creates an object called fish.data, with one entry
per fish species for which data was successfully found, for a total of
`ri I(sprintf("%d", length(fish.data))) ir` species.  

Not all the data available in FishBase is included in these machine-readable 
XML files.  Consequently, `rfishbase` returns taxonomic information, trophic
discription, habitat, distribution, size, lifecycle, morphology and diagnostic
information.  The information returned in each category is provided as plain-text,
consequently `rfishbase` must use regular expression matching to identify
the occurence of particular words or patterns in this text corresponding to 
data of interest. 

Quantitative traits such as length, maximum known age, spine and ray counts,
and depth information are provided consistently for most species, allowing 
`rfishbase` to extract this data directly.  Other queries require pattern matching.
While simple text searches within a given field are usually reliable, the 
`rfishbase` search functions will take any regular expression query, which
permits logical matching, identification of number strings, and much more.
The interested user should consult a reference on regular expressions after
studying the simple examples provided here to learn more.  

# Data extraction, analysis, and visualization

The basic function data extraction function in `rfishbase` is `which_fish()`. 
The function takes a list of fishbase data (usually the entire database, `fish.data`,
or a subset thereof, as illustrated later) and returns an array of those
species matching the query.  This array is given as a list of true/false values
for every species in the query.  This return structure has several advantages which
we illustrate by example.  

Here is a query for reef-associated fish (mention of "reef" in the habitat description),
and second query for fish that have "nocturnal" in their trophic description:

``` {r nocturnal }
reef <- which_fish("reef", "habitat", fish.data)
nocturnal <- which_fish("nocturnal", "trophic", fish.data)
````
One way these returned values are commonly used is to obtain a subset of the 
database that meets this criteria, which can then be passed on to other functions.
For instance, if we want the scientific names of these reef fish, we use the 
`fish_names` function.  Like the `which_fish` function, it takes the database 
of fish, `fish.data` as input.  In this case, we pass it just the subset that are 
reef affiliated,

``` {r }
reef_species <- fish_names(fish.data[reef])
````

Because our `reef` object is a list of logical values (true/false), we can
combine this in intuitive ways with other queries.  For instance, give us the 
names for all fish that are nocturnal and not reef associated,

``` {r }
nocturnal_nonreef_orders <- fish_names(fish.data[nocturnal & !reef], "Class")
````

Note that this time we have also specified that we want taxanomic Class
of the fish matching the query, rather than the species names.  `fish_names`
will allow us to specify any taxanomic level for it to return.

Quantitative trait queries work like `fish_names`, taking a the FishBase 
data and returning the requested information.  For instance, the function
`getSize` returns the length (default), weight, or age of the fish in the query:

``` {r AgeHist }
age <- getSize(fish.data, "age")
````


The real power of programatic access the ease with which we can combine,
visualize, and statistically test a custom compilation of this data. 
To do so it is useful to organize a collection of queries into a data frame.
Here we combine the queries we have made above and a few additional queries
into a data frame in which each row represents a species and each column
represents a variable.  


``` {r fishdataframe }
marine <- which_fish("marine", "habitat", fish.data)
africa <- which_fish("Africa:", "distribution", fish.data)
length <- getSize(fish.data, "length")
order <- fish_names(fish.data, "Order")
dat <- data.frame(reef, nocturnal,  age, marine, africa, length, order)
````

This data frame contains categorical data (*e.g.* is the fish a
carnivore) and continuous data (*e.g.* weight or age of fish). We can
take advantage of the rich data visualization in R to begin exploring
this data.  These examples are simply meant to be illustrative of the 
kinds of analysis possible and how they would be constructed.  

Fraction of marine species found in the 10 largest orders

``` {r fraction_marine, fig.width=10}
biggest <- names(head(sort(table(order),decr=T), 10))
ggplot(subset(dat,order %in% biggest), aes(order, fill=marine)) + geom_bar() 
````

Is age correlated with size?

``` {r Figure1, fig.width=10}
ggplot(dat,aes(age, length, color=marine)) + geom_point(position='jitter',alpha=.8) + scale_y_log10() + scale_x_log10() 
````

A statistical test of this pattern: 

``` {r Table1, results="asis"}
xtable(summary(lm(data=dat,  log(length) ~ log(age)) ))
````


Are reef species longer lived than non-reef species in the marine environment?

``` {r   }
ggplot(subset(dat, marine),aes(reef, log(age))) + geom_boxplot() 
````

## Comparative studies


For instance, we can ask "are there more labrids or goby species of reef
fish?" using the following queries:

Get all species in fishbase from the families "Labridae" (wrasses) or
"Scaridae" (parrotfishes):

``` {r labrid }
labrid <- familySearch("(Labridae|Scaridae)", fish.data)
````

and get all the species of gobies

``` {r gobycoount }
goby <- familySearch("Gobiidae", fish.data)
````

Identify how many labrids are found on reefs

``` {r labridreef }
labrid.reef <- habitatSearch("reef", fish.data[labrid])
nlabrids <- sum(labrid.reef)
````

and how many gobies are found on reefs:

``` {r gobyreef }
ngobies <- sum (habitatSearch("reef", fish.data[goby]) )
````

showing us that there are `ri I(nlabrids) ir` labrid species associated with reefs,
and `ri I(ngobies) ir` goby species associated with reefs.  




Note that any function can take a subset of the data, as specified by
the square brackets.

# Integration of analyses

One of the greatest advantages about accessing FishBase directly through
R is the ability to take advantage of the suite of specialized analyses
available through R packages. Likewise, users familiar with these
packages can more easily take advantage of the data available on
fishbase. We illustrate this with an example that combines phylogenetic
methods available in R with quantitative trait data available from
`rfishbase`.

This series of commands illustrates testing for a phylogenetically
corrected correlation between the maximum observed size of a species and
the maximum observed depth at which it is found.


load a phylogenetic tree and some phylogenetics packages
``` {r results="hide", message=FALSE }
data(labridtree)
require(geiger) 
````

 Find those species on FishBase 
``` {r  }
myfish <- findSpecies(tree$tip.label, fish.data)
````

Get the maxium depth of each species and sizes of each species: 
``` {r   }
depths <- getDepth(fish.data[myfish])[,"deep"]
size <- getSize(fish.data[myfish], "length")
````

Drop tips from the phylogeny for unmatched species.  
``` {r   }
data <- na.omit(data.frame(size,depths))
pruned <- treedata(tree, data)
````


Use phylogenetically independent contrasts [@felsenstein1985] to determine if depth correlates with size after correcting for phylogeny:

``` {r results="asis" }
x <- pic(pruned$data[["size"]],pruned$phy)
y <- pic(pruned$data[["depths"]],pruned$phy)
xtable(summary(lm(y ~ x - 1)))
ggplot(data.frame(x=x,y=y), aes(x,y)) + geom_point() + stat_smooth(method=lm)
````

We can also estimate different evolutionary models for these traits to decide which best describes the data,

``` {r models, results="hide" }
bm <- fitContinuous(pruned$phy, pruned$data[["depths"]], model="BM")[[1]]
ou <- fitContinuous(pruned$phy, pruned$data[["depths"]], model="OU")[[1]]
````

where the Brownian motion model has an AIC score of `ri I(bm$aic) ir` while
the OU model has a score
of `ri I(ou$aic) ir`, suggesting that `ri I(names(which.min(list(BM=bm$aic,OU=ou$aic)))) ir` is the better model.

In a similar fashion, programmers of other R software packages can make
use of the rfishbase package to make this data available to their
functions, further increasing the use and impact of fishbase. For
instance, the project OpenFisheries.org makes use of the fishbase
package to provide information about commercially relevant species.

# Discussion

## The self-updating study

Describe how this package could help make studies that could be
automatically updated as the dataset is improved and expanded (like the
examples in this document which are automatically run when the pdf is
created). 
[@peng2011b; @merali2010].


## Limitations and future directions

Fishbase contains much additional data that has not been made accessible
in it’s machine-readable XML format. We are in contact with the database
managers and look forward to providing access to additional types of
data as they become available.

# Acknowledgements

CB is supported by a Computational Sciences Graduate Fellowship from the
Department of Energy under grant number DE-FG02-97ER25308.


[^\*]: Corresponding author <cboettig@ucdavis.edu> 


# References
