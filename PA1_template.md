# Reproducible Research: Peer Assessment 1



## Loading and preprocessing the data

First we load the data from the `activity.csv`. We use `dplyr` library for easier
manipulation with the data. The dates are handled with the `lubridate` library.


```r
library(dplyr)
library(lubridate)
activity <- tbl_df(read.csv("activity.csv")) %>% mutate(date = ymd(date))
```


## What is mean total number of steps taken per day?

First, we calculate total number of steps taken each day; we omit the incomplete cases, 
i.e., when the number of steps is not reported (missing in the data).


```r
total_steps <- subset(activity, complete.cases(activity)) %>% 
  group_by(date) %>% 
  summarize(total = sum(steps))
```

Next, we make a histogram of the total number of steps in order to see its variance 
and distribution.


```r
hist(total_steps$total, breaks = 10, main = "Histogram of total number of steps taken each day",
     xlab = "Total number of steps per day")
```

![](PA1_template_files/figure-html/hist-1.png) 

Finally, we compute *mean* and *median* values of the total number of steps.


```r
mean(total_steps$total)
```

```
## [1] 10766.19
```

```r
median(total_steps$total)
```

```
## [1] 10765
```

## What is the average daily activity pattern?

Below, we can see as the number of steps changes throught the day (averaged across all the days).


```r
steps_per_interval <- activity %>% 
  group_by(interval) %>% 
  summarize(mean = mean(steps, na.rm = TRUE))
plot(steps_per_interval, type = 'l', main = "Average number of steps per interval",
     xlab = "Time interval", ylab = "Average number of steps")
```

![](PA1_template_files/figure-html/timeseries-1.png) 

The 5-minute interval with maximum number of steps is as follows:


```r
max_steps <- max(steps_per_interval$mean)
subset(steps_per_interval, mean == max_steps)$interval
```

```
## [1] 835
```

## Imputing missing values

The total number of rows with missing values:


```r
sum(!complete.cases(activity))
```

```
## [1] 2304
```

We fill the missing values of steps with the average number of steps
for the given 5-minute interval.


```r
interval_means <- sapply(activity$interval, 
                         function(x) subset(steps_per_interval, interval == x)$mean)
missing <- is.na(activity$steps)

activity_imputed <- activity
activity_imputed[missing, ]$steps <- interval_means[missing]
```

Next, we calculate total number of steps on the imputed data and visualize the histogram
in order to compare it with the original data.


```r
total_steps_imputed <- activity_imputed %>% 
  group_by(date) %>% 
  summarize(total = sum(steps))

hist(total_steps_imputed$total, breaks = 10, main = 
       "Histogram of total number of steps per day with no missing values",
     xlab = "Total number of steps per day")
```

![](PA1_template_files/figure-html/histogram_imputed-1.png) 

For the sake of comparison, we also compute new *mean* and *median* values of the total number of steps.


```r
mean(total_steps_imputed$total)
```

```
## [1] 10766.19
```

```r
median(total_steps_imputed$total)
```

```
## [1] 10766.19
```

## Are there differences in activity patterns between weekdays and weekends?
