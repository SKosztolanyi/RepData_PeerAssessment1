# Peer Assignment 1 for Reproducible Research
##Stefan Kosztolanyi

This file is created using Rstuio, knitr and ggplot2 packages in a R markdown file.
In this file I answer various questions about a dataset and make plots to visualise the answers.
The dataset as well as questions are provided by Coursera's course Reproducible Research, which is part of a Data Science Specialization by John Hopkins University.

## Introduction

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.


## Data

The data for this assignment can be downloaded from the course web site:

Dataset: Activity monitoring data [52K]
The variables included in this dataset are:

steps: Number of steps taking in a 5-minute interval (missing values are coded as NA)

date: The date on which the measurement was taken in YYYY-MM-DD format

interval: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

## Preparing the environment

I will be showing all the code and plots in this document openly, therefore I want to set echo=TRUE value for the whole document:

```r
library(knitr)
```

```
## Warning: package 'knitr' was built under R version 3.2.2
```

```r
require("knitr")
opts_chunk$set(echo=TRUE, results="asis")
```

I will be making the plots in ggplot2, so I can load the package:

```r
library(ggplot2)
```

## Loading and preprocessing the data

The dataset I will be working with is publicly available to download:
Here is the adress to download: https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip

I set working directory to the one where I donwloaded the file

```r
setwd("C:/Users/stefan/Google Drive OldZZZ/Coursera/Data Science/Reproducible research")
```

I then manually extract the zip file with winrar software in the working folder

With this command I read the whole csv file into r and call the dataset "activity"

```r
activity = read.csv("activity.csv")
str(activity)
```

'data.frame':	17568 obs. of  3 variables:
 $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
 $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
 $ interval: int  0 5 10 15 20 25 30 35 40 45 ...

## What is mean total number of steps taken per day?

To answer these questions, I need to further manipulate my dataframe and create the correct variables
1. Calculate the total number of steps taken per day
2. Make a histogram of the total number of steps taken each day
3. Calculate and report the mean and median of the total number of steps taken per day

I create a new data frame that will be without the NA values and call the data frame "ignore"

```r
ignore<- na.omit(activity)
```

Converting the format of ignore dataframe columns into desired format for better manipulation

```r
ignore$steps<- as.numeric(ignore$steps)
ignore$date<- as.Date(ignore$date, format="%Y-%m-%d")
ignore$interval<- as.factor(ignore$interval)
str(ignore)
```

'data.frame':	15264 obs. of  3 variables:
 $ steps   : num  0 0 0 0 0 0 0 0 0 0 ...
 $ date    : Date, format: "2012-10-02" "2012-10-02" ...
 $ interval: Factor w/ 288 levels "0","5","10","15",..: 1 2 3 4 5 6 7 8 9 10 ...
 - attr(*, "na.action")=Class 'omit'  Named int [1:2304] 1 2 3 4 5 6 7 8 9 10 ...
  .. ..- attr(*, "names")= chr [1:2304] "1" "2" "3" "4" ...

I create a dataframe that will use the sum function(for total) for the steps taken for every day that is available. With str() function I can take a quick look at the dataset. Or I can double click the data "total_days_steps" dataset set in Data and see the whole table in new window in R studio.

```r
total_days_steps<- aggregate.data.frame(ignore$steps, list(date=ignore$date), sum)
colnames(total_days_steps)<- c("date", "total_steps")
str(total_days_steps)
```

'data.frame':	53 obs. of  2 variables:
 $ date       : Date, format: "2012-10-02" "2012-10-03" ...
 $ total_steps: num  126 11352 12116 13294 15420 ...

I can start plotting this dataframe. As indicated, I will be using ggplot for plotting.

```r
qplot(total_days_steps$total_steps, geom="histogram", main = "Total steps per day", xlab="steps", fill=I("blue"), col=I("red"), alpha=I(.5))
```

```
## stat_bin: binwidth defaulted to range/30. Use 'binwidth = x' to adjust this.
```

![](PA1_template_files/figure-html/ignore histogram-1.png) 

Lastly I calculate the mean and median of this dataframe

```r
meanTDS<-mean(total_days_steps$total_steps)
medianTDS<-median(total_days_steps$total_steps)
meanTDS
```

[1] 10766.19

```r
medianTDS
```

[1] 10765

## What is the average daily activity pattern?

1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

To answer the next question I create an aggregated data frame based on the "ignore" data frame:

