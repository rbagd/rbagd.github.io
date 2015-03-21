---
layout: post
title:  "Using SDMX protocol within R"
date:   2014-02-26 16:23:39
categories: R
---

`SDMX` stands for *Statistical Data and Metadata Exchange*. It is becoming a widely used standard for economic and population data providers such as ECB, Eurostat, OECD, United Nations and many others.

Currently, most widely used implementation is known as `SDMX-ML̀` where data is exchanged via structured `XML` documents.

`R` has got a package `rsdmx` which works especially well with above listed institutions. Just construct or find a valid `SDMX` query, run `readSDMX()` from within `R` and in case your object contains data, `as.data.frame` method for that object is available.

Typically, an `SDMX` query for obtaining data will consist of 4 parts:
* prefix link to the data warehouse of the data provider. Some examples would be:
  * http://sdw-wsrest.ecb.europa.eu/service/data/
  * http://stats.oecd.org/restsdmx/sdmx.ashx/GetData/
  * http://stat.nbb.be/RestSDMX/sdmx.ashx/GetData/
  * http://ec.europa.eu/eurostat/SDMX/diss-web/rest/data/
  * add UN
* dataflow would refer to a particular dataset. Some examples from above providers would be:
  * ECB:
    * EXR (exchange rates)
    * STS (short-term statistics)
    * FM (financial market data)
  * Eurostat:
    * ...
* key which allows to identify a particular series within a dataset. By using wildcards, one can download multiple series at once, or even the whole dataset.
* additional parameters such as detail level or start/end period (options depend on the implementation)


Jekyll also offers powerful support for code snippets:
Check out the [Jekyll docs][jekyll] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll's GitHub repo][jekyll-gh].

[jekyll-gh]: https://github.com/mojombo/jekyll
[jekyll]:    http://jekyllrb.com
