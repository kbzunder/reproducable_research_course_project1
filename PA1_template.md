
---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---
## Introduction

It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the “quantified self” movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

The data for this assignment can be downloaded from the course web site:

Dataset: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip)
**The variables included in this dataset are:**

- steps: Number of steps taking in a 5-minute interval (missing values are coded as NA)
- date: The date on which the measurement was taken in YYYY-MM-DD format
- interval: Identifier for the 5-minute interval in which measurement was taken

## Loading and preprocessing the data

```r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
library(rio)
library(ggplot2)
library(data.table)
```

```
## 
## Attaching package: 'data.table'
```

```
## The following objects are masked from 'package:dplyr':
## 
##     between, first, last
```

```r
url<-'https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip'
data<-import(url)
```
Removing missing values

```r
cleaned<-filter(data, !is.na(steps))
cleaned$date<-as.Date(cleaned$date)
```
## What is mean total number of steps taken per day?
For this part of the assignment, you can ignore the missing values in the dataset.

Calculate the total number of steps taken per day

```r
aggregated<-aggregate(cleaned$steps, by=list(cleaned$date), sum)
colnames(aggregated)<-c('date','steps')
```
If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day

```r
ggplot(aggregated, aes(x = steps)) +
  geom_histogram(fill = "blue", binwidth = 1000) +
  labs(title = "Steps a day", x = "steps", y = "frequency")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->
Calculate and report the mean and median of the total number of steps taken per day

```r
mean_steps<-mean(aggregated$steps)
mean_steps
```

```
## [1] 10766.19
```

```r
median_steps<-median(aggregated$steps)
median_steps
```

```
## [1] 10765
```

## What is the average daily activity pattern?
Make a time series plot (i.e. type = “l”) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
aver_steps <- aggregate(steps ~ interval, data = cleaned, mean)
plot(aver_steps$interval, aver_steps$steps, type = "l", lwd = 2, col = "red",
     main = "Average number of steps",
     xlab = "interval", ylab = "average steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->
Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
which.max(aver_steps$steps)
```

```
## [1] 104
```

```r
interv<-aver_steps$interval[104]
```
Interval number 835 contains the maximum number of steps.

## Imputing missing values

Note that there are a number of days/intervals where there are missing values (coded as 
NA
NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with 
NA)

```r
nuber_of_missing_values<-sum(is.na(data$steps))
```
Number of missing value in a dataset are `nuber_of_missing_values`

Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.


```r
mean_interval<-aggregate(steps~interval, data=cleaned, mean)
```
Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
new_df <- data
for (i in 1:nrow(new_df)) {
  if(is.na(new_df$steps[i])) {
    dat <- mean_interval$steps[which(mean_interval$interval == new_df$interval[i])]
    new_df$steps[i]<-dat
  }
}
```
Make a histogram of the total number of steps taken each day and 

```r
aggregated_new<-aggregate(new_df$steps, by=list(new_df$date), sum)
colnames(aggregated_new)<-c('date','steps')
ggplot(aggregated_new, aes(x = steps)) +
  geom_histogram(fill = "purple", binwidth = 1000) +
  labs(title = "Steps a day", x = "steps", y = "frequency")
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)<!-- -->
Calculate and report the mean and median total number of steps taken per day. 

```r
mean_steps_new<-mean(aggregated_new$steps)
mean_steps_new
```

```
## [1] 10766.19
```

```r
median_steps_new<-median(aggregated_new$steps)
median_steps_new
```

```
## [1] 10766.19
```

Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

| median steps(imputed) | median steps   |             
| :----------------     | :----------:   |
|   10766.19            |   10765        |


| mean steps(imputed)   |   mean steps   |          
| :----------------     | :----------:   |
|` 10766.19             |    10766.19    |

## Are there differences in activity patterns between weekdays and weekends?

Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.


```r
weekday<-weekdays(as.Date(new_df$date,'%Y-%m-%d'))
new_df_weekdays<-mutate(new_df, weekdays=weekday)
for (i in 1:length(weekday)){
  if(weekday[i]=="Saturday"||weekday[i]=="Sunday"){
    new_df_weekdays$type_day[i]<-"Weekend"
  } else{
    new_df_weekdays$type_day[i]<-"Weekday"
  }
}

new_df_weekdays$type_day<-as.factor(new_df_weekdays$type_day)

steps_per_day_type_imputed<-aggregate(steps~interval+type_day,new_df_weekdays,mean)
```
Make a panel plot containing a time series plot 


```r
plot_1 <- ggplot(steps_per_day_type_imputed, aes(interval, steps)) +
  geom_line(stat = "identity", aes(colour = type_day)) +
  facet_grid(type_day~ ., scales="fixed", space="fixed") +
  labs(x="intervals", y="steps") +
  ggtitle("Average Number of Steps by Interval and Day Type")
print(plot_1)
```

![](PA1_template_files/figure-html/unnamed-chunk-14-1.png)<!-- -->
We can see a difference between average  number of steps on weekends and weekdays: on weekends we don't se picks of activity, and the user starts moving later.
