---
title: Regularized regression classifier
weight: 15
draft: false
---

{{% author %}}By Frederik Hjorth{{% /author %}} 


Regularized regression is a classification technique where the category of interest is regressed on text features using a penalized form of regression where parameter estimates are biased towards zero. Here we will be using a specific type of regularized regression, the Least Absolute Shrinkage and Selection Operator (or simply LASSO), but its main alternative, ridge regression, is conceptually very similar. 

In the lasso estimator, the degree of penalization is determined by the regularization parameter `lambda`. We use the cross-validation function available in the **glmnet** package to select the optimal value for `lambda`. 
We train the classifier using class labels attached to documents, and predict the most likely class(es) of new unlabelled documents. Although regularized regression is not part of the **quanteda.textmodels** package, the functions for regularized regression from the **glmnet** package can be easily worked in to a quanteda workflow.


```r
require(quanteda)
require(quanteda.textmodels)
require(glmnet)
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

The variable "Sentiment" indicates whether a movie review was classified as positive or negative. In this example we use 1500 reviews as the training set and build a regularized regression classifier based on this subset. In a second step, we predict the sentiment for the remaining reviews (our test set).

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

# get training set
dfmat_training <- corpus_subset(corp_movies, id_numeric %in% id_train) %>%
    dfm(remove = stopwords("en"), stem = TRUE)

# get test set (documents not in id_train)
dfmat_test <- corpus_subset(corp_movies, !(id_numeric %in% id_train)) %>%
    dfm(remove = stopwords("en"), stem = TRUE)
```

Next we choose `lambda` using `cv.glmnet` from the **glmnet** package. `cv.glmnet` requires an input matrix X and a response vector Y. For the input matrix, we use the training set converted to a sparse matrix. For the response vector, we use a dichotomous indicator of review sentiment in the training set with negative reviews coded as 1 and positive reviews as 0. 

`cv.glmnet` uses cross-validation to select the value of `lambda` yielding the smallest classification error. Setting `alpha = 1` selects the lasso estimator. Setting `nfold = 5` sets cross-validation across 5 'folds', i.e. partitions of the training set.


```r
lasso <- cv.glmnet(x = dfmat_training,
                   y = as.integer(dfmat_training$sentiment == "pos"),
                   alpha = 1,
                   nfold = 5,
                   family = "binomial")
```

As an initial evaluation of the model, let's take a look at the most predictive features. We begin by obtaining the best value of `lambda`:


```r
index_best <- which(lasso$lambda == lasso$lambda.min)
beta <- lasso$glmnet.fit$beta[, index_best]
```

We can now look at the most predictive features for this value of `lambda`:


```r
dat_pred <- data.frame(
    coef = as.numeric(beta),
    word = names(beta))
dat_pred <- dat_pred[order(-dat_pred$coef),]

head(dat_pred,10)
```

```
##            coef       word
## 22970 1.2112800        250
## 20530 1.1208702  chopsocki
## 20013 1.0245674   flammabl
## 20405 0.8521184 neccessari
## 23379 0.8435197    maniaci
## 5484  0.8128065     finest
## 20525 0.8024634   standoff
## 7540  0.8011388  breathtak
## 22917 0.7999109     haywir
## 19727 0.7933393   gingrich
```

`predict.glmnet` can only take features into consideration that occur both in the training set and the test set, but we can make the features identical by using `dfm_match()`.


```r
dfmat_matched <- dfm_match(dfmat_test, features = featnames(dfmat_training))
```

Next we obtain predicted probabilities for each review in the test set.


```r
preds <- predict(lasso, dfmat_matched, type = "response", s = lasso$lambda.min)
```

Let's inspect how well the classification worked.


```r
actual_class <- as.integer(dfmat_matched$sentiment == "pos")
predicted_class <- as.integer(predict(lasso, dfmat_matched, type = "class"))
tab_class <- table(actual_class, predicted_class)
tab_class
```

```
##             predicted_class
## actual_class   0   1
##            0 199  59
##            1  33 209
```

From the cross-table we see that the model slightly underpredicts negative reviews, i.e. produces slightly more false negatives than false positives, but most reviews are correctly predicted. 

We can use the function `confusionMatrix()` from the **caret** package to quantify the performance of the classification.


```r
confusionMatrix(tab_class, mode = "everything")
```

```
## Confusion Matrix and Statistics
## 
##             predicted_class
## actual_class   0   1
##            0 199  59
##            1  33 209
##                                          
##                Accuracy : 0.816          
##                  95% CI : (0.7792, 0.849)
##     No Information Rate : 0.536          
##     P-Value [Acc > NIR] : < 2.2e-16      
##                                          
##                   Kappa : 0.6328         
##                                          
##  Mcnemar's Test P-Value : 0.009149       
##                                          
##             Sensitivity : 0.8578         
##             Specificity : 0.7799         
##          Pos Pred Value : 0.7713         
##          Neg Pred Value : 0.8636         
##               Precision : 0.7713         
##                  Recall : 0.8578         
##                      F1 : 0.8122         
##              Prevalence : 0.4640         
##          Detection Rate : 0.3980         
##    Detection Prevalence : 0.5160         
##       Balanced Accuracy : 0.8188         
##                                          
##        'Positive' Class : 0              
## 
```

{{% notice note %}}
Precision, recall and the F1 score are frequently used to assess the classification performance. Precision is measured as `TP / (TP + FP)`, where `TP` are the number of true positives and  `FP`  the false positives. Recall divides the false positives by the sum of true positives and false negatives `TP / (TP + FN)`. Finally, the F1 score is a harmonic mean of precision and recall `2 * (Precision * Recall) / (Precision + Recall)`.
{{% /notice %}}

{{% notice ref %}}
- Jurafsky, Daniel, and James H. Martin. 2018. [_Speech and Language Processing. An Introduction to Natural Language Processing, Computational Linguistics, and Speech Recognition_](https://web.stanford.edu/~jurafsky/slp3/4.pdf). Draft of 3rd edition, September 23, 2018 (Chapter 4). 
{{% /notice%}}