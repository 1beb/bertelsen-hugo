---
author: "Brandon Erik Bertelsen"
title: "Rcpp and roxygem2"
date: "2016-08-28"
categories:
- R
- Rcpp
---

Instructions for using Rcpp and Roxygen2 together. This assumes that Roxygen2 is managing your namespace:

DESCRIPTION:

- In your DESCRIPTION file, add the line `LinkingTo: Rcpp`
- Also ensure that you import Rcpp, `Imports: Rcpp` along with all the other packages are imported in your namespace.

Package Documentation:

- Your package documentation (by convention in `zzz.R` or `package.R`) you should have the following:

```r
#' your_package
#' 
#' Description of your package
#' 
#' @docType package
#' @author you <youremail>
#' @import Rcpp another_package another
#' @importFrom Rcpp evalCpp
#' @useDynLib your_package
#' @name your_package
NULL  
```

Functions:

- Add `*.cpp` files to `/src` directory.
- Remember to use `//'` as your comment so that Roxygen2 picks it up appropriately.
- Remember to include `[[Rcpp:export]]` so that it is picked up by `Rcpp::compileAttributes()` when you build and reload or run `roxygenize()` manually as per your project options

```cpp
#include <Rcpp.h>

using namespace Rcpp;


//' Leading NA
//' 
//' This function returns a logical vector identifying if 
//' there are leading NA, marking the leadings NA as TRUE and
//' everything else as FALSE.
//'
//' @param x An integer vector
//' @export
// [[Rcpp::export]]
LogicalVector leading_na(IntegerVector x) {  
  int n = x.size();
  LogicalVector leading_na(n);

  int i = 0;
  while((i < n) &&(x[i] == NA_INTEGER)) {
    leading_na[i] = TRUE;
    i++;
  }
  return leading_na;
}
```