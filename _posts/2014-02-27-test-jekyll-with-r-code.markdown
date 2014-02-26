---
layout: post
title:  "Test Jekyll with R code"
date:   2014-02-27 16:23:39
categories: r
---

This is a sample post to test syntax highlighting for `R` files. It does not work too well locally.

{% highlight r %}
require(ggplot2)
gennorm = function(x) { rnorm(x) }
a = 2
plot(gennorm(200))

# generates a bunch of standard gaussian random variables
{% endhighlight %}

Check out the [Jekyll docs][jekyll] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll's GitHub repo][jekyll-gh].

[jekyll-gh]: https://github.com/mojombo/jekyll
[jekyll]:    http://jekyllrb.com
