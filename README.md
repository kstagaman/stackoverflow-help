R file size issues
================
1/8/2021

Below reads in a phyloseq object with a very large community matrix (47
x 1,060,929)

``` r
library(phyloseq)

obj.size <- function(x) {paste("Object size:", format(object.size(x), units = "auto"))}
f.size <- function(x) {paste("File size:", round(file.size(x) / 1024^2, 1), "Mb")}

ps <- readRDS("example_phyloseq.rds")
```

Next, I create a distance matrix to determine community dissimilarities
between samples It should result in a 47 x 47 matrix

``` r
dist.mat <- phyloseq::distance(ps, method = "bray") # creating a distance matrix
attributes(dist.mat)$Size
```

    ## [1] 47

Okay, as expected, but…

``` r
cat(obj.size(dist.mat), sep = "\n")
```

    ## Object size: 453.3 Mb

This is oddly big for a distance matrix of this size (like orders of
magnitude larger)

After investigating I realize that this is because
`attributes(dist.mat)$call` is huge, because it includes every single
value from the original community matrix.

``` r
dist.mat.small <- dist.mat
attributes(dist.mat.small)$call <- NULL
cat(obj.size(dist.mat.small), sep = "\n")
```

    ## Object size: 12.5 Kb

Okay, so that’s looking good. Much closer to what I’d expect for the
size. But…when I try to save…

``` r
saveRDS(dist.mat, file = "distance_matrix_original.rds")
saveRDS(dist.mat.small, file = "distance_matrix_small.rds")

cat(f.size("distance_matrix_original.rds"), sep = "\n")
```

    ## File size: 112.5 Mb

``` r
cat(f.size("distance_matrix_small.rds"), sep = "\n")
```

    ## File size: 112.5 Mb

What’s happening here? Why are the file sizes the same for objects of
very different size?
