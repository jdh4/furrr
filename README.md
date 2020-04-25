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

# mclapply

This uses a forking approach as opposed to sockets (like snow):

```
‘mclapply’ is a parallelized version of ‘lapply’, it returns a
     list of the same length as ‘X’, each element of which is the
     result of applying ‘FUN’ to the corresponding element of ‘X’.

     It relies on forking and hence is not available on Windows unless
     ‘mc.cores = 1’.
```

```
> library(parallel)
> system.time(l1 <- unlist(mclapply(1:10, function(x) {rnorm(1000000)}, mc.cores=1)))
   user  system elapsed 
  0.708   0.070   0.794 
> system.time(l1 <- unlist(mclapply(1:10, function(x) {rnorm(1000000)}, mc.cores=2)))
   user  system elapsed 
  0.771   0.307   0.638 
> system.time(l1 <- unlist(mclapply(1:10, function(x) {rnorm(1000000)}, mc.cores=4)))
   user  system elapsed 
  0.797   0.371   0.425 
> system.time(l1 <- unlist(mclapply(1:10, function(x) {rnorm(1000000)}, mc.cores=8)))
   user  system elapsed 
  0.797   0.405   0.294 
> system.time(l1 <- unlist(mclapply(1:10, function(x) {rnorm(1000000)}, mc.cores=16)))
   user  system elapsed 
  0.802   0.463   0.293 
```

```
> system.time(l1 <- unlist(mclapply(X=13:23, function(x) {rnorm(1000000);print(x);}, mc.cores=1)))
[1] 13
[1] 14
[1] 15
[1] 16
[1] 17
[1] 18
[1] 19
[1] 20
[1] 21
[1] 22
[1] 23
   user  system elapsed 
  0.685   0.041   0.727 
```

```
> l1 <- unlist(mclapply(1:10, function(x) {rnorm(1000000)}, mc.cores=1))
> str(l1)
 num [1:10000000] -0.473 -3.05 -0.935 -3.109 -0.185 ...
> length(l1)
[1] 10000000
> is.vector(l1) && is.atomic(l1)
[1] TRUE
```

# parallel (snow)

`snow` and `multicore` were subsumed into parallel.

```
> cl <- makeCluster(4)
> clusterCall(cl, function() {Sys.info()["nodename"]})
[[1]]
 nodename 
"adroit4" 

[[2]]
 nodename 
"adroit4" 

[[3]]
 nodename 
"adroit4" 

[[4]]
 nodename 
"adroit4" 

> clusterApply(cl, c('A', 'B', 'C'), function(x) {paste("Executor", x, "ready.")})
[[1]]
[1] "Executor A ready."

[[2]]
[1] "Executor B ready."

[[3]]
[1] "Executor C ready."

> stopCluster(cl)
```

# foreach and doParallel

```
> install.packages(c("foreach", "doParallel"))
> library(foreach)
> library(doParallel)
Loading required package: iterators
Loading required package: parallel
> registerDoParallel(cores=4)
> system.time(l4p <- foreach(i=1:4, .combine='c') %dopar% rnorm(250000000))
   user  system elapsed 
 75.338  29.957  41.277 
> stopImplicitCluster()
> registerDoParallel(cores=8)
> system.time(l4p <- foreach(i=1:8, .combine='c') %dopar% rnorm(125000000))
   user  system elapsed 
 80.042  41.608  33.690
> stopImplicitCluster()
```

# Documentation

> ?parallel
> help(package=parallel)

> lsf.str("parallel")
> lsf.str("package:parallel")
clusterApply : function (cl = NULL, x, fun, ...)  
clusterApplyLB : function (cl = NULL, x, fun, ...)  
clusterCall : function (cl = NULL, fun, ...)  
clusterEvalQ : function (cl = NULL, expr)  
clusterExport : function (cl = NULL, varlist, envir = .GlobalEnv)  
clusterMap : function (cl = NULL, fun, ..., MoreArgs = NULL, RECYCLE = TRUE, SIMPLIFY = FALSE, 
    USE.NAMES = TRUE, .scheduling = c("static", "dynamic"))  
clusterSetRNGStream : function (cl = NULL, iseed = NULL)  
clusterSplit : function (cl = NULL, seq)  
detectCores : function (all.tests = FALSE, logical = TRUE)  
getDefaultCluster : function ()  
makeCluster : function (spec, type = getClusterOption("type"), ...)  
makeForkCluster : function (nnodes = getOption("mc.cores", 2L), ...)  
makePSOCKcluster : function (names, ...)  
mc.reset.stream : function ()  
mcaffinity : function (affinity = NULL)  
mccollect : function (jobs, wait = TRUE, timeout = 0, intermediate = FALSE)  
mclapply : function (X, FUN, ..., mc.preschedule = TRUE, mc.set.seed = TRUE, mc.silent = FALSE, 
    mc.cores = getOption("mc.cores", 2L), mc.cleanup = TRUE, mc.allow.recursive = TRUE, 
    affinity.list = NULL)  
mcMap : function (f, ...)  
mcmapply : function (FUN, ..., MoreArgs = NULL, SIMPLIFY = TRUE, USE.NAMES = TRUE, 
    mc.preschedule = TRUE, mc.set.seed = TRUE, mc.silent = FALSE, mc.cores = getOption("mc.cores", 
        2L), mc.cleanup = TRUE, affinity.list = NULL)  
mcparallel : function (expr, name, mc.set.seed = TRUE, silent = FALSE, mc.affinity = NULL, 
    mc.interactive = FALSE, detached = FALSE)  
nextRNGStream : function (seed)  
nextRNGSubStream : function (seed)  
parApply : function (cl = NULL, X, MARGIN, FUN, ..., chunk.size = NULL)  
parCapply : function (cl = NULL, x, FUN, ..., chunk.size = NULL)  
parLapply : function (cl = NULL, X, fun, ..., chunk.size = NULL)  
parLapplyLB : function (cl = NULL, X, fun, ..., chunk.size = NULL)  
parRapply : function (cl = NULL, x, FUN, ..., chunk.size = NULL)  
parSapply : function (cl = NULL, X, FUN, ..., simplify = TRUE, USE.NAMES = TRUE, chunk.size = NULL)  
parSapplyLB : function (cl = NULL, X, FUN, ..., simplify = TRUE, USE.NAMES = TRUE, chunk.size = NULL)  
pvec : function (v, FUN, ..., mc.set.seed = TRUE, mc.silent = FALSE, mc.cores = getOption("mc.cores", 
    2L), mc.cleanup = TRUE)  
setDefaultCluster : function (cl = NULL)  
splitIndices : function (nx, ncl)  
stopCluster : function (cl = NULL)
