DATA LOADING
============

First step is to load the data to R in order to perform some exploratory
analysis

    download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv","pml-training.csv")
    download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv","pml-testing.csv")

    training<-read.csv("pml-training.csv")
    testing<-read.csv("pml-testing.csv")

EXPLORATORY ANALYSIS
====================

In this part the data is going to be examined regarding the variable to
be predicted.

    str(training$classe)

    ##  Factor w/ 5 levels "A","B","C","D",..: 1 1 1 1 1 1 1 1 1 1 ...

It appears that the variable to be predicted is a factor 5 level
variable.

MODEL SELECTION
===============

Since we don't know anything about how the other variables interact to
achieve the class selection, then the method we are going to use to
select the model is going to be to try various approaches and check on
the errors produced. The selected model will be the one with the highest
accuracy.

Since there is a large number of variables (160), the first step is to
try to reduce this number. The first step will be to check the document
to see if there are some relevant information.

The method will be to select the features based on a decision tree like
random forest and boosting. Then we will try a combination of linear
regression with decision tree will to check if we can configure a better
selected model.

TIDY DATA - DOCUMENT PAPER
--------------------------

From the document, it is informed that there are 12 variable that they
consider more relevant to explain the model. These 12 are:

-   "(1) Sensor on the Belt: discretization of the module of
    acceleration vector, variance of pitch, and variance of roll; (2)
    Sensor on the left thigh: module of acceleration vector,
    discretization, and variance of pitch; (3) Sensor on the right
    ankle: variance of pitch, and variance of roll; (4) Sensor on the
    right arm: discretization of the module of acceleration vector; From
    all sensors: average acceleration and standard deviation of
    acceleration. "\*

To identify the variables we have to subset by column name:

    # Subset by finishing with belt

    a<-grep("belt$",names(training))
    belt<-subset(training, select=a)
    head(belt)

    ##   roll_belt pitch_belt yaw_belt total_accel_belt kurtosis_roll_belt
    ## 1      1.41       8.07    -94.4                3                   
    ## 2      1.41       8.07    -94.4                3                   
    ## 3      1.42       8.07    -94.4                3                   
    ## 4      1.48       8.05    -94.4                3                   
    ## 5      1.48       8.07    -94.4                3                   
    ## 6      1.45       8.06    -94.4                3                   
    ##   kurtosis_picth_belt kurtosis_yaw_belt skewness_roll_belt
    ## 1                                                         
    ## 2                                                         
    ## 3                                                         
    ## 4                                                         
    ## 5                                                         
    ## 6                                                         
    ##   skewness_yaw_belt max_roll_belt max_picth_belt max_yaw_belt
    ## 1                              NA             NA             
    ## 2                              NA             NA             
    ## 3                              NA             NA             
    ## 4                              NA             NA             
    ## 5                              NA             NA             
    ## 6                              NA             NA             
    ##   min_roll_belt min_pitch_belt min_yaw_belt amplitude_roll_belt
    ## 1            NA             NA                               NA
    ## 2            NA             NA                               NA
    ## 3            NA             NA                               NA
    ## 4            NA             NA                               NA
    ## 5            NA             NA                               NA
    ## 6            NA             NA                               NA
    ##   amplitude_pitch_belt amplitude_yaw_belt var_total_accel_belt
    ## 1                   NA                                      NA
    ## 2                   NA                                      NA
    ## 3                   NA                                      NA
    ## 4                   NA                                      NA
    ## 5                   NA                                      NA
    ## 6                   NA                                      NA
    ##   avg_roll_belt stddev_roll_belt var_roll_belt avg_pitch_belt
    ## 1            NA               NA            NA             NA
    ## 2            NA               NA            NA             NA
    ## 3            NA               NA            NA             NA
    ## 4            NA               NA            NA             NA
    ## 5            NA               NA            NA             NA
    ## 6            NA               NA            NA             NA
    ##   stddev_pitch_belt var_pitch_belt avg_yaw_belt stddev_yaw_belt
    ## 1                NA             NA           NA              NA
    ## 2                NA             NA           NA              NA
    ## 3                NA             NA           NA              NA
    ## 4                NA             NA           NA              NA
    ## 5                NA             NA           NA              NA
    ## 6                NA             NA           NA              NA
    ##   var_yaw_belt
    ## 1           NA
    ## 2           NA
    ## 3           NA
    ## 4           NA
    ## 5           NA
    ## 6           NA

From the data it seems that the roll-pitch-yaw and aceleration are the
most consistent data, therefore these columns are selected.

    belt<-belt[,c(1,2,3,4)]

Now considering the dumbbell

    # Subset by finishing with dumbbell

    a<-grep("dumbbell$",names(training))
    dumbbell<-subset(training, select=a)
    dumbbell<-dumbbell[,c(1,2,3,19)]

