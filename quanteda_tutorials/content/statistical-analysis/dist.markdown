---
title: Document/feature similarity
weight: 30
chapter: false
draft: false
---


```r
require(quanteda)
```

`textstat_dist()` calcuates similarites of documents or features for various measures. Its output is compatible with R's `dist()`, so hierachical clustering can be perfromed without any transformation.


```r
inaug_toks <- tokens(data_corpus_inaugural)
inaug_dfm <- dfm(inaug_toks, remove = stopwords('en'))
dist <- textstat_dist(inaug_dfm)
clust <- hclust(dist)
plot(clust, xlab = "Distance", ylab = NULL)
```

<img src="/statistical-analysis/dist_files/figure-html/unnamed-chunk-2-1.svg" width="960" />


