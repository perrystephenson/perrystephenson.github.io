---
layout:     post
title:      "Best Practice for R Packages"
subtitle:   "Things I learned when I submitted to rOpenSci"
date:       2016-11-13 19:30:00
author:     "Perry Stephenson"
header-img: "img/post-bg-01.jpg"
---

_I was pretty pleased with myself when I got my [first package](https://cran.r-project.org/package=refimpact) accepted to CRAN. It wasn't entirely smooth sailing (two rejections due to typos) but overall I came away with a smug feeling, thinking "yeah I nailed that". Then I submitted to rOpenSci, and learned a whole bunch of things about best practice R programming._

_I won't bother going into too much detail about the package itself (check it out at the link above!) except to say that it's a wrapper function for an API._


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

### Documentation

One thing that stuck out in a few of the different pieces of feedback is the importance of documentation. I made the decision very early on to make an API wrapper, rather than utilising the permissive licence attached to the dataset and bundling the dataset inside an R package. There are perfectly sensible reasons for doing this, and the reviewers ended up agreeing with me, but looking at the package from their perspective this must have taken them quite some time to figure out why I'd made the decision. 

I recall one of my professors at UNSW telling us that we should always answer questions as soon as we think the reader will ask them (to ensure that the reader doesn't get distracted trying to find the answer); I think that the same thing goes for package development. If a reviewer or a user of your package would naturally ask a question when they're taking a look at it for the first time, then you should document the answer to that question in the first place they'll look for it. In practice this means verbose vignettes and detailed help documentation. It also means thorough coverage of code comments, for people digging a bit deeper.

Finally, whilst I always knew that R Markdown was awesome, it didn't really occur to me to use R Markdown to generate the README.md file. This is totally worth doing, and it takes no time at all to set up.

### APIs

Since going through the feedback I've learned a lot about APIs. Most data scientists will come across the need to interface through APIs sooner or later, so it makes sense to learn how to do it properly.

Firstly, the best R package for this is the [httr package](https://CRAN.R-project.org/package=httr). Using this package helps you build the API query, and gives you useful error messages when things stop working. There is even a vignette which takes you through a very relevant use-case: [Best practices for writing an API package](https://cran.r-project.org/web/packages/httr/vignettes/api-packages.html).

There is also a bunch of documentation online about HTTP and APIs, including several that are linked to within the httr vignettes:

* [HTTP: The Protocol Every Web Developer Must Know](https://code.tutsplus.com/tutorials/http-the-protocol-every-web-developer-must-know-part-1--net-31177)
* [HTTP Made Really Easy](http://www.jmarshall.com/easy/http/)
* [An Introduction to APIs](https://zapier.com/learn/apis/)

This is something that is well worth learning about. It will be useful regardless of whether or not you intend to develop a package, but it will *definitely* be useful if you intend to write a package that other people are going to be using to access APIs.