```r
steps_at_interval<- aggregate(ignore$steps, list(interval=as.numeric(as.character(ignore$interval))), mean)
colnames(steps_at_interval)<- c("interval", "mean")
```

making time series plot for the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis):

```r
ggplot(steps_at_interval, aes(interval, mean)) + geom_line(color="red") + xlab("interval") + ylab("steps taken")
```

![](PA1_template_files/figure-html/interval plot ignoring NAs-1.png) 

To answer the second question, I subset the line with the interval with biggest mean of steps

```r
steps_at_interval[(steps_at_interval$mean == max(steps_at_interval$mean)), ]
```

    interval     mean
104      835 206.1698

## Imputing missing values

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)
There are only NA values in the step column, so I can count the number of rows with NA values like this:

```r
sum(is.na(activity$steps))
```

[1] 2304

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
I chose to substitute a missing value with mean for the interval. I do it like this:

```r
Activity2<- activity
for (i in 1:nrow(Activity2)) {
     if (is.na(activity$steps[i])) {
           Activity2$steps[i]<- steps_at_interval[which(Activity2$interval[i] == steps_at_interval$interval), ]$mean
     }
}
Activity2$steps<- as.numeric(Activity2$steps)
Activity2$date<- as.Date(Activity2$date, format="%Y-%m-%d")
Activity2$interval<- as.factor(Activity2$interval)
str(Activity2)
```

'data.frame':	17568 obs. of  3 variables:
 $ steps   : num  1.717 0.3396 0.1321 0.1509 0.0755 ...
 $ date    : Date, format: "2012-10-01" "2012-10-01" ...
 $ interval: Factor w/ 288 levels "0","5","10","15",..: 1 2 3 4 5 6 7 8 9 10 ...

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.
The new dataframe is called Activity2

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?
Based on Activity2 dataframe I create an aggregated data frame total_days_steps2. Based on this dataframe I make a new histogram


```r
total_days_steps2<- aggregate.data.frame(Activity2$steps, list(date=Activity2$date), sum)
colnames(total_days_steps2)<- c("date", "total_steps")
qplot(total_days_steps2$total_steps, geom="histogram", main = "Total steps per day 2", xlab="steps", fill=I("blue"), col=I("red"), alpha=I(.5))
```

```
## stat_bin: binwidth defaulted to range/30. Use 'binwidth = x' to adjust this.
```

![](PA1_template_files/figure-html/dataframe and histograms 2-1.png) 

I calculate the new median and mean in the data frame with filled in missing values

```r
meanTDS2<-mean(total_days_steps2$total_steps)
medianTDS2<-median(total_days_steps2$total_steps)

meanTDS2
```

[1] 10766.19

```r
medianTDS2
```

[1] 10766.19

There is almost no change in the mean and median from the first dataframe and this dataframe. The smll difference in median  is insignificant.

## Are there differences in activity patterns between weekdays and weekends?

1. I Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.
But before that, I need to change the environment time to english, otherwise I would get Slovak(my language) names of days:

```r
Sys.setlocale("LC_TIME", "English")
```

[1] "English_United States.1252"

The next step is to add a new columnt to the Activity2 dataframe, that will contain name of the days in week and then I can change the days into two variables - either weekday(Mo-Fri) or weekend(So, Sun).

```r
Activity2$weekdays<- factor(format(Activity2$date, "%A"))
levels(Activity2$weekdays) <- list(weekday = c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday"),
                                   weekend = c("Saturday", "Sunday"))
str(Activity2$weekdays)
```

 Factor w/ 2 levels "weekday","weekend": 1 1 1 1 1 1 1 1 1 1 ...

I prepare a new data frame for plotting:

```r
weekday_frame<- aggregate(Activity2$steps, list(interval=as.numeric(as.character(Activity2$interval)), weekdays=Activity2$weekdays), mean)
colnames(weekday_frame)<- c("interval", "weekdays", "average_steps")
```

2. I Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data:

```r
ggplot(weekday_frame, aes(interval, average_steps)) + geom_line(color="red") + xlab("interval") + ylab("average_steps") + facet_wrap(~weekdays, nrow=2, ncol=1)
```

![](PA1_template_files/figure-html/weekdays plot-1.png) 

From this plot it is evident, that there is difference in average number of steps during weekend and during weekdays. Although it is not obvious just from this graphs, what is the reason behind this difference, as we would need further data to find the reason.

## Conclusion
In this assignment I worked with dataframe activity, which I further manipulated and created plots based on different variables. I created a new subdataframes on the way and added mean values into original NA values.
