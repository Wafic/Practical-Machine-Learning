---
title: "Coursera Machine Learning Project"
author: "Wafic El-Assi"
date: "Saturday, March 26, 2016"
output: html_document
---

```{r}
echo=TRUE
```

This markdown file is a deliverable for the completion of the Practical Machine Learning course - part of the Data Science specialization offered on Coursera.

#Overview

Two accelerometer datasets are provided; a training and a testing dataset. The objective of this project is to use the data, gathered off six participants, to quantify how well do they do a particular set of exercies.

#Loading and Cleaning The Data

Load the required libraries. 

Remove NAs from train and test data while loading them for more accurate predictions.

```{r}
library(caret)
library(ggplot2)
library(randomForest)

#read datasets
testdata <- read.csv(file = "pml-testing.csv", header=TRUE, 
                     na.strings = c("NA", "#DIV/0!"))
traindata <- read.csv(file = "pml-training.csv", header=TRUE,
                      na.strings = c("NA", "#DIV/0!"))
```

Check if the required outcome variable (classe) is available.

Store the classe variable in a vector named "outcome"

```{r}
str(traindata$classe)

outcome = traindata$classe

```

#Selecting Features

Grep variables to be investigated to predict outcome: arm, forearm, dumble, belt.

Next, remove columns with no entries. Don't forget to bind the outcome variable with the train dataset.

```{r}
accel_filter = grep("belt|arm|dumbell|forearm", names(traindata))

train = traindata[,accel_filter]
test = testdata[,accel_filter]

cols.without.na = colSums(is.na(train)) == 0
train = train[, cols.without.na]
test = test[, cols.without.na]

train = cbind(outcome, train)

#Check if there any missing cases in the dataset
table(complete.cases(train))

#check dim of both train and test datasets
#train features should be equal to test features + 1 due to outcome variable
dim(train)
dim(test)
```

#Creating a Validation Set

Set seed for data partitioning and modelling.

Data partitioning is conducted to split the data (by outcome/classe) to get the out-of-sample error. That is to validate the model after fitting.

```{r}
set.seed(3)

inTrain <- createDataPartition(y=train$outcome, p=0.75, list=FALSE)
training <- train[inTrain,]
testing <- train[-inTrain,]

#equal number of features
dim(training)
dim(testing)
```

Select the random forest algorithm to the train the data with number of trees set to 500.

#Applying Random Forest Algorithm

```{r}
set.seed(3)
rf_model <- randomForest(outcome ~ ., data = training, ntree=500)
```

plot the true erroe of every class. It is evident that the error for all 5 people is below 0.1 (very close to 1). This indicates that a random forest algorithm is suitable. No other ML algorithms will be tested accordingly.

```{r}
plot(rf_model)
```

Create a confusion matrix using the caret package to calculate the error. The accuracy of the model is approximately equal to 0.995, with a 0.005 error rate (very small). The model can then be used to answer the quiz questions at the end (blocked out in adherence to the honor code).

```{r}
predictions <- predict(rf_model, newdata=testing)
confusionMatrix(predictions, testing$outcome)

#Quiz predictions using test dataset
#Quiz <- predict(rf_model, newdata=test)
```

