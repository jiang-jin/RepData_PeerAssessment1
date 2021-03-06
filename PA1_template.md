# Reproducible Research: Peer Assessment 1

```r
library (ggplot2)
```

```
## Warning: package 'ggplot2' was built under R version 3.1.3
```

```r
library (knitr)
library (scales)
library (Hmisc)
```

```
## Loading required package: grid
## Loading required package: lattice
## Loading required package: survival
## Loading required package: splines
## Loading required package: Formula
## 
## Attaching package: 'Hmisc'
## 
## The following objects are masked from 'package:base':
## 
##     format.pval, round.POSIXt, trunc.POSIXt, units
```

```r
## Loading and preprocessing the data

if(!file.exists('activity,csv')){unzip('activity.zip')}
activityData <- read.csv('activity.csv')


## What is mean total number of steps taken per day?

stepsByDay <- tapply(activityData$steps, activityData$date, sum, na.rm=TRUE)

## Make a histogram of the total number of steps taken each day

qplot(stepsByDay, xlab='Total steps per day', ylab='Frequency using binwith 500', binwidth=500)
```

![](PA1_template_files/figure-html/unnamed-chunk-1-1.png) 

```r
## Calculate the mean and median total number of steps taken per day

stepsByDayMean <- mean(stepsByDay)

stepsByDayMedian <- median(stepsByDay)
## mean: stepsByDayMean
## median: stepsByDayMedian


## What is the average daily activity pattern?

averageStepsPerTimeBlock <- aggregate(x=list(meanSteps=activityData$steps), by=list(interval=activityData$interval), FUN=mean, na.rm=TRUE)

## Create a time series plot

ggplot(data=averageStepsPerTimeBlock, aes(x=interval, y=meanSteps)) +
    geom_line() +
    xlab("5-minute interval") +
    ylab("average number of steps taken") 
```

![](PA1_template_files/figure-html/unnamed-chunk-1-2.png) 

```r
## Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

mostSteps <- which.max(averageStepsPerTimeBlock$meanSteps)
timeMostSteps <-  gsub("([0-9]{1,2})([0-9]{2})", "\\1:\\2", averageStepsPerTimeBlock[mostSteps,'interval'])

## Most Steps at: timeMostSteps

## Imputing missing values
## Calculate total number of missing values

numMissingValues <- length(which(is.na(activityData$steps)))
## Number of missing values: numMissingValues

## Devise a stragegy for filling in all missing values in the dataset 
## Create a new dataset that is equal to the oritinal dataset but with the missing data filled in.

activityDataImputed <- activityData
activityDataImputed$steps <- impute(activityData$steps, fun=mean)

## Make a histogram of the total number of steps taken each day and  the mean and median total number of steps taken per day.

## steps taken each day
stepsByDayImputed <- tapply(activityDataImputed$steps, activityDataImputed$date, sum)
qplot(stepsByDayImputed, xlab='Total steps per day (Imputed)', ylab='Frequency using binwith 500', binwidth=500)
```

![](PA1_template_files/figure-html/unnamed-chunk-1-3.png) 

```r
## mean and median total number of steps taken per day
stepsByDayMean <- mean(stepsByDayImputed)
stepsByDayMedian<- median(stepsByDayImputed)

## mean: stepsByDayMean
## median: stepsByDayMedian


## Are there differences in activity patterns between weekdays and weekends?

## Create a factor viariable in the dataset with two levels "weekday" and 'weekend" indicating whether a given date is a weekday or weekend day.

activityDataImputed$dateType <-  ifelse(as.POSIXlt(activityDataImputed$date)$wday %in% c(0,6), 'weekend', 'weekday')

## Make a panel plot 
averagedActivityDataImputed <- aggregate(steps ~ interval + dateType, data=activityDataImputed, mean)
ggplot(averagedActivityDataImputed, aes(interval, steps)) + 
    geom_line() + 
    facet_grid(dateType ~ .) +
    xlab("5-minute interval") + 
    ylab("avarage number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-1-4.png) 
