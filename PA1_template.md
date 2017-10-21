Loading and preprocessing the data
----------------------------------

Code needed to load the data (i.e. 𝚛𝚎𝚊𝚍.𝚌𝚜𝚟())

    data <- read.csv("activity.csv")

What is mean total number of steps taken per day?
-------------------------------------------------

For this part of the assignment, we ignore the missing values in the
dataset.

Calculate the total number of steps taken per day and displayed in a
histogram

    library(ggplot2)
    total.steps <- tapply(data$steps, data$date, FUN=sum, na.rm=TRUE)
    qplot(total.steps, binwidth=1000, xlab="Total Number of Steps Taken Each Day")

![](PA1_template_files/figure-markdown_strict/total-1.png)

Calculate and report the mean and median of the total number of steps
taken per day

    mean(total.steps, na.rm=TRUE)

    ## [1] 9354.23

    median(total.steps, na.rm=TRUE)

    ## [1] 10395

What is the average daily activity pattern?
-------------------------------------------

Make a time series plot (i.e. 𝚝𝚢𝚙𝚎 = "𝚕") of the 5-minute interval
(x-axis) and the average number of steps taken, averaged across all days
(y-axis)

    library(ggplot2)
    averages <- aggregate(x=list(steps=data$steps), by=list(interval=data$interval),
                          FUN=mean, na.rm=TRUE)
    ggplot(data=averages, aes(x=interval, y=steps)) +
        geom_line() +
        xlab("5-minute interval") +
        ylab("average number of steps taken")

![](PA1_template_files/figure-markdown_strict/avgStepsByDay-1.png)

On average across all the days in the dataset, the 5-minute interval
contains the maximum number of steps is below:

    averages[which.max(averages$steps),]

    ##     interval    steps
    ## 104      835 206.1698

Imputing missing values
-----------------------

Note that there are a number of days/intervals where there are missing
values (coded as 𝙽𝙰). The presence of missing days may introduce bias
into some calculations or summaries of the data.

Calculate and report the total number of missing values in the dataset
(i.e. the total number of rows with 𝙽𝙰s)

    sum(is.na(data))

    ## [1] 2304

The strategy we use here are the mean for that 5-minute interval.

Create a new dataset that is equal to the original dataset but with the
missing data filled in.

    data.merged <- merge(data,averages, by="interval")
    data.merged$steps.x[is.na(data.merged$steps.x)] <- data.merged$steps.y[is.na(data.merged$steps.x)]

Make a histogram of the total number of steps taken each day and
Calculate and report the mean and median total number of steps taken per
day.

    data.merged <- aggregate(steps.x~interval,data.merged,sum)
    hist(data.merged$steps.x, xlab="Total Steps by Day", ylab="Frequency [Days]",main="Histogram: Number of Daily Steps")

![](PA1_template_files/figure-markdown_strict/Historgram-1.png) The
values differ from the estimates from the first part of the assignment.
We recaculate the mean and median total number of steps taken per day.

    mean2 <- mean(data.merged$steps, na.rm=TRUE)
    median2 <- median(data.merged$steps, na.rm=TRUE)

The impact of imputing missing data on the estimates of the total daily
number of steps are as followed: 1. The histogram now has many more in
the bins values less than 2000 steps per day - the distribution of the
new histogram has changed. 2. The mean and median went from more than
10000 to less than 3000. Replacing the NA values for the 5-mintues
average steps changed resulting analysis.

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

For this part, we use the the 𝚠𝚎𝚎𝚔𝚍𝚊𝚢𝚜() function.

Create a new factor variable in the dataset with two levels – “weekday”
and “weekend” indicating whether a given date is a weekday or weekend
day.

    daytype <- function(date) {
        if (weekdays(as.Date(date)) %in% c("Saturday", "Sunday")) {
            "Weekend"
        } else {
            "Weekday"
        }
    }
    data$daytype <- as.factor(sapply(data$date, daytype))
    str(data)

    ## 'data.frame':    17568 obs. of  4 variables:
    ##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
    ##  $ daytype : Factor w/ 2 levels "Weekday","Weekend": 1 1 1 1 1 1 1 1 1 1 ...

Make a panel plot containing a time series plot (i.e. 𝚝𝚢𝚙𝚎 = "𝚕") of the
5-minute interval (x-axis) and the average number of steps taken,
averaged across all weekday days or weekend days (y-axis). See the
README file in the GitHub repository to see an example of what this plot
should look like using simulated data.

    par(mfrow=c(2,1))
    for (type in c("Weekend", "Weekday")) {
        steps.type <- aggregate(steps ~ interval,
                                data=data,
                                subset=data$daytype==type,
                                FUN=mean)
        plot(steps.type, type="l", main=type)
    }

![](PA1_template_files/figure-markdown_strict/week_diff-1.png)
