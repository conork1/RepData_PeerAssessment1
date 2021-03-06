---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data

```r
# Load the csv data from the zip file into a data frame
step_data <- read.table(unz("activity.zip", "activity.csv"), header=T, quote="\"", sep=",")
# ensure that the date column is a date data type
step_data$date <- strptime(step_data$date, "%Y-%m-%d")
```

## What is mean total number of steps taken per day?

```r
library(plyr)
complete_steps <- step_data[complete.cases(step_data$steps),]
totaldailysteps <- ddply(complete_steps, .(date), summarise, total = max(steps))
hist(totaldailysteps$total, main = "Total number of steps each day", xlab = "Number of Steps")
```

![plot of chunk mean total steps](figure/mean total steps-1.png)

```r
meansteps <- mean(totaldailysteps$total,na.rm = TRUE)
mediansteps <- median(totaldailysteps$total,na.rm = TRUE)
```
The mean number of total daily steps is 600.8867925. The median number of total daily steps is 555.

## What is the average daily activity pattern?


```r
avgsteps <- ddply(complete_steps, .(interval), summarise, average_steps = mean(steps))
plot(x = avgsteps$interval, y = avgsteps$average_steps, type = "l", main = "Average Daily Activity Pattern", xlab = "Daily Interval", ylab = "Average Number of steps")
```

![plot of chunk avg by interval](figure/avg by interval-1.png)

```r
maxinterval <- avgsteps[which.max(avgsteps$average_steps),]$interval
```
The daily 5-minute interval containing the max number of steps on average is 835.

## Imputing missing values


```r
missingrows <- sum(is.na(step_data$steps))
```
The Total number of missing values (coded as NA) is 2304.


```r
#Create a copy of the step_data dataset which will be used to fill in missing values
step_dataimputed <- step_data
#Impute the missing values by using the average steps by interval
step_dataimputed$steps[is.na(step_dataimputed$steps)] <- ave(step_dataimputed$steps, step_dataimputed$interval, FUN = function(x)mean(x, na.rm=TRUE))[c(which(is.na(step_dataimputed$steps)))]
```


```r
library(plyr)
totaldailysteps_imputed <- ddply(step_dataimputed, .(date), summarise, total = max(steps))
hist(totaldailysteps_imputed$total, main = "Total number of steps each day", xlab = "Number of Steps")
```

![plot of chunk mean total steps imputed](figure/mean total steps imputed-1.png)

```r
# calculate the mean with the imputed values
meansteps_imputed <- mean(totaldailysteps_imputed$total,na.rm = TRUE)
# calculate the difference between this mean and the one calcualted above with missing values ignored
meansteps_imputed_diff <- meansteps_imputed - meansteps

if (meansteps_imputed_diff == 0) {
  meanresult <- "no"
} else if (meansteps_imputed_diff < 0) {
  meanresult <- "a negative effect"
}else 
  meanresult <- "a positive effect"

# calculate the median with the imputed values
mediansteps_imputed <- median(totaldailysteps_imputed$total,na.rm = TRUE)
# calculate the difference between this median and the one calcualted above with missing values ignored
mediansteps_imputed_diff <- mediansteps_imputed - mediansteps

if (mediansteps_imputed_diff == 0) {
  medianresult <- "no"
} else if (mediansteps_imputed_diff < 0) {
  medianresult <- "a negative effect"
}else 
  medianresult <- "a positive effect"
```
The mean number of total daily steps (with missing values imputed) is 549.120631. The median number of total daily steps (with missing values imputed) is 542. This is a difference of -51.7661615 for the mean and -13 for the median from the calculations with missing values ignored.

Imputing the missing data has a negative effect on the mean and a a negative effect on the median.

## Are there differences in activity patterns between weekdays and weekends?


```r
step_dataimputed$weekday <- weekdays(step_dataimputed$date,abbreviate = TRUE)
step_dataimputed$DayType <- "weekday"
step_dataimputed$DayType[step_dataimputed$weekday == "Sat" | step_dataimputed$weekday == "Sun"] <- "weekend"
step_dataimputed$DayType <- factor(step_dataimputed$DayType)

library(lattice)
avgsteps_imputed <- ddply(step_dataimputed, .(interval, DayType), summarise, average_steps = mean(steps))
xyplot(average_steps~interval | as.factor(DayType), data = avgsteps_imputed, layout = c(1,2), type="l", xlab = "Interval", ylab = "Number of steps" )
```

![plot of chunk weekend diffs](figure/weekend diffs-1.png)
