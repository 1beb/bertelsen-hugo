---
author: "Brandon Erik Bertelsen"
title: "Montreal FSA Scraping Part Dieux"
date: "2016-08-15"
categories:
- R
- Maps
---

Although we were able to scrape from the web the FSA we wanted, it was unfortunately not a complete list. Instead, let's try another route using some data that's been crowdsourced, namely the geocoder.ca dataset or a subset provided by aggdata (as the geocoder.ca table is 50mbs and I don't need that level of accuracy).

Let's install some packages first. You may need to install some system files for this to work:

```shell
sudo apt-get install libgeos-dev libgdal1-dev libproj-dev
```

Now we can install the appropriate packages in R, if they aren't already:

```R
install.packages("maptools","rgeos","rgdal")  
```

Now we can run a short script to find the FSA's within the boundaries of our economic region.

```R
library(ggplot2)  
library(maptools)  
library(rgeos)  
library(rgdal)

# Canadian shapefiles
# select your own (https://goo.gl/ztd9HY) or 
# economic regions (http://goo.gl/YiHMhY) direct download
shp <- file.path("path/to/ger_000b11a_e.shp")  
map <- readShapePoly(shp, proj4string = CRS("+init=epsg:25832"))  
sel <- map$ERNAME == "Montérégie"

# https://www.aggdata.com/download_sample.php?file=ca_postal_codes.csv
fsa_db <- read.csv("https://goo.gl/q97K3L", fileEncoding = "Windows-1252") setNames(fsa_db, c("fsa","place","province","lat","long"))

region <- map[sel,]  
points <- data.frame(long=as.numeric(fsa_db$long),  
                     lat =as.numeric(fsa_db$lat),
                     id  =fsa_db$fsa, stringsAsFactors=F)

# We know that Monteregie is in JXX FSAs
points$yes <- substr(points$id,0,1) == "J"  
points <- points[points$yes,]

# Identify if FSA Long/Lat is within Economic Region

listing <- list()  
for(i in 1:nrow(points)) {  
  p1 <- points[i,1:2]
  sp2   <- SpatialPoints(p1,proj4string=CRS(proj4string(region)))
  listing[[i]] <- gContains(region,sp2)
}

points <- points[listing %>% unlist,]

ggplot(region, aes(x=long,y=lat,group=group))+  
  geom_polygon(fill="lightgreen")+
  geom_path(colour="grey50") +
  geom_point(data=points,aes(x=long,y=lat,group=NULL, color=id), size=1) +
  coord_fixed() + theme(legend.position = "none")
```