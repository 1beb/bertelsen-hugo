---
author: "Brandon Erik Bertelsen"
title: "Boostrap with logistic regression"
date: "2015-08-01"
categories:
- R
---

A quick example of bootstraping a logistic regression. Nothing special here, example could be extended to any other type of model that has a `coef()` method.

```R
library(boot)  
logit_test <- function(d,indices) {  
d <- d[indices,]  
fit <- glm(your ~ formula, data = d, family = "binomial")  
return(coef(fit))  
}

boot_fit <- boot(  
   data = your_data, 
   statistic = logit_test, 
   R = 1e5
) 
```