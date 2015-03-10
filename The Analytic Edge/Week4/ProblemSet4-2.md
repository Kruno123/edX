ProblemSet4-2
========================================================
PROBLEM 1.1 - PREDICTING B OR NOT B  (1 point possible)
--------------------------------------------------------
Let's warm up by attempting to predict just whether a letter is B or not. To begin, load the file letters_ABPR.csv into R, and call it letters. Then, create a new variable isB in the dataframe, which takes the value "yes" if the observation corresponds to the letter B, and "no" if it does not. 


```r
letters <- read.csv(file.choose())
letters$isB = as.factor(letters$letter == "B")
```


Now split the data set into a training and testing set, putting 50% of the data in the training set. Set the seed to 1000 before making the split. The first argument to sample.split should be the dependent variable "letters$isB". Remember that TRUE values from sample.split should go in the training set.


```r
library(caTools)
```

```
## Warning: package 'caTools' was built under R version 3.0.3
```

```r
set.seed(1000)
split = sample.split(letters$isB, SplitRatio = 1/2)
train = subset(letters, split == T)
test = subset(letters, split == F)
```


Before building models, let's consider a baseline method that always predicts the most frequent outcome, which is "not B". What is the accuracy of this baseline method on the test set?


```r
baseline = nrow(subset(letters, letters$isB == F))/nrow(letters)
baseline
```

```
## [1] 0.7542
```


PROBLEM 1.2 - PREDICTING B OR NOT B  (1 point possible)
------------------------------------------------------------
Now build a classification tree to predict whether a letter is a B or not, using the training set to build your model. Remember to remove the variable "letter" out of the model, as this is related to what we are trying to predict! To just remove one variable, you can either write out the other variables, or remember what we did in the Billboards problem in Week 3, and use the following notation:


```r
library(rpart)
```

```
## Warning: package 'rpart' was built under R version 3.0.3
```

```r
library(rpart.plot)
```

```
## Warning: package 'rpart.plot' was built under R version 3.0.3
```

