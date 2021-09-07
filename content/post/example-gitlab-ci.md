---
author: "Brandon Erik Bertelsen"
title: "Example gitlab-ci.yml for R Projects"
date: "2016-06-27"
categories:
- R
- Docker
---

Most people are running open source projects, so they can easily use github and travis for free. I don't have that luxury, but gitlab has really caught my attention lately. Combining gitlab, gitlab runner with docker makes things very straightforward. Here is an example `.gitlab-ci.yml` file that you would need to include in the base directory of your R project to have it run R CMD check and run testthat tests.

```yaml
image: rocker/rstudio  
test:  
   script:
    - R -e 'install.packages(c("needed here"))'
    - R CMD build . --no-build-vignettes --no-manual
    - PKG_FILE_NAME=$(ls -1t *.tar.gz | head -n 1)
    - R CMD check "${PKG_FILE_NAME}" --no-build-vignettes --no-manual
    - R -e 'devtools::test()'
```