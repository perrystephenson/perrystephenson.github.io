---
layout:     post
title:      "DVN Assignment 1"
subtitle:   "Part A - ggplot2"
date:       2017-03-23 19:30:00
author:     "Perry Stephenson"
header-img: "img/dvn_assn1a.jpg"
---

Assignment 1 for Data Visualisation and Narratives requires the use of three different visualisation tools over three weeks, presumably as part of a journey of discovery through different visualisation tools. For this first tool I'm going to use something I already know reasonably well - Hadley Wickham's ggplot2 package for R. This still counts as an exploration of the tool! I promise to try something new next week.

The dataset I will be using for this analysis is something I've been thinking of looking at for a while: the Australian Road Deaths Database. I've heard a lot of statistics in the media which are presumably derived from this dataset, and I've long suspected that there are some inaccurate claims floating around the place, so I thought it was worth taking a look.

First things first, I need to grab the dataset.

``` r
library(httr)
library(readr)
request <- GET("https://bitre.gov.au/statistics/safety/files/Fatalities_Jan2017.csv")
text <- content(request, as="text")
raw_data <- read_csv(text)
```

Then I need to do some tidying up.

``` r
suppressPackageStartupMessages(library(tidyverse))
```

    ## Conflicts with tidy packages ----------------------------------------------

``` r
suppressPackageStartupMessages(library(lubridate))
fatal <- 
  raw_data %>% 
  mutate(Date = make_datetime(Year, Month, Day, Hour, Minute),
         Bus_Involvement               = Bus_Involvement == "Yes",
         Heavy_Rigid_Truck_Involvement = Heavy_Rigid_Truck_Involvement == "Yes",
         Articulated_Truck_Involvement = Articulated_Truck_Involvement == "Yes",
         State      = as.factor(State),
         Crash_Type = as.factor(Crash_Type),
         Road_User  = as.factor(Road_User),
         Gender     = as.factor(Gender)) %>% 
  select(-Day, -Month, -Year, -Hour, -Minute)
```

The first visualisation I will use is barely even a visualisation, but I would argue that a table of summary statistics counts! It certainly helps to guide the narrative by drawing the eye to interesting patterns.

``` r
fatal %>% 
  select(-CrashID, -Date) %>% 
  summary()
```

    ##      State            Crash_Type    Bus_Involvement
    ##  NSW    :14750   Multiple  :20328   Mode :logical  
    ##  VIC    :10459   Pedestrian: 7536   FALSE:46582    
    ##  QLD    : 9324   Single    :19606   TRUE :888      
    ##  WA     : 5536                      NA's :0        
    ##  SA     : 4095                                     
    ##  NT     : 1471                                     
    ##  (Other): 1835                                     
    ##  Heavy_Rigid_Truck_Involvement Articulated_Truck_Involvement
    ##  Mode :logical                 Mode :logical                
    ##  FALSE:19424                   FALSE:42598                  
    ##  TRUE :1262                    TRUE :4872                   
    ##  NA's :26784                   NA's :0                      
    ##                                                             
    ##                                                             
    ##                                                             
    ##   Speed_Limit                             Road_User         Gender     
    ##  Min.   :  8.00   Bicyclist                    : 1242   Female :13630  
    ##  1st Qu.: 60.00   Driver                       :21276   Male   :33817  
    ##  Median : 90.00   Motor cycle pillion passenger:  343   Unknown:   23  
    ##  Mean   : 93.87   Motor cycle rider            : 5703                  
    ##  3rd Qu.:100.00   Other/unknown                :   62                  
    ##  Max.   :900.00   Passenger                    :11325                  
    ##  NA's   :725      Pedestrian                   : 7519                  
    ##       Age        
    ##  Min.   :  0.00  
    ##  1st Qu.: 22.00  
    ##  Median : 34.00  
    ##  Mean   : 39.16  
    ##  3rd Qu.: 54.00  
    ##  Max.   :100.00  
    ##  NA's   :82

To me, this is a great visualisation, as it raises heaps of questions I want to answer. For example, are you more likely to die on the road if you live in NSW or VIC, or is it due to the relatively large populations in each of these states? I can use ggplot2 to find out by scaling the fatality numbers according to the current population of each state (via the ABS).

``` r
library(ggplot2)
state_pops <- tribble(~State,    ~Pop, 
                       "NSW", 7725900, 
                       "VIC", 6068000, 
                       "QLD", 4844500, 
                       "WA",  2617200,
                       "SA",  1708200, 
                       "NT",   244900,  
                       "TAS",  519100,  
                       "ACT",  396100)
state_fatal <- fatal %>% 
  left_join(state_pops, by = "State") %>% 
  mutate(Weight = 1 / Pop)
ggplot(state_fatal) +
  geom_bar(aes(x = State, weight = Weight)) + 
  ylab("Fatalities per state / state population")
```

![](part_a_ggplot2_files/figure-markdown_github/unnamed-chunk-4-1.png)

This shows that NSW and VIC are right in the middle of the pack, and it was indeed their large populations that were making them appear so large in comparison to the other states. This visualisation also demonstrates the significantly higher fatality rate in the Northern Territory, and shows that the ACT is the best performing state for road fatalities.

The beauty of ggplot2 is that it is now really simple to start overlaying some of the other attributes. For example, we can overlay the crash types, the types of vehicles involved, the road user who was killed, and gender.

``` r
ggplot(state_fatal) +
  geom_bar(aes(x = State, weight = Weight, fill = Crash_Type)) + 
  ylab("Fatalities per state / state population")
```

![](part_a_ggplot2_files/figure-markdown_github/unnamed-chunk-5-1.png)

