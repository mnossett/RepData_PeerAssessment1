---
title: "Reproducible Research: Peer Assessment 1"  
author: "MLN"  
date: "2023-11-27"  
output: 
  html_document: 
    keep_md: yes
---

Project Requirements:  
1. Code for reading in the dataset and/or processing the data  
2. Histogram of the total number of steps taken each day  
3. Mean and median number of steps taken each day  
4. Time series plot of the average number of steps taken  
5. The 5-minute interval that, on average, contains the maximum number of steps  
6. Code to describe and show a strategy for imputing missing data  
7. Histogram of the total number of steps taken each day after missing values are imputed  
8. Panel plot comparing the average number of steps taken per 5-minute interval across weekdays
   and weekends  

--------------------------------------------------------------------------------



--------------------------------------------------------------------------------

## 1. Read in the Dataset


```r
zipurl = "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
download.file(zipurl, destfile="./activity.zip", mode="wb")
unzip(zipfile = "./activity.zip", exdir = "./")
data <- read.csv("activity.csv", na.strings="NA", stringsAsFactors=FALSE)
```

--------------------------------------------------------------------------------

## 2. Histogram of the total number of steps taken each day


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
daily_steps = data %>% select(date, steps) %>% group_by(date) %>% 
     summarise(across(everything(), sum))
hist(daily_steps$steps, main = "Frequency of Steps per Day", xlab = "Number of Steps", 
     ylim= c(0, 30))
```

![](figs/fig-daily_steps-1.png)<!-- -->

--------------------------------------------------------------------------------

## 3. Mean and median number of steps taken each day


```r
mean(daily_steps$steps, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
median(daily_steps$steps, na.rm = TRUE)
```

```
## [1] 10765
```

--------------------------------------------------------------------------------

## 4. Time series plot of the average number of steps taken


```r
data1 = na.omit(data)
daily_pattern = data1 %>% group_by(interval) %>% summarise(average = mean(steps))
plot(daily_pattern$interval, daily_pattern$average, type="l", 
     xlab = "Interval", ylab = "Average Steps", main = "Average Daily Activity Pattern")
```

![](figs/fig-daily_pattern-1.png)<!-- -->

--------------------------------------------------------------------------------

## 5. The 5-minute interval that, on average, contains the maximum number of steps


```r
daily_pattern[which.max(daily_pattern$average),]
```

```
## # A tibble: 1 × 2
##   interval average
##      <int>   <dbl>
## 1      835    206.
```

On average, most steps are take at the 0835 (8:35am) interval.

--------------------------------------------------------------------------------

## 6. Code to describe and show a strategy for imputing missing data

Calculate and report the total number of rows with NAs.


```r
data.frame(sapply(data, function(y) sum(length(which(is.na(y))))))
```

```
##          sapply.data..function.y..sum.length.which.is.na.y.....
## steps                                                      2304
## date                                                          0
## interval                                                      0
```

Devise a strategy for filling in all of the missing values in the dataset. *This
code will replace NA values with the average (or mean) number of steps
calculated across* *all other obverations within the same interval*


```r
library(dplyr)
im_data <- data %>% group_by(interval) %>% mutate(steps = ifelse(is.na(steps), 
                                                                 mean(steps, na.rm=TRUE), steps))
```

   *im_data is equal to the original dataset but with the missing data filled in.*

--------------------------------------------------------------------------------

## 7. Histogram of the total number of steps taken each day after missing values are imputed


```r
library(dplyr)
daily_steps2 = im_data %>% select(date, steps) %>% group_by(date) %>% 
     summarise(across(everything(), sum))
```

```
## Adding missing grouping variables: `interval`
```

```r
hist(daily_steps2$steps, main = "Frequency of Steps per Day (with imputed data)", 
     xlab = "Number of Steps", yaxt = "n")
axis(2, at = seq(0, 40, 5))
```

![](figs/fig-daily_steps_with_imputed_data-1.png)<!-- -->

Calculate and report the mean and median total number of steps taken per day. Do
these values differ from the estimates from the first part of the assignment?
What is the impact of imputing missing data on the estimates of the total daily
number of steps?

mean and median of imputed data:


```r
mean(daily_steps2$steps)
```

```
## [1] 10766.19
```

```r
median(daily_steps2$steps)
```

```
## [1] 10766.19
```

mean and median of raw data with NAs removed:


```r
mean(daily_steps$steps, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
median(daily_steps$steps, na.rm = TRUE)
```

```
## [1] 10765
```

The method of imputing data can having ranging affects on the calculations.Since I used the mean
across each interval, the mean values are the same for both datasets.
The medians, however, did change slightly

--------------------------------------------------------------------------------

## 8. Panel plot comparing the avg # of steps taken per 5-minute interval across weekdays & weekends


```r
im_data$date = as.Date(im_data$date)
data3 = im_data %>% group_by(weekdays(date))
data3 = rename(data3, c("day" = "weekdays(date)"))

weekdays = subset(data3, day == "Monday" | day == "Tuesday" | day == "Wednesday" | 
                          day == "Thursday" | day == "Friday")
weekdays = weekdays %>% group_by(interval) %>% summarise(average = mean(steps))

weekends = subset(data3, day == "Saturday" | day == "Sunday")
weekends = weekends %>% group_by(interval) %>% summarise(average = mean(steps))

par(mfrow = c(2,1))
plot(weekdays$interval, weekdays$average, type="l", xlab = "Interval", ylab = "Average Steps",
     main = "Average Activity Pattern on Weekdays (M-F)")
plot(weekends$interval, weekends$average, type="l", xlab = "Interval", ylab = "Average Steps", 
     main = "Average Activity Pattern on Weekends (Sat & Sun)")
```

![](figs/fig-unnamed-chunk-7-1.png)<!-- -->

--------------------------------------------------------------------------------
