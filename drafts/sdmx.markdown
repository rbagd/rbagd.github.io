---
layout: post
title:  "Using SDMX protocol within R"
date:   2014-02-26 16:23:39
categories: R
---

`SDMX` stands for *Statistical Data and Metadata Exchange*. It is becoming a widely used standard for economic and population data providers such as ECB, Eurostat, OECD and many others.

Currently, most widely used implementation is known as `SDMX-ML̀` where data is exchanged via structured `XML` documents. All providers I have extensively tested implement `SDMX` protocol via `RESTful` API.

`R` has got a package `rsdmx` which works especially well with above listed institutions. Just construct or find a valid `SDMX` query, run `readSDMX()` from within `R` and in case your object contains data, `as.data.frame` method for that object is available.

Typically, an `SDMX-ML` query for obtaining data will consist of 4 parts:
* prefix link to the data warehouse of the data provider. Some examples would be:
  * http://sdw-wsrest.ecb.europa.eu/service/data/
  * http://stats.oecd.org/restsdmx/sdmx.ashx/GetData/
  * http://ec.europa.eu/eurostat/SDMX/diss-web/rest/data/
* dataflow would refer to a particular dataset. Here are two of many possible examples from above providers:
  * ECB:
    * EXR (exchange rates)
    * STS (short-term statistics)
  * Eurostat:
    * ...
  * OECD:
    * ULC_EEQ (unit labour costs)
    * ITF_INV-MTN_DATA (investment in transport infrastructure and maintenance spending)
* key which allows to identify a particular series within a dataset. Usually, it includes frequency of series, sector if available, indicates whether series are seasonally adjusted and so on. Let's analyze an example:`M.U2.EUR...EURIBOR3MD_+DJEURST.HSTA` which is a valid key to query FM (financial markets) dataset from ECB data warehouse. First, the key specifies that we want monthly data with `M`, `U2` specifies *Euro area* while `EUR` stands for euro currency. Three dots are actually wildcards that will allow multiple series selection. For Euribor rates, these three dots can be replaced by `.RT.MM.` (Reuters as source agency, money market). In case of oil prices, it's `.4F.CY.` where `CY` stands for commodity and `4F` for another data provider, though I am unable to identify which one. Obviously, `EURIBOR3MD_+DJEURST` indicates that we would like to download Euribor rates and oil prices. Finally, `HSTA` specifies that queried observation is a *historical close, average of observations through period*. By using wildcards, one can download multiple series at once, or even the whole dataset. Key selection supports `+` as `OR` operator, so that `M+D` would select all monthly and daily obervations.
* additional parameters such as detail level or start/end period (options depend on the implementation)


Jekyll also offers powerful support for code snippets:
Check out the [Jekyll docs][jekyll] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll's GitHub repo][jekyll-gh].

[jekyll-gh]: https://github.com/mojombo/jekyll
[jekyll]:    http://jekyllrb.com
