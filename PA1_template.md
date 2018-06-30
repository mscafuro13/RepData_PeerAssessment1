---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data
- Check if the data file exists
    - If data file doesn't exist, check if the zip file exists
        - If zip file doesn't exist, download it
    - Unzip file
- Read in data file


```r
if(!file.exists("activity.csv")){
    if(!file.exists("activity.zip")){
        url <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
        download.file(url, "activity.zip")
    }
    unzip("activity.zip")
}
activity <- read.csv("activity.csv")
```

## What is mean total number of steps taken per day?
##### Create dataset for number of steps daily. Per instructions, missing values may be ignored.

```r
activityDailySum <- aggregate(steps ~ date, activity, FUN = sum, na.rm = TRUE)
str(activityDailySum)
```

```
## 'data.frame':	53 obs. of  2 variables:
##  $ date : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 2 3 4 5 6 7 9 10 11 12 ...
##  $ steps: int  126 11352 12116 13294 15420 11015 12811 9900 10304 17382 ...
```

##### Create histogram for steps per day.

```r
hist(activityDailySum$steps, xlab = "Steps Per Day", main = "Histogram of Steps Per Day")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

##### Calculate mean and median steps per day.

```r
mean(activityDailySum$steps)
```

```
## [1] 10766.19
```

```r
median(activityDailySum$steps)
```

```
## [1] 10765
```

## What is the average daily activity pattern?
##### Create a dataset for average steps per five-minute intervals across all days.

```r
activityIntervalMean <- aggregate(steps ~ interval, activity, FUN=mean, na.rm = TRUE)
str(activityIntervalMean)
```

```
## 'data.frame':	288 obs. of  2 variables:
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
##  $ steps   : num  1.717 0.3396 0.1321 0.1509 0.0755 ...
```

##### Plot average numbers of steps per 5 minute interval across all days

```r
plot(y = activityIntervalMean$steps, x = activityIntervalMean$interval, 
        type = "l", ylab = 'Steps', xlab = 'Interval')
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

##### Find interval with maximum number of steps

```r
activityIntervalMean[which.max(activityIntervalMean$steps),]$interval
```

```
## [1] 835
```
## Imputing missing values
##### Calculate and report total number of missing values

```r
nrow(activity[is.na(activity),])
```

```
## [1] 2304
```

##### Fill in all missing values in the dataset.  Missing values will be substituted with the mean value across all days for that interval.

```r
# Merge original data frame with activityIntervalMean
tempActivity <- merge(activity, activityIntervalMean, by = "interval")
# Replace NA values with the interval mean
tempActivity$steps.x[is.na(tempActivity$steps.x)] <- tempActivity$steps.y[is.na(tempActivity$steps.x)]
# Reorder data frame by date and interval
tempActivity <- tempActivity[order(tempActivity$date,tempActivity$interval),]

# Reset row index
rownames(tempActivity) <- NULL
# Create new data set with only columns of interest
activityClean <- tempActivity[,c("steps.x","date","interval")]
# Rename columns to original names
names(activityClean) <- c("steps","date","interval")
str(activityClean)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : num  1.717 0.3396 0.1321 0.1509 0.0755 ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

## Are there differences in activity patterns between weekdays and weekends?