Now considering the forearm

    # Subset by finishing with forearm

    a<-grep("forearm$",names(training))
    forearm<-subset(training, select=a)
    forearm<-forearm[,c(1,2,3,19)]

Now considering the arm

    # Subset by finishing with arm

    a<-grep("arm$",names(training))
    arm<-subset(training, select=a)
    arm<-arm[,c(1,2,3,4)]

Therefore we end up with 16 variables + classe to analyze.

    training2<-cbind(classe=training$classe,belt,dumbbell,forearm,arm)

DECISION TREE
-------------

The first approach for feature selection is to run random forest and
boosting and compare results to select the most relevant variables

    library(randomForest)

    ## randomForest 4.6-12

    ## Type rfNews() to see new features/changes/bug fixes.

    fit1<-randomForest(classe ~.,data = training2, na.action=na.exclude)
    fit1$importance

    ##                      MeanDecreaseGini
    ## roll_belt                   2330.6322
    ## pitch_belt                  1618.1701
    ## yaw_belt                    1954.1158
    ## total_accel_belt             649.2497
    ## roll_dumbbell                990.8535
    ## pitch_dumbbell               569.1677
    ## yaw_dumbbell                 794.5605
    ## total_accel_dumbbell         775.7471
    ## roll_forearm                1220.9094
    ## pitch_forearm               1500.9523
    ## yaw_forearm                  553.3274
    ## total_accel_forearm          372.3071
    ## roll_arm                     743.1684
    ## pitch_arm                    379.7377
    ## yaw_arm                      688.6918
    ## total_accel_arm              369.6628

The variable importance indicates that the acceleration variables have
less relevance to explain classe. Also belt and forearm variables seem
to be more relevant. Now lets check on the model accuracy on random
forest for the testing data. Unfortunately, testing data set does not
have a classe variable. Therefore we need to split the training2 data
into 2 data sets to achieve training, and test.

    d1<-training2
    library(caret)

    ## Loading required package: lattice

    ## Loading required package: ggplot2

    ## 
    ## Attaching package: 'ggplot2'

    ## The following object is masked from 'package:randomForest':
    ## 
    ##     margin

    inTrain<-createDataPartition(y=d1$classe,p=0.7, list = FALSE)
    training2<-d1[inTrain,]
    testing2<-d1[-inTrain,]

Now we can refit the random forest model including test data.

    fit1<-randomForest(classe ~.,data = training2, xtest=testing2[,-1], ytest=testing2$classe, na.action=na.exclude, keep.forest=TRUE)
    fit1$confusion

    ##      A    B    C    D    E class.error
    ## A 3890   12    0    2    2 0.004096262
    ## B   25 2596   32    4    1 0.023325809
    ## C    0   16 2360   20    0 0.015025042
    ## D    0    1   19 2228    4 0.010657194
    ## E    0    4    8    7 2506 0.007524752

Model Accuracy

    1-mean(fit1$confusion[,6])

    ## [1] 0.9878742

Now let us explore LDA and GBM in order to search for a better accuracy.
These two models were fit using caret package but computer time was
excesive.

    # LDA
    library(MASS)
    fit16<-lda(classe ~.,data = training2, na.action=na.exclude)
    table(predict(fit16, testing2)$class, testing2$classe)

    ##    
    ##        A    B    C    D    E
    ##   A 1143  253  307   58  146
    ##   B  115  371   60  146  315
    ##   C  146  158  448  122  203
    ##   D  130  121   75  471  139
    ##   E  140  236  136  167  279

    # GBM
    library(gbm)

    ## Warning: package 'gbm' was built under R version 3.4.4

    ## Loading required package: survival

    ## 
    ## Attaching package: 'survival'

    ## The following object is masked from 'package:caret':
    ## 
    ##     cluster

    ## Loading required package: splines

    ## Loading required package: parallel

    ## Loaded gbm 2.1.3

    fit12<-gbm(classe ~.,data = training2)

    ## Distribution not specified, assuming multinomial ...

    head(predict(fit12, newdata=testing2, n.trees = 500))

    ## Warning in predict.gbm(fit12, newdata = testing2, n.trees = 500): Number of
    ## trees not specified or exceeded number fit so far. Using 100.

    ## [1] 0.4106584 0.4106584 0.4106584 0.4106584 0.4106584 0.4106584

As we can see from the results, with a single Random Forest model we
have obtained a better fit. Therefore we proceed on tuning the random
forest parameters to improve accuracy.

    fit1v1<-randomForest(classe ~.,data = training2, xtest=testing2[,-1], ytest=testing2$classe, na.action=na.exclude, keep.forest=TRUE, mtry = 4,ntree = 550 )
    1-mean(fit1v1$confusion[,6])

    ## [1] 0.98807

Given that the parameters obtained in our first model are better we will
keep it as it performs better accuracy.

\`\`\`
