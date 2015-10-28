---
layout: post
title:  "Geomapping with BelgiumStatistics"
date:   2015-10-28 15:23:39
categories: r
---

Last week [SPF Economie][spf] kickstarted Open Data project for easier access to many of their datasets. Just yesterday [`BelgiumStatistics`][belgium-stat] package for `R` got released allowing for easier integration of these datasets within `R`. To celebrate this, I thought of making some maps just to play with available datasets.

Apparently, `BelgiumMaps` will be released soon so that plotting such maps will be much easier. While waiting for it, here is an example already of what we can already do with publicly available data. I use shapefiles provided by [Atlas Numérique de Belgique][ulg-atlas] maintained by Université de Liège. In my case, all of this is stored in a `PostgreSQL` database although this is not necessary. You can download these manually and adjust `readOGR` function accordingly.

Here is the script I will use to import maps that I will need later.

{% highlight r %}
library(RPostgreSQL)

drv <- dbDriver("PostgreSQL")
con <- dbConnect(drv, dbname="geo",
                 host="localhost", port=5432,
                 user=user, password=pw)

library(rgdal)
library(sp)
library(rgeos)
library(maptools)

map_communes <- readOGR(dsn="PG:dbname='geo'", layer = "ulg.communes")
map_provinces <- readOGR(dsn="PG:dbname='geo'", layer = "ulg.provinces")
highways <- readOGR(dsn="PG:dbname='geo'", layer="ulg.autoroutes")

# Let us adjust the projection coordinates so that Belgian map looks good

crs_string <- "+proj=merc +a=6378137 +b=6378137 
               +lat_ts=0.0 +lon_0=0.0 +x_0=0.0 +y_0=0 +k=1.0
               +units=m +nadgrids=@null +no_defs"

map_communes <- spTransform(map_communes, CRS(crs_string))
map_provinces <- spTransform(map_provinces, CRS(crs_string))
highways <- spTransform(highways, CRS(crs_string))
{% endhighlight %}

Now it remains to load up the Belgian data and plot some maps. I decided to check terrain occupation and compare it between years 1983 and 2015. My variable of interest is absolute change in surface area for terrain with constructions (declared officially). To make it short, I will call it construction area. This area includes residential buildings, commercial buildings, public buildings, monuments and a few more. As you may expect, this area increased for almost all municipalities between 1983 and 2015. The only exception is Farciennes municipality in the province of Hainaut where it slightly decreased.

Thankfully, the map and terrain occupation datasets have INS identifiers which allows to easily do joins to match municipality geodata with appropriate statistics.

{% highlight r %}
library(BelgiumStatistics)
data(TF_EAE_LAND_OCCUPTN_1983)
data(TF_EAE_LAND_OCCUPTN_2015)

library(dplyr)

transformData <- function(df, terrain) {
              df %>% group_by(CD_MUNTY_REFNIS, TX_RUB_FR_LVL_2) %>%
                     filter(TX_RUB_FR_LVL_2 == terrain) %>%
                     summarise(total = sum(MS_TOT_SUR))
}

terrain <- "Parcelles bâties"
sumData83 <- transformData(TF_EAE_LAND_OCCUPTN_1983, terrain)
sumData15 <- transformData(TF_EAE_LAND_OCCUPTN_2015, terrain)

to_map_data <- merge(sumData83, sumData15, by="CD_MUNTY_REFNIS") %>%
               mutate(difference = total.y - total.x) %>%
               mutate(diffCut = cut(difference,
                      breaks = c(min(difference), 
                               quantile(difference, c(0.25,0.5,0.75,0.9)),
                               max(difference)))) %>%
               ungroup()

map_data <- merge(map_communes, to_map_data,
                  by.x="ins", by.y="CD_MUNTY_REFNIS")
{% endhighlight %}

We have generated a full dataset that can be now mapped. We will color the map based on a few quantiles calculated for the change in construction area. It suffices to get some nice colors and a few more geographical layers so that the map looks more informative.

{% highlight r %}
library(RColorBrewer)
my_colours <- brewer.pal(8, "Blues")

library(latticeExtra)
p1 <- spplot(map_data, "diffCut",
             main="Change in construction area (ha)",
             col.regions = my_colours,
             lwd=0.5, col="grey",
             sp.layout = list(highways, col="red", lwd=1))
p1 <- p1 + layer(sp.polygons(map_provinces, col="black", lwd=0.8))
plot(p1)
{% endhighlight %}

This code should give you a map looking similarly to this one.

![Change in construction area (in hectares) in Belgium between 1983 and 2015]({{ site.url }}/assets/images/construction.png)

Highest construction activity seems to be concentrated in highly populated areas such as Antwerp, Genk, Charleroi and Namur. In the low populated Luxemburg province this activity seems to be on the contrary very low. This is probably not very surprising.

What looks more interesting is that there is also notable increase especially along the highway E17 between Ghent and Antwerp and along the area covering Antwerp and Limburg provinces. Visually, it really looks like most construction activity outside major cities took place along E17, E313, E314 and E34 highways in the Flanders region. Strangely, the area along the E411 highway connecting Brussels and Luxembourg does not seem to attract too many construction projects.

[spf]: http://statbel.fgov.be/fr/statistiques/opendata/home/
[ulg-atlas]: http://www.atlas-belgique.be/cms/index.php?page=fonds
[belgium-stat]: https://github.com/jwijffels/BelgiumStatistics
