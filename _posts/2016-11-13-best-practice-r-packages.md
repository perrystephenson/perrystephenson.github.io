---
layout:     post
title:      "Best Practice for R Packages"
subtitle:   "Things I learned when I submitted to rOpenSci"
date:       2016-11-13 19:30:00
author:     "Perry Stephenson"
header-img: "img/post-bg-01.jpg"
---

_I was pretty pleased with myself when I got my [first package](https://cran.r-project.org/package=refimpact) accepted to CRAN. It wasn't entirely smooth sailing (two rejections due to typos) but overall I came away with a smug feeling, thinking "yeah I nailed that". Then I submitted to rOpenSci, and learned a whole bunch of things about best practice R programming._


## Things I should have done properly the first time

Obviously I should have just done *everything* better the first time, but there are some things that stand out as things I definitely should have done.

### Good Practice
I tried to make sure I was following the style suggested by Hadley Wickham in the [R Packages](http://r-pkgs.had.co.nz/style.html) book, and thought I'd done a pretty good job. I passed all of the CRAN checks and had 98% unit test coverage, so I thought I had pretty much nailed it on the coding style. It turns out there was a simple package I could have been using to check my coding style as I went: [**goodpractice**](https://github.com/MangoTheCat/goodpractice). The `goodpractice::gp()` function checks for missing unit tests, accidental use of the `=` operator (instead of `<-`), tidy code formatting, R CMD check errors you might have missed, and probably a whole bunch of other stuff too. What a tool! 

Running this function would have given me a few bonus style points for very little effort.

### Vignette

A vignette isn't compulsory for packages on CRAN, however it is a key tool for prospective users hoping to understand what your package does, what the key functions are, and how they should structure their workflow. Given that this package is an API wrapper, it would have been fairly simple to write a vignette that shows the user how to download data that they're interested in, and how to do some basic text analysis using the dataset.

### A 'help' index

The API isn't exactly clearly documented, and neither is my package. All of the functions have help documentation, however I did not create an index and there is no easy way for a new user to discover the functionality within the package and the data available through the API. Producing a 'help' index would go some way to filling this gap, along with the vignette mentioned above.

## Things I learned by submitting to rOpenSci

This was actually a really great experience. Two reviewers spent a total of six hours reviewing all of the code and documentation in my package, and gave me some really great feedback (which is available [here](https://github.com/ropensci/onboarding/issues/78)). I doubt the markers for my undergraduate (honours) thesis spent this long reviewing my work, let alone any of the other assessments I've submitted to various institutions over the years. These two guys just gave me six hours of their time, completely free, and gave me some really detailed and specific feedback. I highly recommend the process!

...to be continued when I have a bit more time...

