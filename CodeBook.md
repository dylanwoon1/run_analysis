---
title: "Run Analysis"
author: "dylan woon"
date: "May 21, 2016"
output: html_document
---

# Create a folder called "data" if it does not exist
if(!file.exists("./data"))
   {dir.create("./data")
}


# Download file from url
download.file(url = "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip",
              destfile = "./data/Dataset.zip",
              method = "libcurl")


# Unzip the file
unzip(zipfile="./data/Dataset.zip", exdir = "./data")


## Obtain file list (under UCI HAR Dataset)
filepath1 <- "./data/UCI HAR Dataset"
files <- list.files(file.path(filepath1), recursive = TRUE)
files


## The file list can be broadly categorized into Activity, Subject & Features. Read the relevant 6 files.
## X: feature, Y: activity, Subject: subject
ActivityTest  <- read.table(file.path(filepath1, "test" , "Y_test.txt" ),header = FALSE)
ActivityTrain <- read.table(file.path(filepath1, "train", "Y_train.txt"),header = FALSE)
SubjectTest  <- read.table(file.path(filepath1, "test" , "subject_test.txt"),header = FALSE)
SubjectTrain <- read.table(file.path(filepath1, "train", "subject_train.txt"),header = FALSE)
FeaturesTest  <- read.table(file.path(filepath1, "test" , "X_test.txt" ),header = FALSE)
FeaturesTrain <- read.table(file.path(filepath1, "train", "X_train.txt"),header = FALSE)


## Observe the structures of the 6 files.
str(ActivityTest)
str(ActivityTrain)
str(SubjectTest)
str(SubjectTest)
str(FeaturesTest)
str(FeaturesTrain)


## These data can be compiled (rbind) into one dataset. 
## 6 files become 3 files.
Subject <- rbind(SubjectTrain, SubjectTest)
Activity<- rbind(ActivityTrain, ActivityTest)
Features<- rbind(FeaturesTrain, FeaturesTest)  # why do this line?

## Set names to variables
names(Subject)<-c("subject")
names(Activity)<- c("activity")


## Extract the V2 column of features.txt into a new table.
features1 <- read.table(file.path(filepath1, "features.txt"), head = FALSE)
names(Features)<- features1$V2


## Now we have the table of Subject, Activity and Features. Merge them.
temp1 <- cbind(Subject, Activity)
data <- cbind(Features, temp1)


## Extracts mean and standard deviation for each measurement
subdatafeatures1 <- features1$V2[grep("mean\\(\\)|std\\(\\)", features1$V2)]


## Subset the data by Subject and Activity
selectedNames <- c(as.character(subdatafeatures1), "subject", "activity" )
data <- subset(data, select = )
data <- subset(data, select = selectedNames)


## Check data structure
str(data)


## Extract activity column names from "activity_labels.txt"
activitycolumn <- read.table(file.path(filepath1, "activity_labels.txt"),header = FALSE)


## Replace activity in "data" data.frame with "activitycolumn"
data[,68] <- activitycolumn[data[,68],2]


## Label data names with descriptive variable names
names(data)<-gsub("^t", "time", names(data))
names(data)<-gsub("^f", "frequency", names(data))
names(data)<-gsub("Acc", "Accelerometer", names(data))
names(data)<-gsub("Gyro", "Gyroscope", names(data))
names(data)<-gsub("Mag", "Magnitude", names(data))
names(data)<-gsub("BodyBody", "Body", names(data))


## Check the names of data
names(data)


## Produce another independent set of data
data2 <- aggregate(. ~subject + activity, data, mean)
data2 <- data2[order(data2$subject, data2$activity),]
write.table(data2, file = "tidydata.txt", row.name = FALSE)

## END