# Reproducible Research: Peer Assessment 1


### Loading and preprocessing the data
1. Load the data

```r
    d <- read.csv("activity.csv", stringsAsFactors = FALSE)
```


2. Transform data (create a grouping by date and a grouping by interval)

```r
    g <- group_by(d, date)
    i <- group_by(d, interval)
```

### What is mean total number of steps taken per day?
1. Calculate the total number of steps per day

```r
    g <- summarize(g, sum(steps, na.rm = TRUE))
    names(g) <- c("date", "total_steps")
```


2. Make a histogram of the total number of steps per day

```r
    hist(g$total_steps)
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png) 


3. Calculate the mean and median of the total number of steps per day

```r
    mean(g$total_steps)
```

```
## [1] 9354.23
```

```r
    median(g$total_steps)
```

```
## [1] 10395
```


## What is the average daily activity pattern?
1. Make a time series plot of the average steps per five min interval

```r
    i <- summarize(i, mean(steps, na.rm=TRUE))
    names(i) <- c("interval","avg_steps")
    plot(i$interval, i$avg_steps, type="l")
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png) 


2. Which interval has the max number of steps?

```r
    subset(i, i$avg_steps==max(i$avg_steps))
```

```
## Source: local data frame [1 x 2]
## 
##   interval avg_steps
##      (int)     (dbl)
## 1      835  206.1698
```


### Imputing missing values
1. Calculate the total number of missing values in the dataset

```r
    sum(is.na(d$steps))
```

```
## [1] 2304
```


2. Devise a strategy for filling in missing values

   * For each missing value, use the average for that interval across all days


3. Create a new dataset with missing values

```r
    n <- d
    n$steps <- ifelse(is.na(n$steps), i$avg_steps, n$steps)
```


4. Make a histogram of the total number of steps per day with the new dataset

```r
    ng <- group_by(n, date)
    ng <- summarize(ng, sum(steps))
    names(ng) <- c("date", "total_steps")
    hist(ng$total_steps) 
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png) 

   * Calculate the mean and median using the new dataset.
   
   * The mean and median are higher than the original dataset.  Because there are eight full days with missing values, total steps for that day is the mean for all days and the median ends up being the same as the mean.

```r
    mean(ng$total_steps)
```

```
## [1] 10766.19
```

```r
    median(ng$total_steps)
```

```
## [1] 10766.19
```
   
   
### Are there differences in activity patterns between weekdays and weekends?
1. Create a factor variable indicating weekend vs weekday

```r
    n$dow <- as.POSIXlt(n$date)$wday
    n$dowf <- factor(n$dow %in% 1:5, levels=c(FALSE,TRUE), labels = c("weekend","weekday"))
```


2. Make a panel plot containing a time series plot of the 5-minute interval and the average number of steps taken, averaged across all weekday days or weekend days.

```r
    nw <- group_by(n, dowf, interval)
    nw <- summarize(nw, mean(steps))
    names(nw) <- c("dowf","interval","avg_steps")
    p <- ggplot(nw, aes(x=interval, y=avg_steps)) + geom_line() + facet_wrap(~dowf, ncol=1)
    print(p)
```

![](PA1_template_files/figure-html/unnamed-chunk-14-1.png) 
