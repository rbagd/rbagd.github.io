---
layout: post
title:  "Visualizing inflation data"
date:   2014-02-26 16:23:39
categories: R
---

Some time ago I have written a small web application which allows to study some basic pre-defined graphs using [inflation data for Belgium][inflation]. This post is a lengthy explanation and justification of the choices made while writing the code.

First, some general remarks about source of the data. Raw data includes price indices for many products and goes up to level 4 of UN COICOP statistical classification. Data is provided as a typical stand-alone Excel file and no API or XML feed exists for accessing the raw data. In order to be able to update these graphs monthly, that is every time new data comes out, I wrote the code accordingly to the current data format. If there are major changes, it may break. But well, there rarely are.

As said, inflation data is a bunch of indices varying over time at monthly frequency. Of most interest is yearly and monthly variation. Currently, these are the ones implemented. It would be straightforward to extend this.

One solution is to use a stacked barchart. If each product category is weighted according to its given weight and importance to general inflation, then the net height of a bar at a given month is by definition the inflation rate of that month. By net I mean that some product categories may experience a decrease in prices which should then be subtracted from the weighted total.

A major disadvantage of some of these barcharts is color scale. When 10 or more categories are present, it gets quite difficult to differentiate colors.

I would especially be interested in different ways to present inflation data. Of course, JavaScript-like interactivity would be a huge improvement already (I am still learning that) but it would be even more exciting to make talk some new methods instead just making it fancier.

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll's GitHub repo][jekyll-gh].

[inflation]: http://statbel.fgov.be/fr/statistiques/chiffres/economie/prix_consommation/
[jekyll-gh]: https://github.com/mojombo/jekyll
[jekyll]:    http://jekyllrb.com
