# GettingData_CourseProject
This repo includes result of GettingData project
The purpose of this project is to demonstrate your ability to collect, work with, and clean a data set. The goal is to prepare tidy data that can be used for later analysis. 
This repository includes:
 1. script for performing the analysis
 2. tidy_data.csv resulted to analysis
 3. a code book that describes the variables

=============================================================================================================
Analysis script:

#Getting data from URL

if (!file.exists("./data/")){dir.create("./data/")}
fileUrl <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
download.file(fileUrl, "./data/HarDataset.zip", method = "curl")

# Unzip HarDataset
unzip(zipfile = "./data/HarDataset.zip", exdir = "./data/")

#Read train and test activities(x,y) and subjects to R
xTrain <- read.table("./data/UCI HAR Dataset/train/X_train.txt", header = FALSE)
yTrain <- read.table("./data/UCI HAR Dataset/train/y_train.txt", header = FALSE)
subjectTrain <- read.table("./data/UCI HAR Dataset/train/subject_train.txt", header = FALSE)
xTest <- read.table("./data/UCI HAR Dataset/test/X_test.txt", header = FALSE)
yTest <- read.table("./data/UCI HAR Dataset/test/y_test.txt", header = FALSE)
subjectTest <- read.table("./data/UCI HAR Dataset/test/subject_test.txt", header = FALSE)

# Merged train and test data

features <- rbind(xTrain, xTest)
activity <- rbind(yTrain, yTest)
subject <- rbind(subjectTrain, subjectTest)

# Assign feaureNamea from features.txt
names(activity) <- "activity"
names(subject) <- "subject"
featuresNames <- read.table("./data/UCI HAR Dataset/features.txt", header = FALSE)
names(features) <- featuresNames$V2

# Merge all columns to form HumanActivity data
HumanActivity <- cbind(features, subject, activity)

# Mean and std columns 

mean_col<- sapply(featuresNames[,2], function(x) grepl("mean()", x, fixed=T))
std_col<- sapply(featuresNames[,2], function(x) grepl("std()", x, fixed=T))

# Extracts only the measurements on the mean and standard deviation for each measurement
subdata <- cbind(features[,(mean_col | std_col)], subject, activity)

# Uses descriptive activity names to name the activities in the data set
# Read activity labels

activity_labels <- read.table("./data/UCI HAR Dataset/activity_labels.txt", header = FALSE)
for(i in 1:6){
  HumanActivity$activity <- gsub("i", activity_labels$V2[i], HumanActivity$activity)      
}

#Appropriately labels the data set with descriptive variable names.
names(HumanActivity)<-gsub("^t", "time", names(HumanActivity))
names(HumanActivity)<-gsub("^f", "frequency", names(HumanActivity))
names(HumanActivity)<-gsub("Acc", "Accelerometer", names(HumanActivity))
names(HumanActivity)<-gsub("Gyro", "Gyroscope", names(HumanActivity))
names(HumanActivity)<-gsub("Mag", "Magnitude", names(HumanActivity))

#5. From the data set in step 4, creates a second, independent tidy data set 
# with the average of each variable for each activity and each subject.

tidy_data <- aggregate(.~subject+activity, HumanActivity, mean)
tidy_data <- tidy_data[order(tidy_data$subject, tidy_data$activity), ]
write.csv(tidy_data, "./data/UCI HAR Dataset/tidy_data.csv", row.names=FALSE)
