# Reproducible Research: Peer Assessment 1


```r
library(knitr)
library(lattice)
```


## Loading and preprocessing the data


```r
myDataf <- read.csv(unz("./activity.zip", "activity.csv"), header = TRUE,
                    colClasses = c("integer", "Date", "numeric"), sep = ",",
                    na.strings = "NA", quote = "\"")
```

## What is mean total number of steps taken per day?

Histogram of the total number of steps taken each day


```r
myDataf.steps <- aggregate(steps ~ date, myDataf, sum)
kable(head(myDataf.steps))
```



date          steps
-----------  ------
2012-10-02      126
2012-10-03    11352
2012-10-04    12116
2012-10-05    13294
2012-10-06    15420
2012-10-07    11015

```r
with(myDataf.steps, plot(date,steps, type="h",
                    xlab = "Date", ylab ="Steps",
                    col="red", main = "Histogram of total steps by day"))
```

![](PA1_template_files/figure-html/Histogram steps x day-1.png)<!-- -->

Mean and median number of steps taken each day

```r
mean(myDataf.steps$steps, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
median(myDataf.steps$steps, na.rm = TRUE)
```

```
## [1] 10765
```

## What is the average daily activity pattern?

Time series plot of the average number of steps taken

```r
myDataf.steps.average <- aggregate(steps ~ interval, myDataf, mean)
with(myDataf.steps.average, 
                    plot(interval, steps, type = "l",
                    xlab = "Interval", ylab = "Average Steps", col = "red",
                    main = "Average number of steps taken"))
```

![](PA1_template_files/figure-html/Plot Avg steps-1.png)<!-- -->

The 5-minute interval that, on average, contains the maximum number of steps

```r
myDataf.steps.average[which.max(myDataf.steps.average$steps),]
```

```
##     interval    steps
## 104      835 206.1698
```


## Imputing missing values

### Code to describe and show a strategy for imputing missing data  
1. Calculate total number of missing values in the dataset 

```r
NA.list <- is.na(myDataf$steps)
sum(NA.list)
```

```
## [1] 2304
```

2. Strategy for filling in all of the missing values in the dataset:  
We will fill in all missing NAs with the average number of steps in the same 5-min interval

3. New Dataset with missing data filled


```r
myDataf.filled <- myDataf                   #new data frame
myDataf.filled$id <- 1:nrow(myDataf.filled) #creates id column. will help order later
newNA <- merge(myDataf.filled[NA.list,], myDataf.steps.average, by="interval", sort = F) #add means 2 NAs
newNA <-  newNA[order(newNA$id), ]          #re-order
myDataf.filled$steps[NA.list] <- newNA$steps.y  #filled data frame
myDataf.filled$id <- NULL                   #drops id column
kable(head(myDataf.filled))                 #shows new data
```

     steps  date          interval
----------  -----------  ---------
 1.7169811  2012-10-01           0
 0.3396226  2012-10-01           5
 0.1320755  2012-10-01          10
 0.1509434  2012-10-01          15
 0.0754717  2012-10-01          20
 2.0943396  2012-10-01          25

4. Histogram of the total number of steps taken each day after missing values are imputed

```r
myDataf.filled.steps <- aggregate(steps ~ date, myDataf.filled, sum)
kable(head(myDataf.filled.steps))
```



date             steps
-----------  ---------
2012-10-01    10766.19
2012-10-02      126.00
2012-10-03    11352.00
2012-10-04    12116.00
2012-10-05    13294.00
2012-10-06    15420.00

```r
with(myDataf.filled.steps, plot(date,steps, type="h",
                    xlab = "Date", ylab ="Steps",
                    col="red", main = "Histogram of total steps by day (Missing Values Imputed)"))
```

![](PA1_template_files/figure-html/Histogram missing imputed-1.png)<!-- -->

 5. Calculate and report the mean and median total number of steps taken per day

```r
mean(myDataf.filled.steps$steps, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
median(myDataf.filled.steps$steps, na.rm = TRUE)
```

```
## [1] 10766.19
```
 
## Are there differences in activity patterns between weekdays and weekends?

1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.


```r
myDataf.filled$weekday <- as.factor(
                          ifelse(as.POSIXlt(myDataf.filled$date)$wday %in% c(0,6),
                          "weekend","weekday"))
```

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).


```r
myDataf.filled.average <- aggregate(steps ~ interval + weekday, myDataf.filled, mean)
xyplot(steps ~ interval | weekday, data = myDataf.filled.average, 
       layout = c(1, 2), type='l', ylab = "Number of steps")
```

![](PA1_template_files/figure-html/Panel Plot Weekdays vs Weekend-1.png)<!-- -->

