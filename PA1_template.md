---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---

## Loading and preprocessing the data

Load the data from the csv file
Dates need to be converted from chr to date format for processing

```r
data <- read.csv("activity.csv")
data$date <- as.Date(data$date, format = "%Y-%m-%d")
```

## What is mean total number of steps taken per day?

Group the number of steps taken per day 


```r
bydate <- aggregate(steps ~ date, data, sum) 
```

Plot a histogram of the daily step count


```r
with(bydate, hist(steps, xlab = "Steps per day", ylab = "Frequency", main = "Total steps per day"))
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

The summary table gives the mean and median of steps taken per day


```r
summary(bydate$steps)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##      41    8841   10765   10766   13294   21194
```


## What is the average daily activity pattern?  

First, aggregate the data by the number of steps in each 5 minute interval, by taking the average.  
Then plot the 5 minute intervals

```r
dailyavg <- aggregate(steps~interval, data, mean)
with(dailyavg, plot(interval, steps, type = "l", xlab = "5 minute intervals", ylab = "Number of steps", main = "Average number of steps per day"))
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

To find the interval with the maximum number of steps:


```r
maxrow <- which.max(dailyavg$steps)
dailyavg[maxrow,]
```

```
##     interval    steps
## 104      835 206.1698
```


## Imputing missing values

Calculate the total number of missing values in the dataset  

```r
sum(is.na(data$steps))
```

```
## [1] 2304
```

Devise strategy for filling in all the missing values in the dataset. A new dataset is created with teh missing data filled in.
Here, the mean of the 5 minute intervals is used. 


```r
idata <- data
idata$steps <- with(idata, ifelse(is.na(steps),dailyavg[dailyavg$interval == interval,2],steps))
```

Histogram of the daily step count:


```r
ibydate <- aggregate(steps ~ date, idata, sum)
with(ibydate, hist(steps, xlab = "Steps per day", ylab = "Frequency", main = "Total steps per day"))
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

Summary table gives the mean and median of steps taken per day:


```r
summary(ibydate$steps)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##      41    8860   10766   10766   13191   21194
```

Note that the mean has stayed the same since the mean was used to impute the data, but the median has increased.  

## Are there differences in activity patterns between weekdays and weekends?

Create a new factor variable with two levels, weekday and weekend indicating whether a given date is a weekday or weekend 

```r
idata$daytype <- with(idata, ifelse(weekdays(date) == "Saturday"| weekdays(date) == "Sunday", "weekend", "weekday"))
```

Summarising weekday and weekend data, the average step per interval is given in the plots below.

```r
wdailyavg <- aggregate(steps~interval + daytype, mean, data = idata)
library(lattice)
xyplot(steps~interval | daytype, data = wdailyavg, type = "l", layout = c(1,2), xlab = "5 min Interval", ylab = "Number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-12-1.png)<!-- -->

