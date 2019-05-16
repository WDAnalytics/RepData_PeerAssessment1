---
title: "Week 2 Assignment"
author: "Sarah Weir"
date: "11 May 2019"
output: html_document
---

The variables included in the activity.csv dataset are:

- steps: Number of steps taking in a 5-minute interval (missing values = NA) 
- date: The date on which the measurement was taken in YYYY-MM-DD format
- interval: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a   
total of 17,568 observations in this dataset.


**1 Code for reading in the dataset and/or processing the data**
```{r}
    setwd("~/Coursera/Reproducible research/repdata_data_activity")
    Act<-read.csv("activity.csv", sep=",", header=TRUE)
```


**2 Histogram of the total number of steps taken each day**  
Process data to work out total number of steps per day
```{r}
    #remove intervals were the number of steps is missing
    Act_nonmissing<-Act[!is.na(Act$steps),]

    #summarise number of steps per day
    StepsPerDay<-tapply(Act_nonmissing$steps,Act_nonmissing$date,sum)
    
    #format dates and convert to dataframe
    DateF<-as.POSIXct(names(StepsPerDay),format="%Y-%m-%d")
    df<-data.frame(date=DateF,StepsPerDay)
    
 
```

Produce histogram
```{r}
    hist(df$StepsPerDay, main="Number of Steps per Day", xlab="Number of Steps",ylab="Number of days")
```


**3 Mean and median number of steps taken each day**

```{r}
       #remove missing days
    df_nonmissing<-df[!is.na(df$StepsPerDay),]

    #calc stats    

    Med<-median(df_nonmissing$StepsPerDay)
    Me<-mean(df_nonmissing$StepsPerDay)
```

The Median number of steps per day was `r Med`  
The Mean number of steps per day was `r format(Me, scientific=FALSE)`

**4 What is the average daily activity pattern?**  
Summarise average number of steps in each 5 minute interval
``` {r}
    #find the average number of steps in each interval
    AvePerInt    <- tapply(Act_nonmissing$steps,Act_nonmissing$interval,mean)
    AvePerInt_df <- data.frame(interval=names(AvePerInt),AvePerInt)

    AvePerInt_df$interval<-as.numeric(as.character(AvePerInt_df$interval))
```

Plot the daily activity on a time series graph

```{r}
    plot(AvePerInt_df$interval,AvePerInt_df$AvePerInt, main="Daily activity", type="l",xlab="Interval", ylab="average number of steps")
   
```

**5 The 5-minute interval that, on average, contains the maximum number of steps**  
Find the average number of steps in each interval
then work out which interval contains the maximum number of steps (on average)
``` {r}
    #work out the maximum average number of steps in any interval
    max_steps    <- max(AvePerInt_df$AvePerInt)
    
    #Work out which interval has that number of steps
    max_int      <- subset(AvePerInt_df,AvePerInt_df$AvePerInt==max_steps)
```
The 5 minute interval that, on average, contains the maximum number of steps is `r max_int$interval`

**6 Code to describe and show a strategy for imputing missing data**  
Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)
```{r}
    missing<-Act[is.na(Act$steps),]
    no_missing<-nrow(missing)
```
There are `r no_missing` records with NA for the number of steps

Impute the average for each interval for the missing values  
- Add the average number of steps per interval to the activity table   
- Add a new column (Act_I) that contains either the number of steps or (if the number of steps is NA) the average for that interval

``` {r}
    Act<-merge(x=Act,y=AvePerInt_df ,by="interval")
    Act$steps_I<- ifelse(is.na(Act$steps) == TRUE, Act$AvePerInt, Act$steps)
    
    #create a new data set with the imputed steps
    New_Act<-data.frame(interval=Act$interval, steps=Act$steps_I,date=Act$date)

```


**7 Histogram of the total number of steps taken each day after missing values are imputed**    

```{r}
    #summarise number of steps per day
    StepsPerDay2<-tapply(New_Act$steps,New_Act$date,sum)
    
    #format dates and convert to dataframe
    DateF2<-as.POSIXct(names(StepsPerDay2),format="%Y-%m-%d")
    df2<-data.frame(date=DateF,StepsPerDay2)
    
 
```

Produce histogram
```{r}
    hist(df2$StepsPerDay, main="Number of Steps per Day", xlab="Number of Steps",ylab="Number of days")
```

```{r}
    #calc stats    

    Med2<-median(df2$StepsPerDay)
    Me2<-mean(df2$StepsPerDay)
```

The Median number of steps per day was `r Med` but now the NAs have been imputed it is `r format(Med2, scientific=FALSE)`  
The Mean number of steps per day was `r format(Me, scientific=FALSE)` but now the NAs have been imputed it is `r format(Me2, scientific=FALSE)` 


**8 Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends**  
- Add the day of the week to the table (day)  
- Split the data into 2 sets: weekends and weekday  
```{r}
Act$day <- weekdays(as.Date(Act$date))
 Weekday<-subset(Act,Act$day != "Saturday" & Act$day !="Sunday")
 Weekend<-subset(Act,Act$day == "Saturday" | Act$day =="Sunday")

```

Summarise the data by 5 minute interval
```{r}
    #work of the average number of steps for each interval on a weekday/weekend
    We_ave_stepsperint<- tapply(Weekend$steps_I, Weekend$interval, mean)
    Wd_ave_stepsperint<- tapply(Weekday$steps_I, Weekday$interval, mean)
    
    #create data frames
    df_intWD<-data.frame(interval=names(Wd_ave_stepsperint),avesteps=Wd_ave_stepsperint, daytype="Weekday")
    df_intWE<-data.frame(interval=names(We_ave_stepsperint),avesteps=We_ave_stepsperint, daytype="Weekend")
    
    #combine data frames into 1
    df_intAll<-rbind(df_intWD,df_intWE)
    df_intAll$interval<-as.numeric(as.character(df_intAll$interval))
```

Plot results
```{r}
library(ggplot2)
   g <- ggplot(df_intAll, aes(x=interval, y=avesteps)) + geom_point()
   g<- g+geom_smooth(method = 'loess', formula ='y ~ x')
   g <- g+facet_grid(daytype ~ .)
   g <- g+ylab("Average number of steps")
   g <- g+ scale_x_continuous(breaks=c(0,300,600,900,1200,1500,1800,2100,2300), labels=c(0,300,600,900,1200,1500,1800,2100,2300))
   g <- g+ theme(axis.text.x = element_text(angle = 90))
   g
```

**9 All of the R code needed to reproduce the results (numbers, plots, etc.) in the report**  
