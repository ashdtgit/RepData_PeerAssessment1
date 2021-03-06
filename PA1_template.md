---
title: "Reproducible Research: Peer Assessment 1"
author: "Ash Thompson"
date: "4 September 2020"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data
### Step 1. Initial Setup - Load dependencies, clear enviroment, setup data folder
The following code does some initial setup and cleaning.

1. First load any dependencies required.
2. Next clear the enviroment.
3. Then setup a new folder called data if it does not already exist.


```r
library(dplyr)
library(ggplot2)

rm(list=ls())

basedir <- getwd()
if (!file.exists("data"))
{
        dir.create("data")
}
```


### Step 2. Downloading and reading the data
The following code is used to download the data from the source, unzip it
and then read it into a dataframe.

1. Save the URL
2. Create a temp file
3. Download the zip into the temp file
4. Unzip into the data directory
5. Remove the temp zip file
6. Read the CSV file into a dataframe


```r
fileUrl = "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
temp <- tempfile()
download.file(fileUrl,temp)
unzip(temp,exdir="./data")
unlink(temp)
activity <- read.csv("./data/activity.csv", header = TRUE)
```

### Step 3. Basic summaries to see what is in the data
Names of variables, top of the file, structure and summary.


```r
names(activity)
```

```
## [1] "steps"    "date"     "interval"
```

```r
head(activity)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```

```r
str(activity)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

```r
summary(activity)
```

```
##      steps                date          interval     
##  Min.   :  0.00   2012-10-01:  288   Min.   :   0.0  
##  1st Qu.:  0.00   2012-10-02:  288   1st Qu.: 588.8  
##  Median :  0.00   2012-10-03:  288   Median :1177.5  
##  Mean   : 37.38   2012-10-04:  288   Mean   :1177.5  
##  3rd Qu.: 12.00   2012-10-05:  288   3rd Qu.:1766.2  
##  Max.   :806.00   2012-10-06:  288   Max.   :2355.0  
##  NA's   :2304     (Other)   :15840
```



## Question - What is mean total number of steps taken per day?

### Step 1. Calculate the total number of steps taken per day
The below code is using summarise to group the data by date, and use sum to calculate total steps for each date.

```r
stepsbyday <- summarise(group_by(activity, date), TotalSteps = sum(steps))
```

### Step 2. If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day
The below code is using the hist function from the base plot package to create a histogram.
The histogram is run on the TotalSteps variable from my summarised dataset.
Breaks are set at 25 to make it a bit more readable, labels are added.


```r
hist(stepsbyday$TotalSteps, breaks=25, xlab="Total Steps per day", ylab="Frequency", main="Histogram of total steps per day")
```

![plot of chunk basehist](figure/basehist-1.png)


### Step 3. Calculate and report the mean and median of the total number of steps taken per day

Calculating the mean. Note that NA's are ignored.

```r
meanTotalSteps <- mean(stepsbyday$TotalSteps, na.rm = TRUE)
```
The mean of the total number of steps taken per day is:
**10766.19**

Calculating the median. Note that NA's are ignored.

```r
medianTotalSteps <- median(stepsbyday$TotalSteps, na.rm = TRUE)
```
The median of the total number of steps taken per day is:
**10765**



## Question - What is the average daily activity pattern?

### Step 1. Setup data to show the average daily activity pattern.

The below code summarises the activity data by interval, giving the average steps across all days for each interval.

```r
aveStepsByInteval <- summarise(group_by(activity, interval), aveSteps = mean(steps, na.rm = TRUE))
```


### Step 2.  Make a time series plot (type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

The below code uses the base plot system to plot the average steps per interval.

```r
with(aveStepsByInteval, 
     plot(interval, aveSteps, type = "l",
          xlab = "Interval", ylab = "Average number of steps",
          main = "Average number of steps for each interval",
          col = "black", lwd = 1.5, ylim = c(0, 210)))