This is useful, but because of the huge differences in bar heights, it's not immediately clear what the relative differences are between states. To fix this, we can use the "fill" position.

``` r
ggplot(state_fatal) +
  geom_bar(aes(x = State, weight = Weight, fill = Crash_Type), 
           position = "fill") + 
  ylab("Fraction of fatalities in state")
```

![](part_a_ggplot2_files/figure-markdown_github/unnamed-chunk-6-1.png)

This is a much better visualisation, as it allows the reader to easily see that Northern Territory and Western Australia seem to have a much higher rate of single vehicle fatalities than the other states, and a lower rate of multiple vehicle fatalities. We can now extend this to the other attributes we want to look at.

``` r
state_fatal %>% 
  mutate(Heavy_Vehicle_Involved = Bus_Involvement | 
                                  Heavy_Rigid_Truck_Involvement | 
                                  Articulated_Truck_Involvement) %>% 
  filter(!is.na(Heavy_Vehicle_Involved)) %>% 
  ggplot() +
  geom_bar(aes(x = State, weight = Weight, 
               fill = Heavy_Vehicle_Involved), 
           position = "fill") + 
  ylab("Fraction of fatalities in state")
```

![](part_a_ggplot2_files/figure-markdown_github/unnamed-chunk-7-1.png)

This is interesting as it shows that the rate of heavy vehicle involvement in fatal crashes is relatively constant across states, with a few notable exceptions. I can not think of any plausible explanations for why this should be the case, but this visualisation certainly provides enough intrigue that I'm probably going to try and figure it out later on.

``` r
ggplot(state_fatal) +
  geom_bar(aes(x = State, weight = Weight, fill = Gender), 
           position = "fill") + 
  ylab("Fraction of fatalities in state")
```

![](part_a_ggplot2_files/figure-markdown_github/unnamed-chunk-8-1.png)

This is hardly surprising given everything that we hear about male drivers. We can drill down further on this one later, given that we also have the age of the deceased.

``` r
ggplot(state_fatal) +
  geom_bar(aes(x = State, weight = Weight, fill = Road_User), 
           position = "fill") + 
  ylab("Fraction of fatalities in state")
```

![](part_a_ggplot2_files/figure-markdown_github/unnamed-chunk-9-1.png)

This is what I really came here for. Every time I tell someone that I'm a cyclist, their immediate assumption is that I am probably going to be killed on the roads. Any time statistics are used to support this widely held belief, they tend to be of the form "fatalities per million km travelled". This is a bit misleading when looking at vehicles that don't travel very far - a typical motorist travels 15-20 thousand km every year, where a commuting cyclist riding 50km per week will only achieve 2500km per year. There isn't an easy way to compare these two risks of death, as you could argue that fatalities per million hours on the road might be better for some people, whilst fatalities per million people per year using average km might be better for others.

All of my existing skepticism aside, the visualisation above clearly shows that the risk of any given fatality (on any given day in any given state) being a cyclist is very low - several orders of magnitude lower than if you were a motorcyclist or even a pedestrian. By far, most fatalities on Australian roads are drawn from the population of people inside passenger vehicles, either as the driver or as a passenger.

Now to drill down further, and explore more of what ggplot2 can do. We can produce density plots, which allows an intuitive way to understand how outcomes change in response to continuous variables. Splitting this up by attributes lets us assess claims such as "young men are the most likely to die on the roads".

``` r
fatal %>% 
  filter(!is.na(Age)) %>% 
  filter(Age > 0) %>% 
  ggplot() +
  geom_density(aes(x = Age, y = ..count.., fill = Gender), 
               alpha = 0.5)
```

![](part_a_ggplot2_files/figure-markdown_github/unnamed-chunk-10-1.png)

From this visualisation it is clear that young men are indeed the most likely to die on the roads, and that men of all ages are more likely to die than women of the same age. It is also clear that the distributions are roughly bimodal, and that there is likely a "youth" factor and an "old age" factor behind each of these peaks.

One final thing that I wanted to inspect was *where* cyclists die on the roads, because I've always had a completely unfounded suspicion that cyclists are dying on main roads in far greater numbers than quieter back roads. This is going to be hard to test with the data available, but I can use speed as a proxy for road traffic, and I will filter for the last 20 years in NSW to account for the introduction of 50km zones in residential areas.

``` r
fatal %>% 
  filter(Date > now() - years(20)) %>% 
  filter(State == "NSW") %>% 
  filter(!is.na(Speed_Limit)) %>% 
  ggplot() +
  geom_bar(aes(x = as.factor(Speed_Limit)))
```

![](part_a_ggplot2_files/figure-markdown_github/unnamed-chunk-11-1.png)

It's fair to say that this visualisation is far from confirming my suspicions, however it is certainly showing that 100km/h roads are single most likely place for a cyclist to die, accounting for over 1/3 of all cyclist deaths. The relative differences between 50/60/80km/h roads are harder to interpret due to the unknown number of kilometers travelled by cyclists on each of these road types - whilst 50km/h roads look relatively safe it could be a function of these roads being relatively rare.

It's worth bringing this commentary back to ggplot2, given that was the actual point of this assignment. As demonstrated above ggplot2 makes it really easy to drill down into datasets and get them to show you what's hidden within. If you can ask a question of the data, you can plot the answer in ggplot2 using just a few lines of code. This means that it fits very nicely into data science workflows as the data never leaves R - you don't need to reformat it for some external tool, and you don't need to export it and move huge files around. It can deal with surprisingly large datasets, and it can be customised to make your plots look as exciting as you like.

In summary, ggplot2 is the visualisation tool I use more than any other, due to the ease of use and the fact I can use it directly from R.
