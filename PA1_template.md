This is a R Markdown document.

#### Load all the library first

    library(dplyr, warn.conflicts = FALSE)
    library(lubridate, warn.conflicts = FALSE)
    library(ggplot2, warn.conflicts = FALSE)

Loading and preprocessing the data
----------------------------------

1.  Load the data
2.  Process/transform the data (if necessary) into a format suitable for
    your analysis

<!-- -->

    activity <- read.csv("./activity.csv", header = TRUE, sep = ",")
    activity <- tbl_df(activity)
    activity$date <- as.Date(as.character(activity$date), "%Y-%m-%d")

What is mean total number of steps taken per day?
-------------------------------------------------

    total <- tapply(activity$steps, activity$date, sum, na.rm = TRUE)

Make a histogram of the total number of steps taken each day

    hist(total, breaks = 30, xlab = "Steps", main = "Histogram of Everyday Total Steps")

![](/Users/chuangchuangzhang/Documents/Freelancer/publab/R/RepData/PA1_template_files/figure-markdown_strict/unnamed-chunk-2-1.png)

Calculate and report the mean and median of the total number of steps
taken per day

    summary(total)

       Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
          0    6778   10395    9354   12811   21194 

What is the average daily activity pattern?
-------------------------------------------

Make a time series plot of the 5-minute interval (x-axis) and the
average number of steps taken, averaged across all days (y-axis)

    activityAvg <- activity %>% 
      group_by(interval) %>%
      summarize(steps = mean(steps, na.rm = TRUE))
    with(activityAvg, plot(interval, steps, type = "l", 
                         xlab = "Interval", ylab = "Steps",
                         main = "Interval Steps"))
    abline(v = activityAvg[[which.max(activityAvg$steps), 'interval']], 
           col = "red", lwd = 4)

![](/Users/chuangchuangzhang/Documents/Freelancer/publab/R/RepData/PA1_template_files/figure-markdown_strict/step3-1.png)

    activityAvg[[which.max(activityAvg$steps), 'interval']]

    [1] 835

Imputing missing values
-----------------------

Calculate and report the total number of missing values in the dataset

    summary(is.na(activity$steps))

       Mode   FALSE    TRUE 
    logical   15264    2304 

Filled the missing data

    dayMeans <- activity %>%
        group_by(date) %>%
        summarize(m = mean(steps, na.rm = TRUE))
    dayMeans[is.nan(dayMeans$m), 'm'] <- 0
    activityMissing <- activity
    activityMissing$steps <- apply(as.matrix(activity), 1, function(x) {
        if (is.na(x[1])) {
            activityAvg[activityAvg$interval == as.integer(x[3]), 'steps'][[1, 'steps']]
        } else {
            as.integer(x[1])
        }
    })

Make a histogram of the total number of steps taken each day

    total <- tapply(activityMissing$steps, activityMissing$date, sum)

Make a histogram of the total number of steps taken each day

    hist(total, breaks = 30, xlab = "Steps", main = "Filled Missing Data, Histogram of Everyday Total Steps")

![](/Users/chuangchuangzhang/Documents/Freelancer/publab/R/RepData/PA1_template_files/figure-markdown_strict/unnamed-chunk-6-1.png)

Calculate and report the mean and median of the total number of steps
taken per day after filling missing data

    summary(total)

       Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
         41    9819   10766   10766   12811   21194 

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

Create a new factor variable in the dataset with two levels – “weekday”
and “weekend” indicating whether a given date is a weekday or weekend
day

    activityMissing$w <- factor(wday(activityMissing$date) %in% allWeekdays,
                                level = c(FALSE, TRUE), labels = c("weekend", "weekday"))
    activityMissingWeek <- activityMissing %>%
        group_by(w, interval) %>%
        summarize(steps = mean(steps))

Make a panel plot containing a time series plot

    ggplot(activityMissingWeek, aes(interval, steps)) +
        geom_line(color = "blue") +
        facet_wrap(facets = vars(w), nrow = 2) +
        xlab("Interval") +
        ylab("Steps") +
        theme(strip.background = element_rect(fill = "orange"), 
              panel.spacing = unit(0, "lines"))

![](/Users/chuangchuangzhang/Documents/Freelancer/publab/R/RepData/PA1_template_files/figure-markdown_strict/unnamed-chunk-8-1.png)
