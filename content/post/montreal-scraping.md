---
author: "Brandon Erik Bertelsen"
title: "Montreal FSA Scraping"
date: "2016-08-14"
categories:
- R
- Maps
- Scraping
---

There's no official database that I can find to define clearly some of the economic areas in Montreal, at least none that are free. However, wikipedia does seem to be rather well organized in this regard. Small scale scraping to identify FSAs for a particular locale, in the example below, Montérégie or "Rive-Sud" an affluent part of the greater Montreal area.

```R
library(rvest)  
links <- read_html("https://en.wikipedia.org/wiki/South_Shore_(Montreal)") %>%  
  html_node(css = ".column-count-3") %>% 
  html_nodes("a") %>% 
  html_attr("href")

links <- paste0("http://en.wikipedia.org",links)  
listing <- list()  
for(link in links) {  
  listing[[link]] <- read_html(link) %>% html_node(".adr") %>% html_text()
}
```

So far so good, but it looks like a bit of extra cleaning will be required.


```R
> listing %>% unlist() %>% as.character %>% strsplit(",") %>% unlist()
 [1] "J3G to J3H"  "J4B"         "J4W to J4Z"  "J5R"         "J3L"        
 [6] "J3L"         "J6J"         " J6K"        "J5B"         "J0L1B0"     
[11] NA            "J5R"         "J3Y"         " J3Z"        " J4G to J4N"
[16] " J4T"        " J4V"        "J4V"         "J3Y"         " J3Z"       
[21] " J4T"        NA            NA            "J3G 6N9"     "J3H"        
[26] "J3H 2M6"     "J3L"         "J0L 1N0"     "J3N"         "J3V"        
[31] "J5A"         "J0L 2A0"     "J3E"         "J5C"         "J4P"        
[36] " J4R"        " J4S"        "J3L 6Z5"     "J0L 2K0"     "J3X"     
trim <- function (x) gsub("^\\s+|\\s+$", "", x)  
listing %>%  
  unlist() %>% 
  as.character %>% 
  strsplit(",") %>% 
  unlist() %>% 
  strsplit("to") %>% 
  unlist %>% 
  na.omit() %>% 
  trim %>% 
  substr(start = 0, stop = 3)
```

And we have a list of FSAs for Montérégie pulled from what we hope is a good resource:

    J3G  
    J3H  
    J4B  
    J4W  
    J4Z  
    J5R  
    J3L  
    J3L  
    J6J  
    J6K  
    J5B  
    J0L  
    J5R  
    J3Y  
    J3Z  
    J4G  
    J4N  
    J4T  
    J4V  
    J4V  
    J3Y  
    J3Z  
    J4T  
    J3G  
    J3H  
    J3H  
    J3L  
    J0L  
    J3N  
    J3V  
    J5A  
    J0L  
    J3E  
    J5C  
    J4P  
    J4R  
    J4S  
    J3L  
    J0L  
    J3X