Reproducable Research Peer Assessment 1
========================================================

## Download the data 
Note: You have to change https to http to get this to work in knitr


```r
target_url <- "http://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
target_localfile = "ActivityMonitoringData.zip"
if (!file.exists(target_localfile)) {
  download.file(target_url, destfile = target_localfile) 
}
```

Unzip the file to the temporary directory

```r
unzip(target_localfile, exdir="extract", overwrite=TRUE)
```

List the extracted files

```r
list.files("./extract")
```

```
## [1] "activity.csv"
```

## Loading and preprocessing the data
-Load the extracted data into R

```r
activity.csv <- read.csv("./extract/activity.csv", header = TRUE)
```

-Remove rows that contain NA

```r
activity1 <- activity.csv[complete.cases(activity.csv),]
str(activity1)
```

```
## 'data.frame':	15264 obs. of  3 variables:
##  $ steps   : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 2 2 2 2 2 2 2 2 2 2 ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

## What is mean total number of steps taken per day?
-Use a histogram to view the number of steps taken each day, but first create an aggregrate to get the mean per day


```r
histData <- aggregate(steps ~ date, data = activity1, sum)
```

and now the histogram

```r
h <- hist(histData$steps,  # Save histogram as object
          breaks = 11,  # "Suggests" 11 bins
          freq = T,
          col = "thistle1", # Or use: col = colors() [626]
          main = "Histogram of Activity",
          xlab = "Number of daily steps")
```

![plot of chunk unnamed-chunk-7](figure/unnamed-chunk-7.png) 

-Obtain the Mean and Median of the daily steps.

I did compare the mean to the summary (to reconcile, but not shown), however they were different.  The reason was the digits by default are 3, so they created different numbers.  Once I added the options command below this fixed the issue:


```r
options(digits=12)
steps <- histData$steps
mean(steps)
```

```
## [1] 10766.1886792
```

```r
median(steps)
```

```
## [1] 10765
```

## What is the average daily activity pattern?

Obtain the mean by again using aggregate, but the function being mean and by interval, not date:


```r
plotData <- aggregate(steps ~ interval, data = activity1, mean)
plot(plotData, type="l", main=" Average number of steps taken, averaged across all days")
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9.png) 

-Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

To find the maximum I will use an SQLDF command, as this is simple

```r
library(sqldf)
```

```
## Loading required package: gsubfn
## Loading required package: proto
## Loading required package: RSQLite
## Loading required package: DBI
## Loading required package: RSQLite.extfuns
```

```r
sqldf("select interval, max(steps) steps from plotData")
```

```
## Loading required package: tcltk
```

```
##   interval         steps
## 1      835 206.169811321
```


##Imputing missing values
Find the amount of rows that contain missing values.  As I removed them earlier this is a simple subtraction:


```r
MissingValues <- (nrow(activity.csv) - nrow(activity1))
MissingValues
```

```
## [1] 2304
```
