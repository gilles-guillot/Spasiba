---
title: "SPASIBA package documentation"
author: G. Guillot, H. Jonsson, A. Hinge,  N. Manchih, L. Orlando
output:  
html_document:
    toc: yes
    number_sections: yes
    theme: unite
date: "07/07/2015"
---





## Overview
This page provides information about the computer program SPASIBA, an R package for spatial  continuous assignment from genetic data.  
SPASIBA provides functions to perform the following tasks:

* Simulating data from a geostatistical model (function `SPASIBA.sim`)

* Inferring parameters of a covariance function model of spatial genetic variation (function `SPASIBA.inf`)

* Performing spatial prediction of allele frequencies (function `SPASIBA.inf`)

* Performing spatial assignment of individuals of unknown geographic origin (function `SPASIBA.inf`)


--------------



## Installation
To run SPASIBA you need to have different things installed first: R, the R packages `INLA`, `RandomFields` and the R package `SPASIBA` itself. 


- To instal R, follow instructions from [CRAN](http://cran.r-project.org/).

- To instal INLA, follow instructions from the [R-INLA project homepage](http://www.r-inla.org/download). 

- To install packages `RandomFields`, type in the R prompt 

```
install.packages('RandomFields')
``` 

- To install SPASIBA: 

```
devtools::install_github('gilles-guillot/SPASIBA',force=TRUE,build_vignette=TRUE)
```

You can check that `SPASIBA` has been installed correctly by trying to load it:

```
library(SPASIBA)
``` 


--------------

## Input and output

### Input data

To use `SPASIBA`, you need to have four data matrices under your R session:

* A matrix of allele counts for the various reference (or training) populations. One row per population, one column per locus.
Missing data not allowed. 

* A matrix with one row per population and one column
    per locus giving haploid population sample size. 
    Missing data not allowed. 
    Missingness of genotypes in the reference population is handled in this way: an individual with SNP genotype {0,1} will 
    resut in allele counts {0,1} and haploid population size 2. An individual with SNP genotype {NA,1} will 
    resut in allele counts {0,1} and haploid population size 1.
    
    
* A matrix containing coordinates of reference sampling sites. One row per
    sampling site, two columns (xy cartesian coordinates or lon-lat). Missing data not allowed. 
    
*  A matrix of genotypes  of individuals of unknwon geographic
    origin. One row per indivdual, one column per locus. This should
    contain allele counts of an arbitrary reference allele at each SNP locus, 
    hence 0,1, 2 or NA (missing data are allowed here). 
    
Assuming these matrices exist somewhere as plain text files on your disk, you can read them from R with the `read.table` function. 
 If you have doubt about the format of the data, you can open the various files on the SPASIBA homepage [data folder](http://www2.imm.dtu.dk/~gigu/Spasiba/data). 
 See below for an example. 





### Output
The main function `SPASIBA.inf` return various objects stacked in a list. This includes a matrix of estimated coordinates for individuals of unknown geographic origin. 


-------------------------------------

## On-line documentation

Besides the present web page, users can find information about the various functions from the R on-line help,
```
?SPASIBA.inf
```

-----------------------------------


## Example 

### Reading data from external files
In the example below, the data are stored on a folder on the SPASIBA homepage. It can be also a folder on your local computer or anywhere else onthe web.

```
## reading coordinates of reference populations
coord.ref = read.table('http://www2.imm.dtu.dk/~gigu/Spasiba/data/coord.ref.txt')

# reading allele counts  of reference populations
geno.ref = read.table('http://www2.imm.dtu.dk/~gigu/Spasiba/data/geno.ref.txt')
geno.ref = as.matrix(geno.ref) 

# reading haploid reference population sizes 
size.pop.ref = read.table('http://www2.imm.dtu.dk/~gigu/Spasiba/data/size.pop.ref.txt')
size.pop.ref = as.matrix(size.pop.ref)

## reading genotypes of individuals of unknown geographic origin
geno.unknown = read.table('http://www2.imm.dtu.dk/~gigu/Spasiba/data/geno.unknown.txt')
geno.unknown = as.matrix(geno.unknown)

## reading true coordinates of individuals  assumed here to be of unknown geographic origin
## if you have such a file you don't need the SPASIBA program!
true.coord.unknown = read.table('http://www2.imm.dtu.dk/~gigu/Spasiba/data/true.coord.unknown.txt')
```


You can check that the various data matrices have been loaded properly with `head` function, e.g.
```
head(coord.ref[1:10,]) ## here inspecting 10 first lines only
```

### Making computations 

```
## loading the packages
require(INLA)
require(SPASIBA)
## Calling SPASIBA function for inference, prediction and assignment
res <- SPASIBA.inf(geno.ref=geno.ref,
                           ploidy=2,
                           coord.ref=coord.ref,
                           sphere=FALSE, 
                           size.pop.ref=size.pop.ref,
                           geno.unknown=geno.unknown,
                           make.inf=TRUE,
                           loc.infcov = 1:30,
                           make.pred=TRUE,
                           make.assign=TRUE)
````

The R object returned and stored in `res` by the code above is a list (an object consisting of several objects). 
The estimated coordinates of samples of unknown geographic origins is named  `coord.unknown.est`. It can be accessed 
as `res$coord.unknown.est` and for example plotted together with sampling sites by 

```
plot(res$coord.unknown.est,pch=3,col=3,cex=1.3,lwd=2,xlab='',ylab='',asp=1,
     axes=TRUE,ylim=c(0,1.2))
legend(col=c(3,2,4),pch=c(3,1,1),#cex=c(1.3,1,1.5),
       legend=c('estimate','ref pops','true'),
       x=.8,y=1.2,border=FALSE)
points(coord.ref,col=2,pch=1,cex=1)
points(true.coord.unknown,col=4,cex=1.5,lwd=2)
arrows(x0=true.coord.unknown[,1],
       y0=true.coord.unknown[,2],
       x1=res$coord.unknown.est[,1],
       y1=res$coord.unknown.est[,2],
       code=2,length=0.1,angle=10,lwd=.3)
```

## Making maps 
See [post on the Molecular Ecologist](http://www.molecularecologist.com/2012/09/making-maps-with-r) for inspiration.

## References
The model and algorithm underlying the SPASIBA program are described in 

* Guillot, Gilles, et al. "Accurate continuous geographic assignment from low-to high-density SNP data." Bioinformatics 32.7 (2015): 1106-1108.

 

 


--------------

