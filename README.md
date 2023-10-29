This is project assignemt of Alexey Volkov for Getting and Cleaning Data Course Project (https://www.coursera.org/learn/data-cleaning). More information about this data set can be found there - https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip and http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones.

This file contains: i) the code describing how the script works, ii) the code book describing the variables.


The following script does the following: You should create one R script called run_analysis.R that does the following:
i) Merges the training and the test sets to create one data set;
ii) Extracts only the measurements on the mean and standard deviation for each measurement; 
iii) Uses descriptive activity names to name the activities in the data set;
iv) Appropriately labels the data set with descriptive variable names; 
v) From the processed data set , creates a second, independent tidy data set with the average of each variable for each activity and each subject.

**************************************************************************
The code with description:
**************************************************************************
library(dplyr)
library(tidyr)
##We need two libraries dplyr and tidyr to process the data.

setwd("D:/Education/R/getdata_projectfiles_UCI HAR Dataset/UCI HAR Dataset/") 
## We set the directory where zip archive was unpacked before as working directory. 
## Download of zip file  was not considered in the code in order to minimize the traffic 
## during the coding and its optimization.
## The raw data was downloaded by using the following provided link:
## https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip

combined_data_test <- "test" %>% 
  file.path(., c("subject_test.txt", "y_test.txt", "X_test.txt")) %>% 
  lapply(read.table, header = FALSE) %>% 
  bind_cols()
## We combine all columns for data test in new data set entitled as "combined_data_test" (2947 observations) 
## from three txt files ("subject_test.txt", "y_test.txt", "X_test.txt") from "test" folder.

combined_data_train <- "train" %>% 
  file.path(., c("subject_train.txt", "y_train.txt", "X_train.txt")) %>% 
  lapply(read.table, header = FALSE) %>% 
  bind_cols()
## We combine all columns for data train in new data set entitled as "combined_data_train" (7352 observations)
## from three txt files ("subject_train.txt", "y_train.txt", "X_train.txt") from "train" folder.

data_raw <- rbind(combined_data_test, combined_data_train)
col_n <- as.character(read.table("features.txt")[, 2])
col_names <- c("subject", "activity", col_n)
colnames(data_raw) <- col_names
## We combine two data sets by columns, assign column names as "subject", "activity", 
## and the rest from "features.txt". The resulted data set "data_raw" consists of 
## 10299 observations, and 563 variables.

activity_name <- as.character(read.table("activity_labels.txt")[, 2])
data_raw$activity <- activity_name[data_raw$activity]
## We replace numbers by activity labels in activity row of "data_raw" by using 
## the corresponded activity labels from file "activity_labels.txt".

data_extracted <- data_raw[, c("subject", "activity", names(data_raw)[grepl("mean|std", names(data_raw))])]
## We extract only the measurements on the mean and standard deviation for each measurement
## together with columns of "subject" and "activity" in new data set entitled "data_extracted" 
## with 10299 observations, and 81 variables.

data_mean <- data_extracted %>%
  pivot_longer(cols = -c(subject, activity), names_to = "measure") %>%
  group_by(subject, activity, measure) %>%
  summarise(mean_value = mean(value)) %>%
  pivot_wider(names_from = measure, values_from = mean_value)
## We group extracted data set "data_extracted" by subject and activity,
## and then we calculate mean values for each variable.
## The resulted tidy data set entitled "data_mean" is created as new one. 

write.table(data_mean,"my_tidy_data_set.txt", row.names=FALSE)
## As requested, the resulted tidy data set with mean values is saved as 
## txt file "my_tidy_data_set.txt".

**************************************************************************
The code book describing the variables:
**************************************************************************

subject - the label of each of 30 volunteers within an age bracket of 19-48 years participated in the experiment.

activity - type of activity performed by volunteer during the data collection (WALKING, WALKING_UPSTAIRS, WALKING_DOWNSTAIRS, SITTING, STANDING, LAYING).

These signals were used to estimate variables of the feature vector for each pattern:  
'-XYZ' is used to denote 3-axial signals in the X, Y and Z directions.

tBodyAcc-XYZ
tGravityAcc-XYZ
tBodyAccJerk-XYZ
tBodyGyro-XYZ
tBodyGyroJerk-XYZ
tBodyAccMag
tGravityAccMag
tBodyAccJerkMag
tBodyGyroMag
tBodyGyroJerkMag
fBodyAcc-XYZ
fBodyAccJerk-XYZ
fBodyGyro-XYZ
fBodyAccMag
fBodyAccJerkMag
fBodyGyroMag
fBodyGyroJerkMag

The set of variables that were estimated from these signals are: 

mean(): Mean value
std(): Standard deviation
mad(): Median absolute deviation 
max(): Largest value in array
min(): Smallest value in array
sma(): Signal magnitude area
energy(): Energy measure. Sum of the squares divided by the number of values. 
iqr(): Interquartile range 
entropy(): Signal entropy
arCoeff(): Autorregresion coefficients with Burg order equal to 4
correlation(): correlation coefficient between two signals
maxInds(): index of the frequency component with largest magnitude
meanFreq(): Weighted average of the frequency components to obtain a mean frequency
skewness(): skewness of the frequency domain signal 
kurtosis(): kurtosis of the frequency domain signal 
bandsEnergy(): Energy of a frequency interval within the 64 bins of the FFT of each window.
angle(): Angle between to vectors.

Additional vectors obtained by averaging the signals in a signal window sample. These are used on the angle() variable:

gravityMean
tBodyAccMean
tBodyAccJerkMean
tBodyGyroMean
tBodyGyroJerkMean
