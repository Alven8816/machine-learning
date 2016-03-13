---
title: "Predict the manner in which they did the exercise"
author: "余文华"
date: "2016年2月27日"
output: html_document
---
#Background

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website [here](http://groupware.les.inf.puc-rio.br/har) (see the section on the Weight Lifting Exercise Dataset).

#Data

The training data for this project are available here:

<https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv>

The test data are available here:

<https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv>

setwd("C:\\Users\\HP\\Documents\\coursera\\coursera project 8 Machine Learning\\couresra project")

#download and read the data set
```{r}
pml <- read.csv("C:\\Users\\HP\\Documents\\coursera\\coursera project 8 Machine Learning\\couresra project\\pml-training.csv",header = TRUE,stringsAsFactors = FALSE,na.strings =c( NA,"#DIV/0!"))
pml$classe <- factor(pml$classe,labels= c("A","B","C","D","E"))
```


##Removing the NA columns and irrelevant variables

```{r,cache=TRUE}
library(mice)
a <- md.pattern(pml)
#there ara 19216 cases which less have one NA value.
na <- sapply(pml,FUN = function(x){sum(is.na(x))})
col <- as.numeric(which(na >= 19216))
new_pml<- pml[,-col]
#removing the irrelated columns
pml <- new_pml[,-(1:8)]
rm(col,na,new_pml)
```

#split the training and testing set
```{r}
library(caret)
set.seed(1234)
train <- createDataPartition(y=pml$classe,p = 0.7,list = FALSE)
training <- pml[train,]
testing <- pml[-train,]
```

## simulate the outcomes using the Random forest
```{r,cache=TRUE}
library(caret)
library(parallel)
library(doParallel)
library(foreach)
cl <- makeCluster(4)
registerDoParallel(cl)
fit_train <- foreach( x = 4,.combine = "rbind",.packages = 'caret') %dopar% train(classe ~ . , training, method = "rf",trControl = trainControl(method = "cv", number = x))
stopCluster(cl)
fit_train
```

##Prediction the testing set
```{r}
prediction<- predict(fit_train,newdata = testing)
confusionMatrix(prediction,testing$classe)
```
##Model assessment

    the accuracy of the random forest model is 0.9946 (p<0.05),and we made the cross validation by using parallel compute with k=4 and got a good result.
    
##Applying the Model on 20 Test Cases
```{r}
test <- read.csv("C:\\Users\\HP\\Documents\\coursera\\coursera project 8 Machine Learning\\couresra project\\pml-testing.csv",header = TRUE,stringsAsFactors = FALSE,na.strings = c(NA,"#DIV/0!"))
predict(fit_train,test)
```
## Conclusion
   In result,we got the outcome of the prediction in the test set:
   B A B A A E D B A A B C B A E E A B B B


