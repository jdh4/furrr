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

The below will create 5 vectors with n=10 elements, mean=<the piped input value>, sd=1 then compute the mean of each vector:

```
1:5 %>%
+   future_map(rnorm, n=10) %>%
+   future_map_dbl(mean)
[1] 1.539254 1.736080 2.341536 3.895655 4.642486
```

# Available packages and currently loaded packages

```
> library()$results[,1]
 [1] "base"         "boot"         "class"        "cluster"      "codetools"   
 [6] "compiler"     "datasets"     "foreign"      "graphics"     "grDevices"   
[11] "grid"         "HiddenMarkov" "KernSmooth"   "lattice"      "MASS"        
[16] "Matrix"       "methods"      "mgcv"         "nlme"         "nnet"        
[21] "parallel"     "rpart"        "spatial"      "splines"      "stats"       
[26] "stats4"       "survival"     "tcltk"        "tools"        "utils"
```

or just use `library()`.

To see currently loaded packages():

```
$ R
> (.packages())
[1] "stats"     "graphics"  "grDevices" "utils"     "datasets"  "methods"  
[7] "base"     
```
