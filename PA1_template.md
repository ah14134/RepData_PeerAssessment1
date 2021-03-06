---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---
### Introduction
This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

### Load and the processing data

```r
activity <- read.csv("./data/activity.csv",header=T, sep=',', na.strings="NA", quote='\"', colClasses = c("integer", "Date", "integer"))
```

### What is mean total number of steps taken per day?
Calculate the total number of steps taken per day.

```r
stepsPerDay <- aggregate(steps ~ date, data = activity, FUN = sum, na.rm=TRUE )
```

Make a histogram of the total number of steps taken each day.


```r
library(ggplot2)
qplot(steps, data=stepsPerDay, main ="Total Number of Steps Taken Each Day",
      fill=I("green"))
```

```
## stat_bin: binwidth defaulted to range/30. Use 'binwidth = x' to adjust this.
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png) 
Calculate and report the mean and median of the total number of steps taken per day

```r
mean(stepsPerDay$steps)
```

```
## [1] 10766.19
```

```r
median(stepsPerDay$steps)
```

```
## [1] 10765
```
### What is the average daily activity pattern?
Make a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
stepsAvg <-aggregate(steps ~ interval, data=activity, FUN = mean, na.rm=TRUE)
library(ggplot2)
qplot(interval, steps, data=stepsAvg,  main="The Average Daily Activity Pattern", geom="line")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png) 
Which 5-minute interval on average across all the days in the dataset contains the maximun number of steps?

```r
max_time <- max(stepsAvg$steps)
stepsAvg[stepsAvg$steps == max_time,]
```

```
##     interval    steps
## 104      835 206.1698
```

### Imputing missing values
Calculate and report the total number of missing values in dataset. 

```r
sum(is.na(activity$steps))
```

```
## [1] 2304
```

Devise a strategy for filling in all the missing values in dataset.
** The missing values of each interval should be filled with the mean of steps taken in the same interval across all days.**

```r
activity_2 <- activity
merge.df <- merge(activity_2, stepsAvg, by.x="interval", by.y="interval")
merge.df$steps.filled <- ifelse(is.na(merge.df$steps.x), merge.df$steps.y, merge.df$steps.x)
```
Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
activity_filled <- data.frame(merge.df$steps.filled, merge.df$date,  merge.df$interval)
colnames(activity_filled) <- c("steps", "date", "interval")
```
Make a histogram of the total number of steps taken each day and calculate and report the mean and median total number of steps taken per day.


```r
library(ggplot2)
stepsPerDay2 <- aggregate(steps ~ date, data = activity_filled, FUN = sum, na.rm=TRUE )
qplot(steps, data=stepsPerDay2, main ="Total Number of Steps Taken Each Day",
      fill=I("green"))
```

```
## stat_bin: binwidth defaulted to range/30. Use 'binwidth = x' to adjust this.
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png) 

```r
mean(stepsPerDay2$steps)
```

```
## [1] 10766.19
```

```r
median(stepsPerDay2$steps)
```

```
## [1] 10766.19
```

### Are there differences in activity patterns between weekdays and weekends?
Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend

```r
activity$day <- weekdays(activity$date)
activity$day_type <- ifelse((activity$day == "Saturday" | activity$day == "Sunday"), "weekend", "weekday")
```

Make a panel plot containing a time series ploy of the 5-minute interval (x-axis) and the average number of step taken, average across all weekday days or weekend days (y-axis)


```r
library(ggplot2)
stepsAvgWeekend <-aggregate(steps ~ interval + day_type, data=activity, FUN = mean, na.rm=TRUE)
names(stepsAvgWeekend) <- c("interval", "day_type", "steps")

qplot(interval, steps, data=stepsAvgWeekend,  facets = day_type~., geom="line")
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12-1.png) 
