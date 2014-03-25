---
layout: post
title:  "Financial data representation with openair"
date:   2014-03-25 16:23:39
categories: r
---

During an `R` meetup, one of the talks presented `openair` package useful for those involved in air quality measurements. Nonetheless, air quality data is just a time-stamped data frame and so resembles to any kind of time series. `openair` plot capabilities can be therefore easily used by anyone outside its precise domain.

{% highlight r %}
data(iris)
corPlot(iris)
{% endhighlight %}

Graphing seems to be based on `lattice` package of which I admittedly am not a user.

One of the functions available in `openair` is a calendar plot which allows for an easy color-based representation of daily events. `openair` package seems to implement its plot capabilities based on a simple data frame with possibly appropriately named columns. The below code represents a yearly overview of Standard & Poor's 500 index performance with green and red standing for positive and negative daily returns respectively. Plots over recent years would give a simple comprehensible overstanding on the impact of financial crisis on stock market performance. When plotted, number of red days is quite impressive in 2008 or 2009.

{% highlight r %}
library(quantmod)       ## This library allows for an easy way to get daily stock data
getSymbols("^GSPC")
data <- as.data.frame(GSPC)
data$returns <- with(data, (GSPC.Close/GSPC.Open-1)*100)
data$date <- as.Date(rownames(data))
calendarPlot(data, pollutant="returns", year=2013,
             cols=c("darkred", "red", "gray", "green", "darkgreen"),
             main="S&P500 returns in % (2013)")
{% endhighlight %}

![My helpful screenshot]({{ site.url }}/assets/images/sp500_calendarplot.png)

Another possibility to represent data is by means of rose plots which are easy to understand for typical air data. They represent frequency of windspeed in either of the possible directions, i.e. those composed of north, east, south or west. In a data frame, these directions are represented by angles from O to 360 degrees. Such classification does not appear naturally for financial series, so one may think of repartitioning one of the variables, e.g. daily trade volume, so that it acts as a degree.

{% highlight r %}
data$volatility <-  (data$returns)^2
data$volume <- cut(data$GSPC.Volume, breaks=19,
                   labels=seq(0,360, length.out=19))
data$volume <- as.numeric(levels(data$volume)[data$volume])
windRose(data, ws="volatility", wd="volume")
{% endhighlight %}

This is somehow artificial but it does support the idea that stock volatility is higher when volume traded is higher. In the graph this would mean moving clockwise further from the North.

