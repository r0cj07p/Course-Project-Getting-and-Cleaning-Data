#Scripts and How They Work

  This document contains a breakdown of the R script created for this project (called "run_analysis.R"), showing the
individual pieces of script written for each step and a description of their function.  The script assumes that the 
"plyr" package has been installed and loaded and that the datasets used have already been downloaded from the following 
URL and unzipped to the working directory in R:
https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip

##1. Merge the training and the test sets to create one data set.
### Read downloaded and unzipped data for training subjects into R 

    feat = read.table("./features.txt",header=FALSE);
    actType = read.table("./activity_labels.txt",header=FALSE);
    subTrain = read.table("./train/subject_train.txt",header=FALSE);
    xTrain = read.table("./train/x_train.txt",header=FALSE);
    yTrain = read.table("./train/y_train.txt",header=FALSE);

### Create new names for the training subject data columns, which replace the generic "V" names

    colnames(actType) = c("Activity_ID", "Activity_Type");
    colnames(subTrain) = "Subject_ID";
    colnames(xTrain) = feat[,2];
    colnames(yTrain) = "Activity_ID"

### Merge the training data tables above into a common data set (training_data)

    training_data <- cbind(subTrain,xTrain,yTrain )

### Read downloaded and unzipped data for testing subjects into R 

    subTest = read.table("./test/subject_test.txt",header=FALSE);
    xTest = read.table("./test/X_test.txt",header=FALSE);
    yTest = read.table("./test/y_test.txt",header=FALSE);

### Create new names for the testing subject data columns, which replace the generic "V" names

    colnames(subTest) = "Subject_ID";
    colnames(xTest) = feat[,2];
    colnames(yTest) = "Activity_ID"

### Merge the testing data tables above into a common data set (testing_data)

    testing_data <- cbind(subTest,xTest,yTest )

### Merge the rows of the training data with the rows of the testing data to make a combined data set including all 30   test subjects

    combined_data <- rbind(training_data, testing_data)

##2.  Extract only the measurements on the mean and standard deviation for each measurement.

### Upon examining the names of the columns in the merged data set, using the "names(combined_data)" function, it is  observed that all
### column headers related to the mean for each measurement contain the character string "mean()" and all of the column headers
### related to the standard deviation for each measurement contain the character string "std()".  Use "grepl" functions to
### automatically select columns with those character strings, along with the columns identifying the subject and the activity
### performed, to include in the final data set.

    merged_colnames = colnames(combined_data) 
    final_variables <- (grepl("Subject_ID",merged_colnames) | 
                grepl("Activity_ID",merged_colnames) |
                grepl("mean()",merged_colnames) | 
                grepl("std()",merged_colnames))

### Create the final dataset with only the data from the columns included in the "final_variables" vector.

    final_data <- combined_data[ ,final_variables]

##3. Uses descriptive activity names to name the activities in the data set

### Replace the numbers that appear in the "Activity_ID" column (column 81 of "final_data") with their corresponding activity name 
### from the "activity_labels.txt" file.

    final_data[ ,81] <- actType[final_data[ ,81],2]

### Rename the "activity_ID" column to just "Activity", and move it to the position adjacent to the "SubjectID" column.

    colnames(final_data)[81] <- "Activity"
    final_data <- final_data[,c(1,81,2:80)]

##4. Appropriately labels the data set with descriptive variable names.

### Update the "colNames" value to reflect the names of the columns in the "final_data" dataset, for use in the next step.

    colNames <- colnames(final_data)

### Remove all instances of hyphens (-), underscores (_) and parentheses () in column names to tidy them up.

    colNames[1:81] = gsub("()","",colNames[1:81])
    colNames[1:81] = gsub("-","",colNames[1:81])
    colNames[1:81] = gsub("_","",colNames[1:81])
    colNames[1:81] = gsub("std","StndDev",colNames[1:81])
    colNames[1:81] = gsub("mean","Mean",colNames[1:81])
    colNames[1:81] = gsub("tBody","TimeBody",colNames[1:81])
    colNames[1:81] = gsub("fBody","FreqBody",colNames[1:81])
    colNames[1:81] = gsub("tGravity","TimeGravity",colNames[1:81])
    colNames[1:81] = gsub("Mag","Magnitude",colNames[1:81])

### Assign the new names to the "final_data" dataset

    colnames(final_data) = colNames

##5. From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity
##   and each subject.

    final_data_means <- ddply(final_data, c("SubjectID","Activity"), numcolwise(mean))

### Use "write.table" command to write the dataset to a .txt file

    write.table(final_data_means, "Tidy_Data.txt", row.name=FALSE)
