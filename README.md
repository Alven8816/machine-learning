# machine-learning
Predict the manner in which they did the exercise
余文华

2016年2月27日

Background
Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here (see the section on the Weight Lifting Exercise Dataset).

Data
The training data for this project are available here:

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv

The test data are available here:

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv

setwd(“C:\Users\HP\Documents\coursera\coursera project 8 Machine Learning\couresra project”)

download and read the data set
pml <- read.csv("C:\\Users\\HP\\Documents\\coursera\\coursera project 8 Machine Learning\\couresra project\\pml-training.csv",header = TRUE,stringsAsFactors = FALSE,na.strings =c( NA,"#DIV/0!"))
pml$classe <- factor(pml$classe,labels= c("A","B","C","D","E"))
Removing the NA columns and irrelevant variables
library(mice)
a <- md.pattern(pml)
## Warning in data.matrix(x): 强制改变过程中产生了NA
## Warning in data.matrix(x): 强制改变过程中产生了NA
## Warning in data.matrix(x): 强制改变过程中产生了NA
#there ara 19216 cases which less have one NA value.
na <- sapply(pml,FUN = function(x){sum(is.na(x))})
col <- as.numeric(which(na >= 19216))
new_pml<- pml[,-col]
#removing the irrelated columns
pml <- new_pml[,-(1:8)]
rm(col,na,new_pml)
split the training and testing set
library(caret)
## Loading required package: lattice
## Loading required package: ggplot2
set.seed(1234)
train <- createDataPartition(y=pml$classe,p = 0.7,list = FALSE)
training <- pml[train,]
testing <- pml[-train,]
simulate the outcomes using the Random forest
library(caret)
library(parallel)
library(doParallel)
## Loading required package: foreach
## Loading required package: iterators
library(foreach)
cl <- makeCluster(4)
registerDoParallel(cl)
fit_train <- foreach( x = 4,.combine = "rbind",.packages = 'caret') %dopar% train(classe ~ . , training, method = "rf",trControl = trainControl(method = "cv", number = x))
stopCluster(cl)
fit_train
## Random Forest 
## 
## 13737 samples
##    51 predictor
##     5 classes: 'A', 'B', 'C', 'D', 'E' 
## 
## No pre-processing
## Resampling: Cross-Validated (4 fold) 
## Summary of sample sizes: 10303, 10303, 10303, 10302 
## Resampling results across tuning parameters:
## 
##   mtry  Accuracy   Kappa      Accuracy SD  Kappa SD   
##    2    0.9890806  0.9861851  0.001313236  0.001661087
##   26    0.9900269  0.9873827  0.001374218  0.001738404
##   51    0.9868238  0.9833295  0.002576413  0.003260213
## 
## Accuracy was used to select the optimal model using  the largest value.
## The final value used for the model was mtry = 26.
Prediction the testing set
prediction<- predict(fit_train,newdata = testing)
## Loading required package: randomForest
## randomForest 4.6-12
## Type rfNews() to see new features/changes/bug fixes.
## 
## Attaching package: 'randomForest'
## 
## The following object is masked from 'package:ggplot2':
## 
##     margin
confusionMatrix(prediction,testing$classe)
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1674   11    0    0    0
##          B    0 1127    5    0    0
##          C    0    1 1018    5    1
##          D    0    0    3  958    1
##          E    0    0    0    1 1080
## 
## Overall Statistics
##                                           
##                Accuracy : 0.9952          
##                  95% CI : (0.9931, 0.9968)
##     No Information Rate : 0.2845          
##     P-Value [Acc > NIR] : < 2.2e-16       
##                                           
##                   Kappa : 0.994           
##  Mcnemar's Test P-Value : NA              
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            1.0000   0.9895   0.9922   0.9938   0.9982
## Specificity            0.9974   0.9989   0.9986   0.9992   0.9998
## Pos Pred Value         0.9935   0.9956   0.9932   0.9958   0.9991
## Neg Pred Value         1.0000   0.9975   0.9984   0.9988   0.9996
## Prevalence             0.2845   0.1935   0.1743   0.1638   0.1839
## Detection Rate         0.2845   0.1915   0.1730   0.1628   0.1835
## Detection Prevalence   0.2863   0.1924   0.1742   0.1635   0.1837
## Balanced Accuracy      0.9987   0.9942   0.9954   0.9965   0.9990
Model assessment
the accuracy of the random forest model is 0.9946 (p<0.05),and we made the cross validation by using parallel compute with k=4 and got a good result.
Applying the Model on 20 Test Cases
test <- read.csv("C:\\Users\\HP\\Documents\\coursera\\coursera project 8 Machine Learning\\couresra project\\pml-testing.csv",header = TRUE,stringsAsFactors = FALSE,na.strings = c(NA,"#DIV/0!"))
predict(fit_train,test)
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
Conclusion
In result,we got the outcome of the prediction in the test set: B A B A A E D B A A B C B A E E A B B B
