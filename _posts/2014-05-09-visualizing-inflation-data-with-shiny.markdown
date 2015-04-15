---
layout: post
title:  Visualizing inflation data with shiny
date:   2014-05-09 16:00:00
categories: R
---

Some time ago I have written a small web application which allows to plot some basic pre-defined graphs using [inflation data for Belgium][inflation]. This post serves as an explanation for the code behind it. The application itself can be found [here][my-app]. In short, it uses `shiny` package for `R` for web deployment and `lattice` for plotting. It will be a little slow to load but should be fast enough afterwards.

For starters, some general remarks about data itself. Raw data includes price indices for many products and goes up to level 4 of UN COICOP statistical classification. Unfortunately, data is provided as a typical stand-alone Excel file and no API or `XML` feed exists for accessing it. In order to be able to update these graphs monthly, I wrote the code accordingly to the current format of the Excel sheet (converted to `csv` for speed purposes). If there happens to be major changes in the Excel file, code will break. On the other hand, I don't expect any big ones in the near future.

As said, inflation data is a bunch of indices varying over time at monthly frequency. The value of any indice has no particular meaning but its monthly or annual growth rate is a measure of price change. There are two typical questions that anyone facing such data would ask. Firstly, what is current level of inflation? Is it high, is it low? What about its rate compared to recent history? Is inflation speeding up or slowing down? And so on. Nothing much fancier than a descriptive summary. Another question is much more interesting: what is driving inflation movements? Inflation index is a weighted measure of a myriad of products composing an average consumption bundle. Does any product seem to drive movements more? Furthermore, consumers are heterogeneous and may care about different categories. For example, some people may be concerned to notice that alcohol and tobacco have been experiencing significant price increases for two years. Is that a result of government policy intervention?

One solution to illustrate the drivers in a category is by using a stacked barchart. Descriptive graphs then become a useful side-effect. If each product category is weighted according to its given weight and importance to general inflation index, then the net height of a bar at a given month is by definition the inflation rate of that month. Stacks would then represent weighted contributions of different categories to the whole inflation index. Since contributions may be negative, the inflation rate is given by the net bar height. A disadvantage of these barcharts is a color scale which gets confusing when 10 or more categories are present.

As an illustration for possible analysis, some simple plots over the years would highly suggest that inflation in Belgium is in particular driven by oil and energy prices as general inflation trend seems to be highly correlated with contribution of categories that includes oil and energy prices. That is likely not a shocker but it underlines the importance of oil in Belgian economy.

Although a stacked barchart seems to me the most natural way to present inflation data, I would especially be any different means to do it. Please share if you have an idea. Of course, JavaScript-like interactivity would be a huge improvement already (I am still learning about that) but it would be even more exciting to think of whole new methodology instead of just making it fancier. For the latter, there is also `rCharts` package available on `GitHub` which allows easy transition from `R` to some JavaScript plotting libraries.

Any bugs, feature requests and further improvements can be posted at the [dedicated GitHub repo][github].

[my-app]: https://rytis.shinyapps.io/inflation
[inflation]: http://statbel.fgov.be/fr/statistiques/chiffres/economie/prix_consommation/
[github]: https://github.com/rbagd/inflation
