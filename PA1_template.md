---
title: "Coursera Reproducible Research, Project 1"
output: html_document
---

Reading and cleaning data using read.csv() and complete.cases() respectively

```r
df <- read.csv("activity.csv")
dfComp <- df[complete.cases(df),]
```

***

## Question: What is mean total number of steps taken per day?

#### 1. Calculate the total number of steps taken per day

```r
dfSteps <- aggregate(dfComp$steps, by=list(dfComp$date), FUN=sum)
```
Changing the column names of the resulting data frame which holds the total number of steps corresponding to each date

```r
colnames(dfSteps) <- c("Date", "Total_Number_of_Steps")
```

#### 2. Make a histogram of the total number of steps taken each day

```r
hist(dfSteps$Total_Number_of_Steps, main = "Histogram of Steps per Day", xlab = "Total Number of Steps", breaks = 20)
```

<img src="PA1_template_files/figure-html/unnamed-chunk-3-1.png" width="672" />

#### 3. Calculate and report the mean and median of the total number of steps taken per day

```r
mean(dfSteps$Total_Number_of_Steps, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
median(dfSteps$Total_Number_of_Steps, na.rm = TRUE)
```

```
## [1] 10765
```

***

## Question: What is the average daily activity pattern?

#### 1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis). (The aggregate() function produces columns "Group.1" and "x" representing intervals and the total number of steps corresponding to these intervals respectively.)

```r
dfInt <- aggregate(dfComp$steps, by=list(dfComp$interval), FUN=mean)
colnames(dfInt) <- c("Interval", "Mean_Total_Number_of_Steps")
plot(dfInt$Interval, dfInt$M, type = "l", main = "Average number of steps taken at each interval", xlab = "Interval", ylab = "Steps")
```

<img src="PA1_template_files/figure-html/unnamed-chunk-5-1.png" width="672" />

#### 2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
dfInt[which.max(dfInt$Mean_Total_Number_of_Steps),]
```

```
##     Interval Mean_Total_Number_of_Steps
## 104      835                   206.1698
```

***

## Question: Imputing missing values

#### 1.Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)
The difference between the number of rows in df (data frame with the NA values) and dfComp (the "cleanded" data frame equals to the number of rows with NA values)

```r
dim(df)[1] - dim(dfComp)[1]
```

```
## [1] 2304
```

#### 2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
I wanted to replace the NA values with means of correponding dates, however not every date had a mean

```r
NAs <- is.na(df$steps)
meanDates <- tapply(df$steps, df$date, mean, na.rm=TRUE)
sum(is.na(meanDates))
```

```
## [1] 8
```

Thefore I decided on replacing NA values with the means of corresponding 5-min intervals.

```r
meanInt <- tapply(df$steps, df$interval, mean, na.rm=TRUE)
sum(is.na(meanInt))
```

```
## [1] 0
```
(tapply is used in order to calculate means of each interval and the "df[x]"" notation when updating the cells with NA values.)

#### 3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
dfNoNa <- df
dfNoNa$steps[NAs] <- meanInt[as.character(dfNoNa$interval[NAs])]
```
(Means were extracted from the meanInt array. Since intervals are denoted as "0, 1, 2, ...", these needed to be coerced to "characters" so that df[x] notation gives the right output [i.e. the mean value corresponding to the fifth interval instead of the fifth value in the array])

#### 4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?
A new data frame for the total number of steps taken each day is calculated from the data frame (dfNoNa), in which the NAs were replaced with interval means.

```r
dfNoNaSteps <- aggregate(dfNoNa$steps, by=list(dfNoNa$date), FUN=sum)
colnames(dfNoNaSteps) <- c("Date", "Total_Number_of_Steps")
```

Two histograms (with NA values removed and with NA values replaced with mean of intervals) are drawn side by side for comparison purposes

```r
par(mfrow=c(1,2))
hist(dfSteps$Total_Number_of_Steps, main = "Histogram of Steps per Day\n(NAs removed)", xlab = "Total Number of Steps", breaks = 20, ylim = c(0,20))
hist(dfNoNaSteps$Total_Number_of_Steps, main = "Histogram of Steps per Day\n(NAs replaced with interval means)", xlab = "Total Number of Steps", breaks = 20, ylim = c(0,20))
```

<img src="PA1_template_files/figure-html/unnamed-chunk-12-1.png" width="672" />

Sum, Mean and median values corresponding to data frames with NAs removed and NAs replaced:
Data with NAs removed:

```r
sum(dfSteps$Total_Number_of_Steps)
```

```
## [1] 570608
```

```r
mean(dfSteps$Total_Number_of_Steps, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
median(dfSteps$Total_Number_of_Steps, na.rm = TRUE)
```

```
## [1] 10765
```
Data with NAs replaced with interval means":

```r
sum(dfNoNaSteps$Total_Number_of_Steps)
```

```
## [1] 656737.5
```

```r
mean(dfNoNaSteps$Total_Number_of_Steps, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
median(dfNoNaSteps$Total_Number_of_Steps, na.rm = TRUE)
```

```
## [1] 10766.19
```

Imputing the missing data or using interval means instead of NA values did not have pronounced effect on the mean and the median of total daily number of steps. Total number of steps, on the other hand, obviously increased

***

## Question: Are there differences in activity patterns between weekdays and weekends?

#### 1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
dfWeek <- df
dfWeek$day <- weekdays(as.Date(dfWeek$date, format = "%Y-%m-%d"))
dfWeek$day <- ifelse(dfWeek$day == "Saturday" | dfWeek$day == "Sunday", "weekend", "weekday")
```

#### 2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

```r
dfWeekInt <- aggregate(steps ~ interval + day, data = dfWeek, FUN = mean, na.action = na.omit)
colnames(dfWeekInt) <- c("Interval", "Day", "Mean_Total_Number_of_Steps")
```

It is rather difficult to plot factorized data in the base R graphics system. I used ggplot2 for visualizing mean of total number of steps per interval per weekday / weekend.

```r
library(ggplot2)
ggplot(data = dfWeekInt, aes(x = Interval, y = Mean_Total_Number_of_Steps, color = as.factor(Day))) +
    geom_line() +
    scale_color_discrete(name="Days")+
    labs(title = "Average of Total Number of Steps per Interval", y = "Mean of Total Number of Steps")
```

<img src="PA1_template_files/figure-html/unnamed-chunk-17-1.png" width="672" />

The same graphic in two rows.

```r
library(ggplot2)
ggplot(data = dfWeekInt, aes(x = Interval, y = Mean_Total_Number_of_Steps)) +
    geom_line() +
    labs(title = "Average of Total Number of Steps per Interval", y = "Mean of Total Number of Steps") +
    facet_wrap(~Day, nrow = 2, ncol=1)
```

<img src="PA1_template_files/figure-html/unnamed-chunk-18-1.png" width="672" />

Activity patterns are somewhat different in the weekdays and in the weekdends. Subjects are more in the weekends.
