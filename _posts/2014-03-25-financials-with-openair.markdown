---
layout: post
title:  "Financial data representation with openair"
date:   2014-03-25 16:23:39
categories: r
---

During an `R` meetup, one of the talks presented `openair` package useful for those involved in air quality measurements. Nonetheless, air quality data is just a time-stamped data frame and as such is similar to any kind of time series. `openair` plot capabilities could be therefore easily used by anyone outside weather modelling domain. This code snippet generates a simple but pretty looking correlation plot.

{% highlight r %}
library(openair)       # Let's not forget to load the package itself!

data(iris)
corPlot(iris)
{% endhighlight %}

This is what you get.

![Correlation plot for 'iris' dataset]({{ site.url }}/assets/images/iris_corplot.png)

Graphing in `openair` seems to be based on `lattice` package of which I admittedly am not a user. Experienced `lattice` users will probably find this less impressive.

One of the very useful functions available in `openair` is a calendar plot which allows for an easy color-based representation of daily events. As `openair` package seems to implement its plot capabilities on a class of simple data frames with appropriately named columns, these plots can be easily recycled. The below code represents a yearly overview of Standard & Poor's 500 index performance with green and red standing for positive and negative daily percentage returns respectively. Plots over recent years would give a simple comprehensible overview on the impact of financial crisis on stock market performance. When plotted, number of red days is quite impressive in 2008 or 2009.

{% highlight r %}
library(quantmod)       # This library allows for an easy way to get daily stock data
getSymbols("^GSPC")
data <- as.data.frame(GSPC)
data$returns <- with(data, (GSPC.Close/GSPC.Open-1)*100)
data$date <- as.Date(rownames(data))
calendarPlot(data, pollutant="returns", year=2013,
             cols=c("darkred", "red", "gray", "green", "darkgreen"),
             main="S&P500 returns in % (2013)")
{% endhighlight %}

Here is a picture of how the graph looks in practice. It looks neat, doesn't it?

![S&P500 returns in 2013]({{ site.url }}/assets/images/sp500_calendarplot.png)

Another possibility to represent data is by means of rose plots which are easy to understand for typical air data. They represent frequency of windspeed in either of the possible directions, i.e. combinations of north, east, south or west. In a data frame relevant to `openair`, these directions are represented by angles from 0 to 360 degrees. Such classification does not appear naturally for financial series, so one could think of repartitioning one of the variables, e.g. daily trade volume, so that it acts as a degree.

{% highlight r %}
data$volatility <-  (data$returns)^2
data$volume <- cut(data$GSPC.Volume, breaks=19,
                   labels=seq(0,360, length.out=19))
data$volume <- as.numeric(levels(data$volume)[data$volume])
windRose(data, ws="volatility", wd="volume")
{% endhighlight %}

This is somehow an artificial construct but it does support the idea that stock volatility is higher when volume traded is higher (in the graph, higher volume would mean moving clockwise further from the North). It would probably be much more telling if traded volume of the observed stock would be more uniformly distributed across chosen intervals. Alas, some may be right to argue that even a simple grouped barplot could do a better job in this case.

