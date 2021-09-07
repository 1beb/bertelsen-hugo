---
author: "Brandon Erik Bertelsen"
title: "Expand strsplit to data.frame"
date: "2015-06-15"
categories:
- R
- Rcpp
---

Take a vector, with some seperator (i.e., ;), vec and expand it into a data.frame, filling in NA positions:

```R
vec <- strsplit(as.character(vec), ";")  
df <- rbind.fill(  
  lapply(
    tmp, function(x) data.frame(t(x))
  )
)
df <- as.data.frame(df)  
```