# MachineLearningFinalProject
## Background

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible
to collect a large amount of data about personal activity relatively
inexpensively. These type of devices are part of the quantified self movement
a group of enthusiasts who take measurements about themselves regularly to
improve their health, to find patterns in their behavior, or because they are
tech geeks. One thing that people regularly do is quantify how much of a
particular activity they do, but they rarely quantify how well they do it. In
this project, your goal will be to use data from accelerometers on the belt,
forearm, arm, and dumbell of 6 participants. They were asked to perform barbell
lifts correctly and incorrectly in 5 different ways. More information is
available from the website here:
http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset).

## Data

The training data for this project are available here:
https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv

The test data are available here:
https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv

The data for this project come from this source:
http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har. 
If you use the document you create for this class for any purpose please cite
them as they have been very generous in allowing their data to be used for this
kind of assignment.

## What you should submit
The goal of your project is to predict the manner in which they did the
exercise. This is the "classe" variable in the training set. You may use any of
the other variables to predict with. You should create a report describing how
you built your model, how you used cross validation, what you think the
expected out of sample error is, and why you made the choices you did. You will
also use your prediction model to predict 20 different test cases.

## Setting up the data

First, I download some packages, load the data, and set the seed as follows:

```{r}
library(caret)
library(randomForest)
library(rpart)
library(rpart.plot)
pml_training <- read.csv("pml-training.csv", na.strings=c("NA","#DIV/0!", ""))
pml_testing <- read.csv("pml-testing.csv", na.strings=c("NA","#DIV/0!", ""))
set.seed(1234)
dim(pml_training)
dim(pml_testing)
```

## Cleaning Up Data

We now clean up the data, getting rid of some unneccessary variables and NA
values are taken out.

```{r}
pml_training<-pml_training[,colSums(is.na(pml_training)) == 0]
pml_testing <-pml_testing[,colSums(is.na(pml_testing)) == 0]
pml_training<-pml_training[,-c(1:7)]
pml_testing<-pml_testing[,-c(1:7)]
dim(pml_training)
dim(pml_testing)
```

## Cross-validation

Then, we split the data into training and testing, with a 0.75 and 0.25 split.

```{r}
subsamples <- createDataPartition(y=pml_training$classe, p=0.75, list=FALSE)
pml_trainingsub <- pml_training[subsamples, ] 
pml_testingsub <- pml_training[-subsamples, ]
```

## Exploratory Data Analysis

We now look at a graph to see the distribution of classe variables:

```{r}
plot(pml_trainingsub$classe, col="blue", main="Bar Plot of levels of the variable classe within the PML Training Sub data set", xlab="classe levels", ylab="Frequency")
```

As you can see, A is the most frequent by a lot, and D is the least frequent.

## Prediction Models

We will do two prediction models: a Decision Tree and a Random Forest.

## Decision Tree

Here we get both the Decision Tree and a Confusion Matrix that gives a 
numerical breakdown.

```{r}
firstmodel <- rpart(classe ~ ., data=pml_trainingsub, method="class")
firstprediction <- predict(firstmodel, pml_testingsub, type = "class")
prp(firstmodel, main="Classification Tree", extra=102, under=TRUE, faclen=0)
confusionMatrix(firstprediction, pml_testingsub$classe)
```

## Random Forest

Now, the Random Forest, with the same process.

```{r}
secondmodel <- randomForest(classe ~. , data=pml_trainingsub, method="class")
importance(secondmodel)
secondprediction <- predict(secondmodel, pml_testingsub, type = "class")
confusionMatrix(secondprediction, pml_testingsub$classe)
```

## Decision on which Prediction Model to Use

Random Forest algorithm performed better than Decision Trees. Accuracy for
Random Forest model was 0.995 (95% CI: (0.993, 0.997)) compared to Decision
Tree model with 0.739 (95% CI: (0.727, 0.752)). The Random Forests model is
choosen. The expected out-of-sample error is estimated at 0.005, or 0.5%. So
the high majority of these points are correctly classified.

## Final Outcome

Here is the final prediction using the Random Forest Model.

```{r}
finalprediction <- predict(secondmodel, pml_testing, type="class")
finalprediction
```