```r
CARTb = rpart(isB ~ . - letter, data = train, method = "class")
prp(CARTb)
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 


We are just using the default parameters in our CART model, so we don't need to add the minbucket or cp arguments at all. We also added the argument method="class" since this is a classification problem.

What is the accuracy of the CART model on the test set? (Use type="class" when making predictions on the test set.)


```r
CARTbPred = predict(CARTb, newdata = test, type = "class")
table(test$isB, CARTbPred)
```

```
##        CARTbPred
##         FALSE TRUE
##   FALSE  1118   57
##   TRUE     43  340
```

```r
(1118 + 340)/(1118 + 57 + 43 + 340)
```

```
## [1] 0.9358
```


PROBLEM 1.3 - PREDICTING B OR NOT B  (1 point possible)
----------------------------------------------------------------
Now, build a random forest model to predict whether the letter is a B or not. Use the default settings for ntree and nodesize (don't include these arguments at all). Right before building the model, set the seed to 1000. (NOTE: You might get a slightly different answer on this problem, even if you set the random seed. This has to do with your operating system and the implementation of the random forest algorithm.)


```r
library(randomForest)
```

```
## randomForest 4.6-7
## Type rfNews() to see new features/changes/bug fixes.
```

```r
randomForestb <- randomForest(isB ~ . - letter, data = train, method = "class")
randomForestbPred = predict(randomForestb, newdata = test, type = "class")
table(test$isB, randomForestbPred)
```

```
##        randomForestbPred
##         FALSE TRUE
##   FALSE  1164   11
##   TRUE     10  373
```

```r
(1164 + 375)/(1164 + 11 + 8 + 375)
```

```
## [1] 0.9878
```

What is the accuracy of the model on the test set?

In lecture, we noted that random forests tends to improve on CART in terms of predictive accuracy. Sometimes, this improvement can be quite significant, as it is here.

PROBLEM 2.1 - PREDICTING THE LETTERS A, B, P, R  (1 point possible)
---------------------------------------------------
Let us now move on to the problem that we were originally interested in, which is to predict whether or not a letter is one of the four letters A, B, P or R. However, we have only seen so far how CART and random forests can be applied to binary classification problems (e.g. B or not B). What can we do?

One approach is to build several layers of predictive models corresponding to each of the letters. The idea would be that we would have one tree to predict whether a letter is a B or not. If it is predicted to be a B, we would output B as our final prediction. Otherwise, we would run the observation through another tree to predict (say) whether the letter is an A or not. We would then repeat this process for the other layers.

Although such a proposal may sound reasonable, it is rather complicated. Fortunately, it turns out that CART and random forests generalize to multiclass classification problems such as, for example, predicting one of our four letters of interest. As we will see shortly, building the corresponding models in R turns out to be no harder than building the models for binary classification problems.

The variable in our data frame which we will be trying to predict is "letter". Start by converting letter in the original data set (letters) to a factor by running the following command in R:


```r
letters$letter = as.factor(letters$letter)
```


Now, generate new training and testing sets of the letters data frame using letters$letter as the first input to the sample.split function. Before splitting, set your seed to 2000. Again put 50% of the data in the training set. (Why do we need to split the data again? Remember that sample split balances the outcome variable in the training and testing sets. With a new outcome variable, we want to re-generate our split.)


```r
set.seed(2000)
split = sample.split(letters$letter, SplitRatio = 1/2)
train = subset(letters, split == T)
test = subset(letters, split == F)
```


In a multiclass classification problem, a baseline model is to predict the most frequent class of all of the options.

What is the baseline accuracy on the testing set?


```r
max(table(letters$letter))/nrow(letters)
```

```
## [1] 0.2577
```


PROBLEM 2.2 - PREDICTING THE LETTERS A, B, P, R  (1 point possible)
--------------------------------------------------------------
Now build a classification tree to predict the letter, using the training set to build your model. (Remember to remove the variable "isB" out of the model, as this is related to what we are trying to predict!) Just use the default parameters in your CART model. Add the argument method="class" since this is a classification problem. Even though we have multiple classes here, nothing changes in how we build the model from the binary case.


```r
CARTAll = rpart(letter ~ . - isB, data = train, method = "class")
prp(CARTAll)
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10.png) 

```r
CARTAllPred = predict(CARTAll, newdata = test, type = "class")
table(test$letter, CARTAllPred)
```

```
##    CARTAllPred
##       A   B   P   R
##   A 348   4   0  43
##   B   8 318  12  45
##   P   2  21 363  15
##   R  10  24   5 340
```

```r
(348 + 318 + 363 + 340)/sum(table(test$letter, CARTAllPred))
```

```
## [1] 0.8787
```


What is the test set accuracy of your CART model?

(HINT: When you are computing the test set accuracy using the confusion matrix, you want to add everything on the main diagonal and divide by the total number of observations in the test set, which can be computed with nrow(test), where test is the name of your test set).

PROBLEM 2.3 - PREDICTING THE LETTERS A, B, P, R  (1 point possible)
-------------------------------------------------------------
Now estimate a random forest model on the training data -- again, don't forget to remove the isB variable. Set the seed to 1000 right before building your model. (Remember that you might get a slightly different result even if you set the random seed.)


```r
set.seed(1000)
randomForestAll <- randomForest(letter ~ . - isB, data = train, method = "class")
randomForestAllPred = predict(randomForestAll, newdata = test, type = "class")
table(test$letter, randomForestAllPred)
```

```
##    randomForestAllPred
##       A   B   P   R
##   A 390   0   3   2
##   B   0 380   1   2
##   P   0   5 393   3
##   R   3  12   0 364
```

```r
(390 + 380 + 393 + 364)/sum(table(test$letter, randomForestAllPred))
```

```
## [1] 0.9801
```


You should find this value rather striking, for several reasons. The first is that it is significantly higher than the value for CART, highlighting the gain in accuracy that is possible from using random forest models. The second is that while the accuracy of CART decreased significantly as we transitioned from the problem of predicting B/not B (a relatively simple problem) to the problem of predicting the four letters (certainly a harder problem), the accuracy of the random forest model decreased by a tiny amount.
What is the test set accuracy of your random forest model?