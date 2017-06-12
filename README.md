At the beginning of the run_analysis.R you need to set your working directory. 




##SET PATH
PATH = "./data"

#####################################################################
## 1.Merges the training and the test sets to create one data set.
#####################################################################

## Download and unzip the file
fileUrl <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
destfile <- paste0(PATH,"/Dataset.zip")
download.file(fileUrl, destfile)
unzip(destfile, exdir = PATH)

## Read data
subject_train <-read.table(paste0(PATH,"/UCI HAR Dataset/train/subject_train.txt"),header = FALSE)
X_train <- read.table(paste0(PATH,"/UCI HAR Dataset/train/X_train.txt"),header = FALSE)
Y_train <- read.table(paste0(PATH,"/UCI HAR Dataset/train/Y_train.txt"),header = FALSE)

subject_test <- read.table(paste0(PATH,"/UCI HAR Dataset/test/subject_test.txt"),header = FALSE)
X_test <- read.table(paste0(PATH,"/UCI HAR Dataset/test/X_test.txt"),header = FALSE)
Y_test <- read.table(paste0(PATH,"/UCI HAR Dataset/test/Y_test.txt"),header = FALSE)

features <- read.table(paste0(PATH,"/UCI HAR Dataset/features.txt"),head=FALSE)

#concatenate data and set var name 
subject <- rbind(subject_train, subject_test)
names(subject)<-c("subject")
y<- rbind(Y_train, Y_test)
names(y)<- c("activityid")
x<- rbind(X_train, X_test)
names(x)<- features$V2

dataCombine <- cbind(subject, y)
Data <- cbind(x, dataCombine)

#####################################################################
## 2.Extracts only the measurements on the mean and standard deviation 
##   for each measurement.
#####################################################################
FeaturesNames<-features$V2[grep("mean\\(\\)|std\\(\\)", features$V2)]
length(FeaturesNames)
selectedNames<-c(as.character(FeaturesNames), "subject", "activityid" )
Data_2<-subset(Data,select=selectedNames)
str(Data_2)
#####################################################################
## 3.Uses descriptive activity names to name the activities in the data set
#####################################################################
activity_labels <- read.table(paste0(PATH,"/UCI HAR Dataset/activity_labels.txt"), header = FALSE)
colnames(activity_labels)<- c("activityid","activity")
data_3 <- merge(x=Data, y=activity_labels, by="activityid")
unique(data_3[,c("activity")])

#####################################################################
## 4.Appropriately labels the data set with descriptive variable names.
#####################################################################
colnames <-colnames(data_3)

newnames<-gsub("^t", "time", colnames)
newnames<-gsub("^f", "frequency", newnames)
newnames<-gsub("BodyBody", "Body", newnames)
newnames<-gsub("Acc", " Accelerometer", newnames)
newnames<-gsub("Gyro", " Gyroscope", newnames)
newnames<-gsub("Mag", " Magnitude", newnames)
newnames<-gsub("-", " ", newnames)
newnames<-gsub("Body", " Body", newnames)
newnames<-gsub("Gravity", " Gravity", newnames)
newnames<-gsub("Jerk", " Jerk", newnames)

names(data_3) <- newnames

#####################################################################
## 5.From the data set in step 4, creates a second, independent tidy 
##   data set with the average of each variable for each activity and 
##   each subject.
#####################################################################
install.packages("plyr")
library(plyr)
data_tidy<- ddply(data_3, c("subject","activity"), numcolwise(mean))
write.table(data_tidy, file = paste0(PATH,"/tidydata.txt"),row.name=FALSE)

