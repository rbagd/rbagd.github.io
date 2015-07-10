---
layout: post
title:  "Seasonal time series adjustment with X-13ARIMA-SEATS in R"
date:   2015-07-15 16:00:00
categories: r statistics 
---

Time series data, particularly in economics, may and does often contain seasonality and trend components. Data analysis and forecasts can be significantly improved if these components are taken into account. It is one of the reasons why many economic data you are presented with is most likely seasonally adjusted. For example, fresh food prices are a neat example of data subject to important seasonal patterns.

A few methods emerged over time to help statisticians adjust time series for seasonality, of which `X11` became the dominant one, notably due to its relative simplicity.

Here is a small dictionary:

* `X11`: in short, seasonal adjustment is achieved through linear `ARMA`-like filtering and outlier detection
* `TRAMO`: time series regression with ARIMA noise, missing observations and outliers
* `SEATS`: signal extraction in ARIMA time series

Some more discussion on `X11` and seasonal adjustment is available [here][gomezmaravall]. More on filters can be found [here][filterinfo]. Information on `TRAMO/SEATS` is available on its [website][tramoseats].

Currently, there are two main software packages implementing above described methods. Others also exist but are likely to be not free. [X-13ARIMA-SEATS][x13] maintained by U.S. Census Bureau is based on `X11` method and its improvements but also implements `SEATS`. Alternatively, [`JDemetra+`][demetra] maintained by National Bank of Belgium and Eurostat provides a `Java`-based implementation for all of the above methods. `JDemetra+` is well done, cross-platform and provides plenty of point and click options to work with your time series, although documentation is quite seriously lacking. On the other hand, `X-13ARIMA-SEATS` does not come with a `GUI` unless you are on `Windows` but has an excellent [reference manual][x13manual]. An advantage for the latter is also the possibility to remain within the `R` universe.

In `R`, two packages provide an interface to `X-13ARIMA-SEATS` software: [`seasonal`][cranseasonal] and [`x12`][cranx12]. Additionally, the latter has a GUI via [`x12GUI`][cranx12gui] package providing some button clicking options.

So just to reiterate and eliminate all confusion: `X11` is a seasonal adjustment method, its major implementation is found in the software called `X-13ARIMA-SEATS` while `x12` package for `R` provides only an interface to this software. `X-13ARIMA-SEATS` superseded its previous version known as `X-12-ARIMA` (also supported by `x12` package), hence the variation in numbering.

Both `R` packages can be found to be useful in general. `seasonal` uses `SEATS` method as default, while `x12` supports only `X11` method. Both packages also have the same workflow: first, rewrite the user-defined model in a `.spc` file understood by `X13-ARIMA-SEATS` binary, run the binary, read the `.out` file and parse it back into `R`. Both also aim to implement all or most of the options available in the true `X-13ARIMA-SEATS` software.

For an occasional use, `seasonal` is probably preferable as its more intuitive and will probably easier to use. In both cases, `X-13ARIMA-SEATS` binary has to be downloaded manually and its path set up as explained in documentation in `X13_PATH` is set up as explained in the vignette of the package.

{% highlight r %}
library(seasonal)
Sys.setenv(X13_PATH = 'some_dir')
seas(AirPassengers, regression.variables="seasonal")

library(x12)
x13path('some_dir/x13as')
x12(AirPassengers, setP(new("x12Parameter",
                        list(regression.variables="seasonal"))))
{% endhighlight %}

Note that the two snippets above will give different outputs due to differences in default settings. For identical results, add `outlier=NULL, regression.aictest=NULL, x11=''` options within `seas` function.

Some advantages of `seasonal` over `x12`:

* `seasonal` supports both `X11` and `SEATS` methods. `x12` does not support `SEATS` yet.
* Although it is a question of taste, `seasonal` is probably better polished visually, e.g. graphical outlier selection is very neat as well as interactive `shiny`-based model discovery.
* In case of external regressors, `seasonal` does not require writing those to a file as `x12` does. It's a minor inconvenience though.
* `seasonal` has a function `genhol` for easier generation of holiday-related seasonal effects. This simple function is really great.
* `x12` only provides coefficient estimate table for external regressors, not `AR` or `MA` coefficients contrary to `seasonal`. Unless I am missing something, `ARMA` coefficients aren't part of the final `x12` estimated fit object.

Some advantages of `x12` over `seasonal`:

* `summary` method in `x12` also provides `M`-quality statistics for the model which is a nice guidance during modelling procedure. These statistics are however only available for `X11` method. If you estimate an `X11` model with `seasonal`, those statistics can only for now be read in the output file. It's also quite nice that `x12` provides a possibility for an extended summary output with `fullSummary=TRUE` option.
* plotting capabilities are quite similar, though `x12` provides a nice plot with original series combined with forecasts and their confidence band.
* `x12` takes an `S4`-type `OOP` approach to model estimation. While it's not as intuitive as `seasonal`, it gets quite useful if you have to deal with many series sharing similar parametrization. `x12` would also allow easier parallelization via `x12Batch` class. It's more a different approach rather than a (dis)advantage.

Feel free to test everything I have discussed to find what suits your workflow best.

[gomezmaravall]: http://www.bde.es/f/webbde/SES/servicio/software/tramo/sasex.pdf
[filterinfo]: http://www.abs.gov.au/websitedbs/d3310114.nsf/51c9a3d36edfd0dfca256acb00118404/5fc845406def2c3dca256ce100188f8e!OpenDocument
[tramoseats]: http://www.bde.es/bde/en/secciones/servicios/Profesionales/Programas_estadi/Programas_estad_d9fa7f3710fd821.html
[demetra]: https://github.com/jdemetra/jdemetra-app/releases
[x13]: https://www.census.gov/srd/www/x13as
[x13manual]: https://www.census.gov/ts/x13as/docX13AS.pdf
[cranseasonal]: http://cran.r-project.org/web/packages/seasonal/index.html
[cranx12]: http://cran.r-project.org/web/packages/x12/index.html
[cranx12gui]: http://cran.r-project.org/web/packages/x12GUI/index.html
