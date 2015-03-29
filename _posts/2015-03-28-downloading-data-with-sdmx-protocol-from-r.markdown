---
layout: post
title:  "Downloading data with SDMX protocol from R"
date:   2015-03-28 00:03:00
categories: R
---

##### About SDMX

`SDMX` stands for *Statistical Data and Metadata Exchange*. It is becoming a widely used standard for economic and population data providers such as [ECB][ecb-sdmx], [Eurostat][eurostat-sdmx], [OECD][oecd-sdmx] and many others.

Currently, most widely used `SDMX` implementation for economic data is known as `SDMX-ML̀` where data is exchanged via structured `XML` documents. All providers I have extensively tested implement `SDMX` protocol via `RESTful` API. Queries for data may be of two major types: data itself or description of data (metadata). 

##### Data

Let's start with data itself. Typically, an `SDMX-ML` URL for obtaining `XML` document with data will consist of 4 parts:

* prefix link to the data warehouse of the data provider. Some examples would be:
  * [ECB](http://sdw-wsrest.ecb.europa.eu/service/data/)
  * [OECD](http://stats.oecd.org/restsdmx/sdmx.ashx/GetData/)
  * [Eurostat](http://ec.europa.eu/eurostat/SDMX/diss-web/rest/data/)
<br /> These prefix links by themselves will not return anything and need to be combined with the rest of the query.

* name of a dataset (commonly referred to as dataflow). Here are two of many possible examples for each of the above providers:
  * ECB: `EXR` (exchange rates), `FM` (financial markets)
  * Eurostat: `demo_pjan` (population on 1 January), `teicp000` (harmonized price indices)
  * OECD: `ULC_EEQ` (unit labour costs), `GOV_DEBT` (government debt) <br /> I don't know any other way of discovering datasets other than browsing through the website of data provider. ECB has a [link](https://sdw-wsrest.ecb.europa.eu/service/datastructure/ECB?references=dataflow) with all dataflows, however I couldn't locate anything similar for other institutions.
* key which allows to identify a particular series within a dataset. Usually, it includes frequency of series, sector, country or type if available, indicates whether series are seasonally adjusted and so on. Let's analyze a single example:
  * `M.U2.EUR...EURIBOR3MD_+OILBRNI.HSTA` <br/>
This is a valid key to query `FM` (financial markets) dataset from ECB data warehouse. First, the key specifies that we want monthly data with `M`, `U2` stands for *Euro area* code while `EUR` stands for euro currency. Three sequential dots are actually wildcards that will allow multiple series selection. For Euribor rates, these three dots `...` can be replaced by `.RT.MM.` (Reuters as source agency, money market). In case of oil prices, it's `.4F.CY.` where `CY` stands for commodity and `4F` for another data provider, though I am unable to identify which one. Obviously, `EURIBOR3MD_+OILBRNI` indicates that we would like to download Euribor rates and Brent oil prices. Finally, `HSTA` specifies that queried observation is a *historical close, average of observations through period*. By using wildcards as shown, i.e. skipping a part of the key while keeping the dot, one can select all possibilities in that part of the query, thus one could also download the whole dataset. Key selection supports `+` as `OR` operator, so that `M+D` would select all monthly and daily obervations.
* additional parameters such as detail level or start/end period (options depend on the implementation). Usually, one can specify `some_key/?startPeriod=2000&endPeriod=2005`. Here, some additional flexibility may be allowed, for example downloading only newly updated or latest data. One should consult data provider's documentation for extra details.

To finish up, here's the [link to the `XML` data file](http://sdw-wsrest.ecb.europa.eu/service/data/FM/M.U2.EUR...EURIBOR3MD_+OILBRNI.HSTA/?startPeriod=2000&endPeriod=2005) for the above Euribor and oil prices. It suffices to concatenate the listed elements separated with a backslash.

Most of the work with `SDMX` goes into key construction. Unfortunately, there is no single standard of constructing keys so that frequency or seasonal adjustment dimensions may appear in the beginning or at the end of key depending on the data provider, or even the dataset. Here's why it's useful to be able to study metadata for each dataset first.

##### Metadata

`M` in `SDMX` stands for metadata and thus additional information on series can also be exchanged in a structured manner via `Data Structure Documents` (DSD). DSD queries can be much simpler and require only two elements: a prefix and a dataset. I list here below a sample link to a DSD structure of one of the datasets for each of the 3 providers. You can easily infer the query structure.

* [ECB](https://sdw-wsrest.ecb.europa.eu/service/datastructure/ECB/ECB_EXR1/1.0?references=children)
* [OECD](http://stats.oecd.org/restsdmx/sdmx.ashx/GetDataStructure/GOV_DEBT)
* [Eurostat](http://ec.europa.eu/eurostat/SDMX/diss-web/rest/datastructure/ESTAT/DSD_teicp000)

Let's take an example. [This link](http://stats.oecd.org/restsdmx/sdmx.ashx/GetDataStructure/GOV_DEBT) requests an `XML` document from OECD describing data in **Government debt** dataset. Analyzing this structure allows to construct more accurate data queries.

In particular, we are interested in two elements: `CodeLists` and `Dimension`. `Dimension` element explains in what order and how key for queries is constructed while `CodeLists` provides possible values for query elements. In this particular case, `Dimension` elements indicates that a key is constructed as follows: country, debt type, frequency, unit and variable. Thus, taking a look at `CodeLists` element suggests that a possible key would be `FIN+BEL.AMT+GRS+NET.A.USD.3` which requests total Finnish and Belgian government debt flows and stocks in US Dollars at annual frequency. Here's the [full link to the `XML` file](http://stats.oecd.org/restsdmx/sdmx.ashx/GetData/GOV_DEBT/FIN+BEL.AMT+GRS+NET.A.USD.3). It turns that only `AMT` (outstanding amounts) data is provided.

##### SDMX and `R`

I cannot help but mention `R` package `rsdmx` which works especially well with above listed institutions and is meant to become an `SDMX-ML` format abstraction library. I suggest to get it directly from Github instead of CRAN. Lead maintainer and developer of `rsdmx`, [Emmanuel Blondel](https://github.com/eblondel), is very responsive and package is routinely improved. Kudos to him. Its usage is extremely simple. Just construct or find a valid `SDMX` query, whether for data or data structure, run `readSDMX()` from within `R` and in case your object is of appropriate `SDMX` class, `as.data.frame` method for that object is available. With some work, key construction and data download can be fully automated from `R`.

To finish up, here's a small illustration in `R` to obtain a piece of data and optionally convert it to a timestamped dataframe.

```
library(rsdmx)
url <- "http://stats.oecd.org/restsdmx/sdmx.ashx/GetData/GOV_DEBT/FIN+BEL.AMT.A.USD.3"
xml_data <- readSDMX(url)
data <- as.data.frame(xml_data)

library(reshape2)
dcast(data, date ~ COU+DTYPE, value.var="obsValue")
```

Dealing with public data has never been as fun. `SDMX` is a great protocol for easily constructing and maintaining public data you are interested in. No more random `Excel` or `csv` files. Amen.

[ecb-sdmx]: https://www.ecb.europa.eu/stats/services/sdmx/html/index.en.html
[eurostat-sdmx]: http://ec.europa.eu/eurostat/en/data/sdmx-data-metadata-exchange
[oecd-sdmx]: http://stats.oecd.org/
