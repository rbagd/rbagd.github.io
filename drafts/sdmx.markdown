---
layout: post
title:  "Using SDMX protocol within R"
date:   2015-03-22 16:23:39
categories: R
---

###### About SDMX

`SDMX` stands for *Statistical Data and Metadata Exchange*. It is becoming a widely used standard for economic and population data providers such as [ECB][ecb-sdmx], [Eurostat][eurostat-sdmx], [OECD][oecd-sdmx] and many others.

Currently, most widely used `SDMX` implementation for economic data is known as `SDMX-ML̀` where data is exchanged via structured `XML` documents. All providers I have extensively tested implement `SDMX` protocol via `RESTful` API. Typically, an `SDMX-ML` URL for obtaining `XML` document with data will consist of 4 parts:

* prefix link to the data warehouse of the data provider. Some examples would be:
  * http://sdw-wsrest.ecb.europa.eu/service/data/
  * http://stats.oecd.org/restsdmx/sdmx.ashx/GetData/
  * http://ec.europa.eu/eurostat/SDMX/diss-web/rest/data/
* name of a dataset (commonly referred to as dataflow). Here are two of many possible examples for each of the above providers:
  * ECB: `EXR` (exchange rates), `STS` (short-term statistics)
  * Eurostat: ...
  * OECD: `ULC_EEQ` (unit labour costs), `ITF_INV-MTN_DATA` (investment in transport infrastructure and maintenance spending)
* key which allows to identify a particular series within a dataset. Usually, it includes frequency of series, sector if available, indicates whether series are seasonally adjusted and so on. Let's analyze an example:`M.U2.EUR...EURIBOR3MD_+DJEURST.HSTA` which is a valid key to query FM (financial markets) dataset from ECB data warehouse. First, the key specifies that we want monthly data with `M`, `U2` specifies *Euro area* while `EUR` stands for euro currency. Three dots are actually wildcards that will allow multiple series selection. For Euribor rates, these three dots can be replaced by `.RT.MM.` (Reuters as source agency, money market). In case of oil prices, it's `.4F.CY.` where `CY` stands for commodity and `4F` for another data provider, though I am unable to identify which one. Obviously, `EURIBOR3MD_+DJEURST` indicates that we would like to download Euribor rates and oil prices. Finally, `HSTA` specifies that queried observation is a *historical close, average of observations through period*. By using wildcards, one can download multiple series at once, or even the whole dataset. Key selection supports `+` as `OR` operator, so that `M+D` would select all monthly and daily obervations.
* additional parameters such as detail level or start/end period (options depend on the implementation)

Simple queries as above are a . However, `M` in `SDMX` stands for metadata and thus additional information on series can also be exchanged in a structured manner via `Data Structure Documents` (DSD). DSD queries can be much simpler. For example, `http://stats.oecd.org/restsdmx/sdmx.ashx/GetDataStructure/GOV_DEBT` requests an `XML` document from OECD describing data in **Government debt** dataset. Analyzing this structure allows to construct more accurate data queries.

###### SDMX and `R`

`R` has got a package `rsdmx` which works especially well with above listed institutions. Just construct or find a valid `SDMX` query, run `readSDMX()` from within `R` and in case your object contains data, `as.data.frame` method for that object is available.


[ecb-sdmx]: https://www.ecb.europa.eu/stats/services/sdmx/html/index.en.html
[eurostat-sdmx]: http://ec.europa.eu/eurostat/en/data/sdmx-data-metadata-exchange
[oecd-sdmx]: http://stats.oecd.org/
