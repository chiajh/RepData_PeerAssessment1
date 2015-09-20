# Reproducible Research: Peer Assessment 1

### Basic settings

```r
echo = TRUE  # Always make code visible
options(scipen = 1)  # Turn off scientific notations for numbers
```

## Loading and preprocessing the data

```r
library(ggplot2)
library(knitr)
myData <- read.csv("activity.csv", header=TRUE, sep =",")
myData$date <- as.Date(myData$date)
```

## What is mean total number of steps taken per day?
For this part of the assignment, you can ignore the missing values in the dataset.

1. Calculate the total number of steps taken per day

```r
totalStepByDay <- aggregate(steps ~ date, myData, FUN = sum, na.rm = TRUE) 
```

2. Make a histogram of the total number of steps taken each day

```r
plotHist <- ggplot(data = totalStepByDay, aes(x = steps)) + geom_histogram(binwidth = 2000) +
            labs(title = "Total Number Of Steps Taken Each Day", x = "Steps", y = "Count")
plotHist
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png) 

3. Calculate and report the mean and median total number of steps taken per day

```r
mean(totalStepByDay$steps) 
```

```
## [1] 10766.19
```

```r
median(totalStepByDay$steps)
```

```
## [1] 10765
```

## What is the average daily activity pattern?
1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
avgStepByInterval <- aggregate(steps ~ interval, myData, FUN = mean, na.rm = TRUE) 
plotLine <- ggplot(avgStepByInterval, aes(interval, steps)) +  geom_line(color = "blue", size = 1) + 
            labs(title = "Time Series Plot Of The 5-minute Interval", x = "5-minute intervals", y = "Average number of steps taken")
plotLine
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png) 

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
avgStepByInterval[avgStepByInterval$steps == max(avgStepByInterval$steps), "interval"]
```

```
## [1] 835
```

## Imputing missing values
1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
sum(is.na(myData$steps))
```

```
## [1] 2304
```

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

The strategy is to create a new dataset which is equal to the original dataset and fill up the missing data (NA) for "steps" column by using the mean for "5-minute interval" from avgStepByInterval dataset above.

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
myDataNew <- myData 
for (i in 1:nrow(myDataNew)) {
  if (is.na(myDataNew$steps[i])) {
    myDataNew$steps[i] <- avgStepByInterval[which(myDataNew$interval[i] == avgStepByInterval$interval), "steps"]
  }
}
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
totalStepByDayNew <- aggregate(steps ~ date, myDataNew, FUN = sum, na.rm = TRUE) 
plotHist <- ggplot(data = totalStepByDayNew, aes(x = steps)) + geom_histogram(binwidth = 2000) +
         labs(title = "Total Number Of Steps Taken Each Day", x = "Steps", y = "Count")
plotHist
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png) 

```r
mean(totalStepByDayNew$steps) 
```

```
## [1] 10766.19
```

```r
median(totalStepByDayNew$steps)
```

```
## [1] 10766.19
```
Answer: The mean and the median are almost the same after replacing missing values with the mean for "5-minute interval". So the median value increased after this strategy of replacing the missing value. Also, the new mean and old mean of total steps taken per day are the same but the new median is greater than old median.

## Are there differences in activity patterns between weekdays and weekends?
1. Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```r
myDataNew$weekday <- as.factor(ifelse(weekdays(myDataNew$date) %in% c("Saturday", "Sunday"), "Weekend", "Weekday")) 
```

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). The plot should look something like the following, which was created using simulated data:

```r
avgStepByIntervalWeekday <- aggregate(steps ~ interval + weekday, myDataNew, FUN = mean, na.rm = TRUE) 
library(lattice)
xyplot(steps ~ interval | weekday, data = avgStepByIntervalWeekday,
       type = 'l',  xlab = 'Interval', ylab = 'Number of steps', layout = c(1,2))
```

![](PA1_template_files/figure-html/unnamed-chunk-12-1.png) 