```

![plot of chunk intervalPlot](figure/intervalPlot-1.png)

### Step 3. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

The below code uses which.max function to find which row contains the max average.
Then puts the interval value from that row into a new variable.

```r
which.max(aveStepsByInteval$aveSteps)
```

```
## [1] 104
```

```r
maxinterval <- aveStepsByInteval[[which.max(aveStepsByInteval$aveSteps), 1]]
```

The 5-minute interval, on average across all the days in the dataset, that contains the maximum number of steps is:

Interval **835**


## Imputing missing values

*Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.*

### Step 1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NA)

Summary can be used to show that in the activity data, 2304 NA's appear in the steps variable.
But to make it easier to work with these NAs, the below code uses is.na to create a logical vector of the rows containing NA in the steps variable, 
and then uses sum to count them into a new variable countNAs.


```r
summary(activity)
```

```
##      steps                date          interval     
##  Min.   :  0.00   2012-10-01:  288   Min.   :   0.0  
##  1st Qu.:  0.00   2012-10-02:  288   1st Qu.: 588.8  
##  Median :  0.00   2012-10-03:  288   Median :1177.5  
##  Mean   : 37.38   2012-10-04:  288   Mean   :1177.5  
##  3rd Qu.: 12.00   2012-10-05:  288   3rd Qu.:1766.2  
##  Max.   :806.00   2012-10-06:  288   Max.   :2355.0  
##  NA's   :2304     (Other)   :15840
```

```r
listNAs <- is.na(activity$steps)
countNAs <- sum(listNAs)
```
The number of rows containing NA in the steps variable is :
**2304**


### Step 2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc. Create a new dataset that is equal to the original dataset but with the missing data filled in.

I decided upon using the average interval value to impute the missing values.

First the code creates a vector of the average steps by interval in the same dimension as the original activity data, using match.
Then creates a new version of the activity dataset, next adds in the average steps as a new column.
Finally if the activity data was NA, replaces with the contents of the average column.

```r
activityAveStepsList <- aveStepsByInteval$aveSteps[match(activity$interval, aveStepsByInteval$interval)]

activityNew <- activity
activityNew$intAveSteps <- activityAveStepsList 

activityNew$steps <- ifelse(is.na(activity$steps), yes = activityNew$intAveSteps, no = activity$steps)
```


### Step 3. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

Same as earlier, the code uses summarise to group by date and calculate total steps per day.
Except this time it is referring to the new activity data with imputed steps.

```r
stepsbydayNew <- summarise(group_by(activityNew, date), TotalSteps = sum(steps))
```

The histogram code is the same as earlier, just refers to the new activity data.

```r
hist(stepsbydayNew$TotalSteps, breaks=25, xlab="Total Steps per day", ylab="Frequency", main="Histogram of total steps per day (imputed)")
```

![plot of chunk basehistNew](figure/basehistNew-1.png)

Calculating the mean from the data that includes imputation.

```r
meanTotalStepsNew <- mean(stepsbydayNew$TotalSteps, na.rm = TRUE)
```
The mean of the total number of steps taken per day is:
**10766.19**
Compared to the original mean which was:
**10766.19**

Calculating the median from the data that includes imputation.

```r
medianTotalStepsNew <- median(stepsbydayNew$TotalSteps, na.rm = TRUE)
```
The median of the total number of steps taken per day is:
**10766.19**
Compared to the original median which was:
**10765**

#### Conclusion: 
#### Imputing has only slightly changed the mean and median, but has made them the same.



## Are there differences in activity patterns between weekdays and weekends?
*For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.*

### Step 1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.
First create a new formatted date variable using the as.date() function, and a new variable for the name of the weekday.
Then create new variable type, which has either Weekend or Weekday, based on whether day was Saturday, Sunday or other.

```r
activity$dateFormat <- as.Date(activity$date, format = "%Y-%m-%d")
activity$day <- weekdays(activity$dateFormat)
activity$type <- ifelse(activity$day=='Saturday' | activity$day=='Sunday', yes = 'Weekend', no = 'Weekday')
```


### Step 2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). 
*See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.*

First make a new average steps by interval summary, adding in another group for type.
Then use ggplot to create the panel plot containing the time series plot.

```r
aveStepsByIntevalT <- summarise(group_by(activity, interval, type), aveSteps = mean(steps, na.rm = TRUE))

g <- ggplot(aveStepsByIntevalT, aes(interval, aveSteps))
g+      geom_line(col="blue")+
        xlab("Interval")+
        ylab("Steps")+
        ggtitle("Average steps by interval during a Weekday compared to the Weekend")+
        facet_grid(type ~ .)
```

![plot of chunk avestepsandtype](figure/avestepsandtype-1.png)
