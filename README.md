# furrr

This code takes 18 seconds using cpus-per-task=4 compared to 41 seconds with 1 core:

```R
library(furrr)
plan(multiprocess)

x <- rnorm(10000000)
print(is.vector(x))
mymap <- future_map(x*x, sqrt)
print(mean(x))
```

```
library(furrr)
plan(multiprocess)
library(tibble)
library(dplyr)

mm <- tibble(K=c(1,2,3)) %>% mutate(x=future_map_dbl(K, sqrt))
mm
# A tibble: 3 x 2
      K     x
  <dbl> <dbl>
1     1  1   
2     2  1.41
3     3  1.73
```
