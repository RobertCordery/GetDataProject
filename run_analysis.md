---
title: "Coursera Getting Data Course Project"
author: "Robert Cordery"
date: "Thursday, February 19, 2015"
output: html_document
---
# Project steps

* Merge the training and the test sets to create one data set.  
* Extract only the measurements on the mean and standard deviation for each measurement.   
* Use descriptive activity names to name the activities in the data set  
* Appropriately label the data set with descriptive variable names.   


# Load the data


```r
library(dplyr)
library(reshape2)
```

In this section I provide the various paths to files.


```r
dataFile <- "UCI HAR Dataset.zip"
unzip(dataFile)
dataRoot <- "UCI HAR Dataset"
dataTest <- "UCI HAR Dataset/test"
dataTestInertial <- "UCI HAR Dataset/test/Inertial Signals"
dataTrain <- "UCI HAR Dataset/train"
dataTrainInertial <- "UCI HAR Dataset/train/Inertial Signals"
```

Use the names of the activities and features provided with the dataset to 
label the data. The provided feature names have some special characters "(", ")" 
and "-" that are 
changed when used to label columns in R, so I replace them with "_". 


```r
featureNames<-read.table(paste(dataRoot,"/features.txt",sep=""),
                         colClasses=c("NULL","character"),
                         col.names = c("NULL","Names"),
                         stringsAsFactors = FALSE)
featureNames <- gsub(pattern="-", replacement="_", featureNames$Names, 
     ignore.case = FALSE, perl = FALSE, fixed = FALSE, useBytes = FALSE)
featureNames <- gsub(pattern="\\(\\)", replacement="_", featureNames, 
                     ignore.case = FALSE, perl = FALSE, fixed = FALSE, useBytes = FALSE)

activityNames<-read.table(paste(dataRoot,"/activity_labels.txt",sep=""),
                         colClasses=c("NULL","character"),
                         col.names = c("NULL","activity"),
                         stringsAsFactors = FALSE)
activityNames <- activityNames$activity
```

Load the measurements for the test data into a dataframe "wearableTest". Following the advice 
on the course Discussion Board, I ginored the inertial data for now. 


```r
X_test <- read.table(paste(dataTest,"/X_test.txt", sep=""), 
                     col.names = featureNames)
activityNum_test <- read.table(paste(dataTest,"/y_test.txt", sep=""), header=FALSE)
activityNum_test <- activityNum_test$V1
activity <- activityNames[activityNum_test]
subject_test <- read.table(paste(dataTest,"/subject_test.txt", sep=""), header=FALSE)
subject <- subject_test$V1
wearableTest <- cbind(subject,activity, X_test)
```

Load the measurements for the training data into a dataframe "wearableTrain". Prepend
with columns for the subject and activity. 


```r
X_train <- read.table(paste(dataTrain,"/X_train.txt", sep=""), 
                     col.names = featureNames)
activityNum_train <- read.table(paste(dataTrain,"/y_train.txt", sep=""), header=FALSE)
activityNum_train <- activityNum_train$V1
activity <- activityNames[activityNum_train]
subject_train <- read.table(paste(dataTrain,"/subject_train.txt", sep=""), header=FALSE)
subject <- subject_train$V1
wearableTrain <- cbind(subject, activity, X_train)
```

# Merge the test and training data
As the column names and order are consistent fopr the test and training data, combining
the two data sets is simple. 


```r
wearableDf <- rbind(wearableTest, wearableTrain)
```

I select the columns that have mean and standard deviation values using the strings 
"mean_" and "std_" respectively. 


```r
meanCols <- grep("mean_", featureNames)
stdCols <- grep("std_", featureNames)
meanNames <- featureNames[meanCols]
stdNames <- featureNames[stdCols]
wearableStats <- cbind(as.factor(wearableDf[,1]),
                       wearableDf[,2],
                       wearableDf[,meanNames],
                       wearableDf[,stdNames])
colnames(wearableStats)<-c("subject","activity",meanNames,stdNames)

dim(wearableStats)
```

```
## [1] 10299    68
```

Check that the data has no missing values

```r
all(colSums(is.na(wearableStats))==0)
```

```
## [1] TRUE
```

# Tidy the data
Now we tidy up the data using melt() and dcast(), using the subject and activity 
as id values.  



```r
wearableStatsMelt <- melt(wearableStats, 
                          id=c("activity","subject"), 
                          measure.vars = c(meanNames,stdNames))
wearableStatSummary <- dcast(wearableStatsMelt, 
                             subject+activity ~ variable, mean)
```
Now we output the required table as a text file. 


```r
write.table(wearableStatSummary, 
            file = "wearableStatSummary.txt", 
            row.name=FALSE, sep="\t")
```

The resulting table has two id columns for subject and activity. For each subject the statistics for each measured feature is provided in a column. 

# License:
========
Use of this dataset in publications must be acknowledged by referencing the following publication [1] 

[1] Davide Anguita, Alessandro Ghio, Luca Oneto, Xavier Parra and Jorge L. Reyes-Ortiz. Human Activity Recognition on Smartphones using a Multiclass Hardware-Friendly Support Vector Machine. International Workshop of Ambient Assisted Living (IWAAL 2012). Vitoria-Gasteiz, Spain. Dec 2012

This dataset is distributed AS-IS and no responsibility implied or explicit can be addressed to the authors or their institutions for its use or misuse. Any commercial use is prohibited.

Jorge L. Reyes-Ortiz, Alessandro Ghio, Luca Oneto, Davide Anguita. November 2012.

