---
title: "PA1_template"
author: "EGB"
date: "April 10, 2016"
output: html_document
---

Code for reading in the dataset and/or processing the data:

```{r, echo = TRUE}
activity <- read.csv("activity.csv", stringsAsFactors = FALSE)
library(lubridate)
activity$date <- ymd(activity$date)
```

Histogram of the total number of steps taken each day:

```{r, echo = TRUE}
spday <- tapply(activity$steps, activity$date, sum)
hist(spday, main = "Histogram of Steps a Day", xlab = "Total of Steps a Day")
```

Mean and median number of steps taken each day:

```{r, echo = TRUE}
mean(activity$steps, na.rm = TRUE)
median(activity$steps, na.rm = TRUE)
```

Time series plot of the average number of steps taken:

```{r, echo = TRUE}
#creating subset with no NA's
st <- activity[complete.cases(activity),]
#getting average steps for all days per interval
y <- tapply(st$steps, st$interval, mean)
#time-series plot
plot(unique(st$interval), y, type = "l", main = "Time Series of Steps per Interval", xlab = "Interval", ylab = "Total Steps")

#The 5-minute interval that, on average, contains the maximum number of steps:

#Find interval with max steps
y[y==max(y)]
#Designate with abline and text
abline(v = 835, lwd = 2, col = "red")
text(835,max(y), "Max average steps of about 206 at interval 835")

#Code to describe and show a strategy for imputing missing data:

#Number of NA rows - is about 13-14%
str(activity[is.na(activity),])
#Replacing NA's with respective interval average while creating another dataset
#Making table with averages for interval
z <- activity
yy <- unique(z$interval)
avez <- as.data.frame(cbind(yy,y))
#loop to replace NA's
for (i in length(z$interval)){
  if (is.na(z$steps[i]) == TRUE){
    z$steps[i] <- avez[avez$yy == z$interval[i],2]
  }
 }
```

Histogram of the total number of steps taken each day after missing values are imputed:

```{r, echo = TRUE}
z1 <- tapply(z$steps, z$date, sum)
hist(z1)
```

Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends:

```{r, echo = TRUE}
#Make dataset with id for weekday or weekend
WK <- NULL
for (i in 1:17568){WK[i]<-wday(z$date[i])}
WK <- cbind(z,WK)
#Get subset with only weekdays
WD <- WK[WK$WK != 1 & WK$WK != 7,]
WD$WK <- "Weekday"
#Get subset with only weekends
WE <- WK[WK$WK == 1 | WK$WK == 7,]
WE$WK <- "Weekend"
#Calculate averages per interval for each subset
WDA <- tapply(WD$steps, WD$interval, mean)
WEA <- tapply(WE$steps, WE$interval, mean)
#Combine subsets to panel plot
WK <- rbind(WD,WE)
#Panel plot
library(ggplot2)
qplot(interval, steps, data = WK, facets = WK ~ ., geom = c("point", "smooth"), main = "Average Steps per Interval based on Weekday or Weekend")
```
