---
title: "analysis.R Codebook"
author: "Jeffrey Strickland"
date: "12/20/2021"
output:
  html_document:
    df_print: paged
---
### Markdown
* This Codebook is an R Markdown document (Knitted). Markdown is a simple formatting syntax for authoring HTML, PDF, and MS Word documents.
* Included embedded R code chunks within the document and output

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
## Preliminaries
* Library loading and installation if required
* Data retrieval, decomressing and naming assignments

```{r cars}
library(dplyr)

getwd()
filename <- "Coursera_DS3_Final.zip"

# Checking if archive already exists.
if (!file.exists(filename)){
  fileURL <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
  download.file(fileURL, filename, method="curl")
}  

# Checking if folder exists
if (!file.exists("UCI HAR Dataset")) { 
  unzip(filename) 
}
```
## STEP 1. Merging Training and Test Set into one Data Set
* Assign each data to variables
* Merges feature sets
* merges activities code sets
* merges subject sets
* merges all prior sets

#### The features selected for this database come from the accelerometer and gyroscope 3-axial raw signals tAcc-XYZ and tGyro-XYZ.
```{r}
features <- read.table("UCI HAR Dataset/features.txt", col.names = c("n","functions"))
```
#### Activities List
List of activities performed when the corresponding measurements were taken and its codes (labels)

```{r}
activities <- read.table("UCI HAR Dataset/activity_labels.txt", col.names = c("code", "activity"))
```
#### Volunteer Test Subject 
Contains test data of 9/30 volunteer test subjects being observed

```{r}
subject_test <- read.table("UCI HAR Dataset/test/subject_test.txt", col.names = "subject")
```
#### Recorder Feature Data
Contains recorded features testing data 

```{r}
x_test <- read.table("UCI HAR Dataset/test/X_test.txt", col.names = features$functions)
```
#### Activities Codes
Contains testing data of activities’code labels

```{r}
y_test <- read.table("UCI HAR Dataset/test/y_test.txt", col.names = "code")
```
#### Subject Training Set
Contains train data of 21/30 volunteer subjects being observed

```{r}
subject_train <- read.table("UCI HAR Dataset/train/subject_train.txt", col.names = "subject")
```
#### Feature Training Set
contains recorded features train data

```{r}
x_train <- read.table("UCI HAR Dataset/train/X_train.txt", col.names = features$functions)
```
#### Acitivities Code Lables 
Contains train data of activities’code labels

```{r}
y_train <- read.table("UCI HAR Dataset/train/y_train.txt", col.names = "code")
```
### Set Merging
* Merges the training and the test sets (x) to create one data set

```{r}
X <- rbind(x_train, x_test)
```
* Merges the train data of activities’code (y) labels

```{r}
Y <- rbind(y_train, y_test)
```
* The set Subject is created by merging subject_train and subject_test using rbind() function

```{r}
Subject <- rbind(subject_train, subject_test)
```
* The set Merged_Data is created by merging Subject, Y and X using cbind() function

```{r}
Merged_Data <- cbind(Subject, Y, X)
```
## STEP2. Measure Extraction
Extracts only the measurements on the mean and standard deviation for each measurement

```{r}
TidyData <- Merged_Data %>% select(subject, code, contains("mean"), contains("std"))
```
## STEP 3. Activity Naming
Uses descriptive activity names to name the activities in the data set

```{r}
TidyData$code <- activities[TidyData$code, 2]
```
##  STEP 4. Code Labeling Process
Appropriately labels the data set with descriptive variable names
* The `code` column in `TidyData` renamed into `activities`

```{r}
names(TidyData)[2] = "activity"
```
* All `Acc` in column’s name replaced by `Accelerometer`

```{r}
names(TidyData)<-gsub("Acc", "Accelerometer", names(TidyData))
```
* All `Gyro` in column’s name replaced by `Gyroscope`

```{r}
names(TidyData)<-gsub("Gyro", "Gyroscope", names(TidyData))
```
* All `BodyBody` in column’s name replaced by `Body`

```{r}
names(TidyData)<-gsub("BodyBody", "Body", names(TidyData))
```
* All `Mag` in column’s name replaced by `Magnitude`

```{r}
names(TidyData)<-gsub("Mag", "Magnitude", names(TidyData))
```
* All start with character `t` in column’s name replaced by `Time`

```{r}
names(TidyData)<-gsub("^t", "Time", names(TidyData))
```
* All start with character `f` in column’s name replaced by `Frequency

```{r}
names(TidyData)<-gsub("^f", "Frequency", names(TidyData))
```
* All `tbody` in column’s name replaced by `TimeBody`
```{r}
names(TidyData)<-gsub("tBody", "TimeBody", names(TidyData))
```
* All `-mean()` in column’s name replaced by `Mean`
```{r}
names(TidyData)<-gsub("-mean()", "Mean", names(TidyData), ignore.case = TRUE)
```
* All `-std()` in column’s name replaced by `Stand_Dev`
```{r}
names(TidyData)<-gsub("-std()", "STD", names(TidyData), ignore.case = TRUE)
```
* All `-freq()` in column’s name replaced by `Frequency`
```{r}
names(TidyData)<-gsub("-freq()", "Frequency", names(TidyData), ignore.case = TRUE)
```
## STEP 5. Tidy Data Creation & Final Dataset
* From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject

* The set `FinalData` (180 rows, 88 columns) is created by sumarizing `TidyData` taking the means of each variable for each activity and each subject, after groupped by subject and activity.

```{r}
FinalData <- TidyData %>%
  group_by(subject, activity) %>%
  summarise_all(funs(mean))
```
### Final File Checking

```{3}
## str(FinalData) not printed
```

### File Export `FinalData` into `FinalData.txt` file.

```{r}
write.table(FinalData, "FinalData.txt", row.name=FALSE)
```
### Print `FianlData`

```{r}
FinalData
```

