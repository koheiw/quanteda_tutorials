---
title: Compound multi-word expressions
weight: 20
draft: false
---

We can compound multi-word expressions through collocation analysis. In this example, we identify sequences of capitalized words and compound them as proper names, which are important linguistic features of newspaper articles.


```r
require(quanteda)
require(quanteda.textstats)
require(quanteda.corpora)
options(width = 110)
```

This corpus contains 6,000 Guardian news articles from 2012 to 2016.


```r
corp_news <- download("data_corpus_guardian")
```





We remove punctuation marks and symbols in `tokens()` and stopwords in `tokens_remove()` with `padding = TRUE` to keep the original positions of tokens. 


```r
toks_news <- tokens(corp_news, remove_punct = TRUE, remove_symbols = TRUE, pading = TRUE) %>% 
    tokens_remove(stopwords("en"), padding = TRUE)
```

```
## Warning: pading argument is not used.
```

One of the most common type of multi-word expressions is proper names, which we can select simply based on capitalization in English texts.


```r
toks_news_cap <- tokens_select(toks_news, 
                               pattern = "^[A-Z]",
                               valuetype = "regex",
                               case_insensitive = FALSE, 
                               padding = TRUE)

tstat_col_cap <- textstat_collocations(toks_news_cap, min_count = 10, tolower = FALSE)
head(tstat_col_cap, 20)
```

```
##           collocation count count_nested length    lambda         z
## 1       David Cameron   861            0      2  8.159232 147.27725
## 2        Donald Trump   774            0      2  8.326850 122.69417
## 3      George Osborne   364            0      2  8.660679 107.61955
## 4     Hillary Clinton   527            0      2  9.109436 102.41482
## 5            New York  1016            0      2 10.447749 100.26546
## 6       Islamic State   330            0      2  9.802083  98.14442
## 7         White House   479            0      2  9.921867  96.17493
## 8      European Union   351            0      2  8.261458  94.76074
## 9       Jeremy Corbyn   244            0      2  8.729428  90.79316
## 10      Boris Johnson   245            0      2  9.664060  84.76700
## 11     Bernie Sanders   394            0      2  9.901301  84.61682
## 12 Guardian Australia   237            0      2  6.329359  83.70146
## 13   Northern Ireland   205            0      2  9.883087  83.25673
## 14        Home Office   216            0      2  9.690405  78.66972
## 15        Ed Miliband   174            0      2  9.867962  78.44764
## 16           Ted Cruz   417            0      2 10.756268  77.87774
## 17       Barack Obama   344            0      2  9.775204  77.72635
## 18       South Africa   172            0      2  7.568553  77.61108
## 19     South Carolina   271            0      2  9.404656  76.80054
## 20       Black Friday   190            0      2  8.458893  76.79848
```

We will only compound strongly associated multi-word expressions here by subsetting `tstat_col_cap` with the z-score (`z > 3`).


```r
toks_comp <- tokens_compound(toks_news, pattern = tstat_col_cap[tstat_col_cap$z > 3], 
                             case_insensitive = FALSE)
kw_comp <- kwic(toks_comp, pattern = c("London_*", "British_*"))
head(kw_comp, 10)
```

```
## Keyword-in-context with 10 matches.                                                                                      
##     [text9204, 351]         however researchers publishing | British_Medical_Journal |
##   [text150582, 329] overseas territories including Bermuda | British_Virgin_Islands  |
##   [text150582, 584]                        included Panama | British_Virgin_Islands  |
##   [text120395, 971]        John_Longworth director general |    British_Chambers     |
##     [text3527, 155]                           sacked mayor |  London_Boris_Johnson   |
##  [text145860, 1584]              rental period Association |    British_Insurers     |
##   [text148174, 391]                 EZY5258 Rome Fiumicino |     London_Gatwick      |
##    [text109224, 17]                            Coast range |    British_Columbia     |
##   [text109224, 101]                                  coast |    British_Columbia     |
##   [text109224, 194]                          Alberta coast |    British_Columbia     |
##                             
##  found drop heart           
##  Cayman_Islands legislation 
##  published commission       
##  Commerce said businesses   
##  Blair told inquiry         
##  ABI says insurers becoming 
##  29 March delayed           
##  Hanging nearly             
##  Today however Chief Na'Moks
##  plan carry
```
