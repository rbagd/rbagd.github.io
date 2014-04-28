---
layout: post
title:  "First attempt at d3"
date:   2014-02-26 16:23:39
categories: d3
---

This is an attempt to construct a basic data visualization based on geographical data for learning purposes. In this case, I chose to illustrate housing price distribution in Belgium at a provincial level. Data is taken from [SPF Economie][bestat-housing]. 

* Download a shapefile or `RData` file from [GADM](http://www.gadm.org/country)
* Download separately data for housing, combine it with geodata and export it as GeoJSON. For dealing with JSON files, I found `jsonlite` package for `R` very useful.
* Load the file with `d3` library and make it look awesome  

I used [Plunker](http://plnkr.co) for livecoding. Some challenges for a novice like me included correctly importing the data as well as nicely displayed tooltips and legend.

Code for this visualization is available at `bitbucket`.

[bestat-housing]: http://bestat.economie.fgov.be/BeStat/BeStatMultidimensionalAnalysis?loadDefaultId=479
