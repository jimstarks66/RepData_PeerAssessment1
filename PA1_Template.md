Load libraries and Set Options
------------------------------

    library(ggplot2)
    library(scales)
    library(data.table)
    theme_set(theme_minimal())

Code for reading in the dataset and processing the data
-------------------------------------------------------

    activity<-read.csv("activity.csv",sep=",",header=TRUE)
    activity$dayofweek<-weekdays(as.Date(activity$date))
    activity$weekend[activity$dayofweek %in% c('Saturday','Sunday')]<-"Weekend"
    activity$weekend[is.na(activity$weekend)]<-"Weekday"

Histogram of the total number of steps taken each day
-----------------------------------------------------

    stepsPerDay<-tapply(activity$steps, activity$date, FUN=sum,na.rm=TRUE)
    hist(stepsPerDay, main="Histogram of Average Steps per Day by Interval (missing values excluded)")

![](PA1_Template_files/figure-markdown_strict/unnamed-chunk-3-1.png)

Mean and median number of steps taken each day
----------------------------------------------

    mean(stepsPerDay)

    ## [1] 9354.23

    median(stepsPerDay)

    ## [1] 10395

Create dataset to aggregate across all days by time interval
------------------------------------------------------------

    avgStepsInterval<-aggregate(activity$steps, list(activity$interval), FUN=mean,na.rm=TRUE)
    colnames(avgStepsInterval)<-c("interval","steps")

    hour<-sprintf("%02d",floor(avgStepsInterval[,1]/100))
    minute<-sprintf("%02d",avgStepsInterval[,1]-floor(avgStepsInterval[,1]/100)*100)
    avgStepsInterval$hmTime<-strptime(paste(hour, minute, sep=":"),"%H:%M")

    avgStepsInterval$hmTime<-as.POSIXct(avgStepsInterval$hmTime)

Time series plot of the average number of steps taken
-----------------------------------------------------

    ggplot(data = avgStepsInterval, aes(x = hmTime, y=steps))+
      geom_line(color = "#00AFBB", size = 2)+
      scale_x_datetime("",labels = date_format("%H:%M"))

![](PA1_Template_files/figure-markdown_strict/unnamed-chunk-6-1.png)

The 5-minute interval that, on average, contains the maximum number of steps
----------------------------------------------------------------------------

    avgStepsInterval[which(avgStepsInterval[,2]==max(avgStepsInterval[,2])),]

    ##     interval    steps              hmTime
    ## 104      835 206.1698 2018-08-05 08:35:00

Code to describe and show a strategy for imputing missing data
--------------------------------------------------------------

### Find the total number of rows with NA's

    sum(is.na(activity$steps))

    ## [1] 2304

### impute missing values by weekday/weekend and time interval

    avg_steps_bytime <-aggregate(activity$steps, by=list(activity$weekend,activity$interval), FUN=mean, na.rm=TRUE)
    colnames(avg_steps_bytime)<-c("weekend", "interval", "avgsteps")

### Check that there are no missing values in steps to be imputed

    sum(is.na(avg_steps_bytime$avgsteps))

    ## [1] 0

### Create dataset with imputed missing values

    activity_nomiss<-activity
    activity_nomiss<-data.table(activity_nomiss)
    avg_steps_bytime<-data.table(avg_steps_bytime)
    keys <- c("weekend","interval")
    setkeyv(activity_nomiss, keys)
    setkeyv(avg_steps_bytime, keys)

    activity_nomiss<-merge(activity_nomiss, avg_steps_bytime)
    activity_nomiss$avgsteps<-NULL

Histogram of the total number of steps taken each day after missing values are imputed
--------------------------------------------------------------------------------------

    stepsPerDay_nomiss<-tapply(activity_nomiss$steps, activity$date, FUN=sum,na.rm=TRUE)
    hist(stepsPerDay_nomiss, main="Histogram of Average Steps per Day by Interval (missing values imputed)")

![](PA1_Template_files/figure-markdown_strict/unnamed-chunk-12-1.png)

    mean(stepsPerDay_nomiss)

    ## [1] 9354.23

    median(stepsPerDay_nomiss)

    ## [1] 7220

Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends
---------------------------------------------------------------------------------------------------------

    avgStepsInterval2<-aggregate(activity$steps, list(activity$interval, activity$weekend), FUN=mean,na.rm=TRUE)
    colnames(avgStepsInterval2)<-c("interval","weekend", "steps")

    hour<-sprintf("%02d",floor(avgStepsInterval2[,1]/100))
    minute<-sprintf("%02d",avgStepsInterval2[,1]-floor(avgStepsInterval2[,1]/100)*100)
    avgStepsInterval2$hmTime<-strptime(paste(hour, minute, sep=":"),"%H:%M")

    avgStepsInterval2$hmTime<-as.POSIXct(avgStepsInterval2$hmTime)

    ggplot(data = avgStepsInterval2, aes(x = hmTime, y=steps))+
      geom_line(color = "#00AFBB", size = 2)+
      scale_x_datetime("",labels = date_format("%H:%M"))+
      facet_wrap(~weekend, ncol=1)

![](PA1_Template_files/figure-markdown_strict/unnamed-chunk-13-1.png)
