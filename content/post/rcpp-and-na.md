---
author: "Brandon Erik Bertelsen"
title: "Rcpp example using NA"
date: "2016-08-28"
categories:
- R
- Rcpp
---

Dealing with NA in Rcpp can be a bit tricky to sort out, but the speed boosts are incredible and if you're dealing with large data - worth the time to figure it out. But usage cases are important to consider in the example below you can see that vectorized operations are still very competitive and very concise. Rcpp doesn't shine so much in this example, but it can when you have a problem that requires a waterfall of results (one result relies on the next and so on).

Here's a basic function that takes NA values into account and basically converts anything < 0 to -1 and anything > 0 to +1.

```r
library(Rcpp)  
library(microbenchmark)

signR <- function(x) {  
  for(i in 1:length(x)) {
    if(is.na(x[i])) {
      next
    }
    if (x[i] > 0) {
      x[i] <- 1
    } else {
      x[i] <- -1
    }
  }
  return(x)
}

signR2 <- function(x) {  
  x[x > 0] <- 1
  x[x < 0] <- -1
  x
}

cppFunction('  
NumericVector signC(NumericVector x) {  
int n = x.size();  
NumericVector out(n);

for(int i = 0; i<n; ++i ) {  
      if (NumericVector::is_na(x[i])) {
        out[i] = NA_REAL;
      } else if (x[i] > 0) {
        out[i] = 1;
      } else {
        out[i] = -1;
      }
    }
  return out;
}')


test <- sample(c(rnorm(25),NA),100000, replace = TRUE)

microbenchmark(  
  signR(test),
  signR2(test),
  signC(test)
)

# Unit: milliseconds
#         expr        min         lq       mean     median         uq        max neval
#  signR(test) 197.676873 201.344711 205.951881 202.437384 204.712847 317.973634   100
# signR2(test)   4.267238   4.335184   4.424379   4.412983   4.467380   4.869512   100
#  signC(test)   2.095438   2.120071   2.185537   2.160100   2.219835   2.676364   100

all.equal(signR(test),signR2(test),signC(test))  
# TRUE
```