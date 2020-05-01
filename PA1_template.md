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

    dates <- as.factor(activity$date)
    startDate <- as.Date("2012-10-01")
    total <- activity %>%
        group_by(date) %>%
        summarize(t = sum(steps, na.rm = TRUE)) %>%
        mutate(days = as.numeric(date - startDate) + 1)
    totalCount <- vector()
    for (i in 1:nrow(total)) {
        totalCount <- c(totalCount, rep(total[[i, "days"]], total[[i, "t"]]))
    }

Make a histogram of the total number of steps taken each day

    hist(totalCount, breaks = nrow(total), xlab = "Day", main = "Everyday Total Steps")

![](/Users/chuangchuangzhang/Documents/Freelancer/publab/R/RepData/PA1_template_files/figure-markdown_strict/unnamed-chunk-2-1.png)

Calculate and report the mean and median of the total number of steps
taken per day

    total1 <- activity %>%
        subset(!is.na(steps)) %>%
        group_by(date) %>%
        summarize(mean = mean(steps), median = median(steps))
    head(total1)

    # A tibble: 6 x 3
      date         mean median
      <date>      <dbl>  <dbl>
    1 2012-10-02  0.438      0
    2 2012-10-03 39.4        0
    3 2012-10-04 42.1        0
    4 2012-10-05 46.2        0
    5 2012-10-06 53.5        0
    6 2012-10-07 38.2        0

What is the average daily activity pattern?
-------------------------------------------

Make a time series plot of the 5-minute interval (x-axis) and the
average number of steps taken, averaged across all days (y-axis)

    activity1 <- activity %>%
        mutate(datetime = `minute<-`(date, interval)) %>%
        filter(!is.na(steps))
    with(activity1, plot(datetime, steps, type = "l", 
                         xlab = "Datetime", ylab = "Steps",
                         main = "Every Five Minutes Steps"))
    abline(v = activity1[[max(activity1$steps, na.rm = TRUE), 'datetime']], 
           col = "red", lwd = 4)

![](/Users/chuangchuangzhang/Documents/Freelancer/publab/R/RepData/PA1_template_files/figure-markdown_strict/step3-1.png)

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
    activityMissing$steps <- apply(as.matrix(activityMissing), 1, function(x) {
        if (is.na(x[1])) {
            dayMeans[dayMeans$date == x[2], 'm'][[1, 'm']]
        } else {
            as.numeric(x[1])
        }
    })

Make a histogram of the total number of steps taken each day

    total <- activityMissing %>%
        group_by(date) %>%
        summarize(t = sum(steps, na.rm = TRUE)) %>%
        mutate(days = as.numeric(date - startDate) + 1)
    totalCount <- vector()
    for (i in 1:nrow(total)) {
        totalCount <- c(totalCount, rep(total[[i, "days"]], total[[i, "t"]]))
    }
    hist(totalCount, breaks = nrow(total), xlab = "Day", main = "Filled Missing Data, Everyday Total Steps")

![](/Users/chuangchuangzhang/Documents/Freelancer/publab/R/RepData/PA1_template_files/figure-markdown_strict/unnamed-chunk-5-1.png)

Calculate and report the mean and median total number of steps taken per
day

    total1 <- activityMissing %>%
        group_by(date) %>%
        summarize(mean = mean(steps), median = median(steps))
    head(total1)

    # A tibble: 6 x 3
      date         mean median
      <date>      <dbl>  <dbl>
    1 2012-10-01  0          0
    2 2012-10-02  0.438      0
    3 2012-10-03 39.4        0
    4 2012-10-04 42.1        0
    5 2012-10-05 46.2        0
    6 2012-10-06 53.5        0

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

Create a new factor variable in the dataset with two levels – “weekday”
and “weekend” indicating whether a given date is a weekday or weekend
day

    allWeekdays <- c(2:6)
    activityMissing$w <- factor(wday(activityMissing$date) %in% allWeekdays,
                                level = c(FALSE, TRUE), labels = c("weekend", "weekday"))
    activityMissing <- activityMissing %>%
        group_by(w) %>%
        mutate(x = 1:n())

Make a panel plot containing a time series plot

    ggplot(activityMissing, aes(x, steps)) +
        geom_line(color = "blue") +
        facet_wrap(facets = vars(w), nrow = 2) +
        xlab("Interval") +
        ylab("Steps") +
        theme(strip.background = element_rect(fill = "orange"), 
              panel.spacing = unit(0, "lines"))

![](/Users/chuangchuangzhang/Documents/Freelancer/publab/R/RepData/PA1_template_files/figure-markdown_strict/unnamed-chunk-7-1.png)
