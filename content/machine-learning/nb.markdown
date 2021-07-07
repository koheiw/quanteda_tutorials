---
title: Naive Bayes classifier
weight: 10
draft: false
---

Naive Bayes is a supervised model usually used to classify documents into two or more categories. We train the classifier using class labels attached to documents, and predict the most likely class(es) of new unlabelled documents.


```r
require(quanteda)
require(quanteda.textmodels)
require(caret)
```

`data_corpus_moviereviews` from the **quanteda.textmodels** package contains 2000 movie reviews classified either as "positive" or "negative".


```r
corp_movies <- data_corpus_moviereviews
summary(corp_movies, 5)
```

```
## Corpus consisting of 2000 documents, showing 5 documents:
## 
##             Text Types Tokens Sentences sentiment   id1   id2
##  cv000_29416.txt   354    841         9       neg cv000 29416
##  cv001_19502.txt   156    278         1       neg cv001 19502
##  cv002_17424.txt   276    553         3       neg cv002 17424
##  cv003_12683.txt   313    555         2       neg cv003 12683
##  cv004_12641.txt   380    841         2       neg cv004 12641
```

The variable "Sentiment" indicates whether a movie review was classified as positive or negative. In this example we use 1500 reviews as the training set and build a Naive Bayes classifier based on this subset. In a second step, we predict the sentiment for the remaining reviews (our test set).

Since the first 1000 reviews are negative and the remaining reviews are classified as positive, we need to draw a random sample of the documents.


```r
# generate 1500 numbers without replacement
set.seed(300)
id_train <- sample(1:2000, 1500, replace = FALSE)
head(id_train, 10)
```

```
##  [1]  590  874 1602  985 1692  789  553 1980 1875 1705
```

```r
# create docvar with ID
corp_movies$id_numeric <- 1:ndoc(corp_movies)

# tokenize texts
toks_movies <- tokens(corp_movies, remove_punct = TRUE, remove_number = TRUE) %>% 
               tokens_remove(pattern = stopwords("en")) %>% 
               tokens_wordstem()
dfmt_movie <- dfm(toks_movies)

# get training set
dfmat_training <- dfm_subset(dfmt_movie, id_numeric %in% id_train)

# get test set (documents not in id_train)
dfmat_test <- dfm_subset(dfmt_movie, !id_numeric %in% id_train)
```

Next we train the naive Bayes classifier using `textmodel_nb()`.


```r
tmod_nb <- textmodel_nb(dfmat_training, dfmat_training$sentiment)
summary(tmod_nb)
```

```
## 
## Call:
## textmodel_nb.dfm(x = dfmat_training, y = dfmat_training$sentiment)
## 
## Class Priors:
## (showing first 2 elements)
## neg pos 
## 0.5 0.5 
## 
## Estimated Feature Scores:
##         plot      two      teen     coupl       go    church     parti
## neg 0.002579 0.002318 0.0002870 0.0007157 0.002663 8.719e-05 0.0002652
## pos 0.001507 0.002338 0.0001656 0.0005456 0.002348 8.768e-05 0.0002728
##         drink     drive      get     accid      one       guy       die
## neg 1.199e-04 0.0003052 0.004486 9.445e-05 0.007389 0.0014458 0.0005486
## pos 9.417e-05 0.0002630 0.003783 1.851e-04 0.007355 0.0009937 0.0005488
##     girlfriend   continu      see     life  nightmar      deal    watch
## neg  0.0003124 0.0003161 0.002557 0.001435 0.0001199 0.0004323 0.001642
## pos  0.0002338 0.0003215 0.003020 0.002497 0.0001202 0.0005196 0.001539
##         movi     sorta     find   critiqu mind-fuck   generat     touch
## neg 0.010117 1.090e-05 0.001453 9.445e-05 3.633e-06 0.0002652 0.0002289
## pos 0.007657 1.624e-05 0.001630 8.443e-05 3.247e-06 0.0002923 0.0004449
##          cool      idea
## neg 0.0003052 0.0008210
## pos 0.0002273 0.0005845
```

Naive Bayes can only take features into consideration that occur both in the training set and the test set, but we can make the features identical using `dfm_match()`


```r
dfmat_matched <- dfm_match(dfmat_test, features = featnames(dfmat_training))
```

Let's inspect how well the classification worked.


```r
actual_class <- dfmat_matched$sentiment
predicted_class <- predict(tmod_nb, newdata = dfmat_matched)
tab_class <- table(actual_class, predicted_class)
tab_class
```

```
##             predicted_class
## actual_class neg pos
##          neg 213  45
##          pos  37 205
```

From the cross-table we see that the number of false positives and false negatives is similar. The classifier made mistakes in both directions, but does not seem to over- or underestimate one class.

We can use the function `confusionMatrix()` from the **caret** package to assess the performance of the classification.


```r
confusionMatrix(tab_class, mode = "everything")
```

```
## Confusion Matrix and Statistics
## 
##             predicted_class
## actual_class neg pos
##          neg 213  45
##          pos  37 205
##                                           
##                Accuracy : 0.836           
##                  95% CI : (0.8006, 0.8674)
##     No Information Rate : 0.5             
##     P-Value [Acc > NIR] : <2e-16          
##                                           
##                   Kappa : 0.672           
##                                           
##  Mcnemar's Test P-Value : 0.4395          
##                                           
##             Sensitivity : 0.8520          
##             Specificity : 0.8200          
##          Pos Pred Value : 0.8256          
##          Neg Pred Value : 0.8471          
##               Precision : 0.8256          
##                  Recall : 0.8520          
##                      F1 : 0.8386          
##              Prevalence : 0.5000          
##          Detection Rate : 0.4260          
##    Detection Prevalence : 0.5160          
##       Balanced Accuracy : 0.8360          
##                                           
##        'Positive' Class : neg             
## 
```

{{% notice note %}}
Precision, recall and the F1 score are frequently used to assess the classification performance. Precision is measured as `TP / (TP + FP)`, where `TP` are the number of true positives and  `FP`  the false positives. Recall divides the true positives by the sum of true positives and false negatives `TP / (TP + FN)`. Finally, the F1 score is a harmonic mean of precision and recall `2 * (Precision * Recall) / (Precision + Recall)`.
{{% /notice %}}

{{% notice ref %}}
- Jurafsky, Daniel, and James H. Martin. 2018. [_Speech and Language Processing. An Introduction to Natural Language Processing, Computational Linguistics, and Speech Recognition_](https://web.stanford.edu/~jurafsky/slp3/4.pdf). Draft of 3rd edition, September 23, 2018 (Chapter 4). 
{{% /notice%}}
