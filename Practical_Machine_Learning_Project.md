
---
title: "Practical Machine Learning - Project"
output:html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

##Background and Introduction
Using devices such as Jawbone Up, Nike FuelBand, and Fitbit is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement â a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it.

In this project, we will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participant They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. The five ways are exactly according to the specification (Class A), throwing the elbows to the front (Class B), lifting the dumbbell only halfway (Class C), lowering the dumbbell only halfway (Class D) and throwing the hips to the front (Class E). Only Class A corresponds to correct performance. The goal of this project is to predict the manner in which they did the exercise, i.e., Class A to E. More information is available from the website here: http://groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset).

##Data Processing
#Import Data
Loading R packages needed and downloading training and test data sets:
```{r, message=F,warning=F}
library(caret); library(rattle); library(rpart); library(rpart.plot)
library(randomForest); library(repmis)

#importing data from the URLs
trainurl <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
testurl <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
training <- source_data(trainurl, na.strings = c("NA", "#DIV/0!", ""), header = TRUE)
testing <- source_data(testurl, na.strings = c("NA", "#DIV/0!", ""), header = TRUE)
#loading data
```
Summary: training data has 19,622 observations and 160 variables; testing data set has 20 observations and 160 variables.
##Clearning data
```{r, message=F,warning=F}
#deleting columns that contain missing values
training <- training[, colSums(is.na(training)) == 0]
testing <- testing[, colSums(is.na(testing)) == 0]

#remove first 7 predictors because these variables have little to none predicting power for "classe""
trainData <- training[, -c(1:7)]
testData <- testing[, -c(1:7)]

```

##Spliting data
Split training set into 70% for prediction and 30% for validation.
```{r}
set.seed(7826) 
inTrain <- createDataPartition(trainData$classe, p = 0.7, list = FALSE)
train <- trainData[inTrain, ]
valid <- trainData[-inTrain, ]
```
##Prediction
Using classification trees and random forests to predict the outcome
```{r}
#using 5-fold cross valdiation.
control <- trainControl(method = "cv", number = 5)
fit_rpart <- train(classe ~ ., data = train, method = "rpart", 
                   trControl = control)
print(fit_rpart, digits = 4)
```
```{r}
fancyRpartPlot(fit_rpart$finalModel)
```
```{r}
# predict outcomes using validation set
predict_rpart <- predict(fit_rpart, valid)
# Show prediction result
(conf_rpart <- confusionMatrix(valid$classe, predict_rpart))

(accuracy_rpart <- conf_rpart$overall[1])
```
##Random Forests
Due to the length of this computation and my PC's computing power, I have set ntree=100
```{r}
fit_rf <- train(classe ~ ., data = train, method = "rf",ntree=100,do.trace=F, 
                   trControl = control)
print(fit_rf, digits = 4)

# predict outcomes using validation set
predict_rf <- predict(fit_rf, valid)
# Show prediction result
(conf_rf <- confusionMatrix(valid$classe, predict_rf))

(accuracy_rf <- conf_rf$overall[1])
```
##Prediction on Testing Set
```{r}
(predict(fit_rf, testData))
```
