------------------------------------------------------------------------

**Preliminary steps**

<br> First of all, read in the raw data:

    dataset <- read.csv("./repdata_data_activity/activity.csv")

<br> Then check some basic information about the dataset:

    names(dataset)

    ## [1] "steps"    "date"     "interval"

    print(paste("ncol(dataset) =", ncol(dataset),"   nrow(dataset) =", nrow(dataset), "   length(unique(dataset$date)) =", length(unique(dataset$date))))

    ## [1] "ncol(dataset) = 3    nrow(dataset) = 17568    length(unique(dataset$date)) = 61"

So there are 3 columns/variables, 17568 rows/observations, but only 61
unique dates.

<br> Then I load some necessary libraries:

    library(ggplot2)

    ## Warning: package 'ggplot2' was built under R version 3.4.1

    library(data.table)

<br>
----

**Plot 1: A histogram of the total number of steps taken each day**

<br> I start by creating a data table object from the original data set,
stored as 'DT':

    DT <- data.table(dataset)

<br> I then create a second, smaller data table from DT called 'DT2' by
calculating the sum of all the steps recorded for a given date:

    DT2 <- DT[ , sum(steps), by = date]
    print(paste("nrow(DT2) =", nrow(DT2)))

    ## [1] "nrow(DT2) = 61"

I can see it has the expected number of rows (61).

<br> I can then calculate the mean and median of these daily totals,
ignoring any missing values:

    mean_total_steps_per_day <- mean(DT2$V1, na.rm = TRUE)
    print(paste("mean_total_steps_per_day =", mean_total_steps_per_day))

    ## [1] "mean_total_steps_per_day = 10766.1886792453"

    median_total_steps_per_day <- median(DT2$V1, na.rm = TRUE)
    print(paste("median_total_steps_per_day =", median_total_steps_per_day))

    ## [1] "median_total_steps_per_day = 10765"

<br> Finally I can generate the histogram, including a vertical line to
indicate the mean value (the median is so close to the mean that it's
not worth plotting a second vertical line at the median value):

    hist(x = DT2$V1, col = "red", breaks = 30, xlab = "Sum of steps in a day",
         main = "Total number of steps taken each day")
    abline(v = mean_total_steps_per_day, col = "black", lwd = 2)

![](Week_2_assignment_files/figure-markdown_strict/unnamed-chunk-7-1.png)

<br>
----

**Plot 2: A time series plot of the average number of steps taken**

<br> I start by creating a data table object from the original data set,
stored as 'DT':

    DT <- data.table(dataset)

<br> I then create a second, smaller data table from DT called 'DT2', by
calculating the mean of all the steps recorded for a given interval,
ignoring missing values:

    DT2 <- DT[ , mean(steps, na.rm = TRUE), by = interval]

<br> Next I find out what the highest average number of steps was,
storing the value in the variable 'max\_steps':

    max_steps <- max(DT2$V1)
    print(paste("max_steps =", max_steps))

    ## [1] "max_steps = 206.169811320755"

<br> And then I use max\_steps to find the corresponding interval,
storing the value in the variable 'max\_interval':

    max_interval <- NULL
    for(i in 1:nrow(DT2)){
      if(DT2$V1[i] == max_steps){
        max_interval <- DT2$interval[i]
        break
        }
    }
    print(paste("max_interval =", max_interval))

    ## [1] "max_interval = 835"

So the interval that has the highest number of steps on average is
interval 835.

<br> Finally I generate a line plot of the mean number of steps vs. the
interval, with a vertical line indicating the interval with the highest
average number of steps, as follows:

    plot(x = DT2$interval, y = DT2$V1, type = "l", col = "blue",
         xlab = "Interval", ylab = "Mean number of steps",
         main = "Time series plot of the average number of steps taken")
    abline(v = max_interval, col = "black", lwd = 2) 

![](Week_2_assignment_files/figure-markdown_strict/unnamed-chunk-12-1.png)

<br>
----

**Plot 3: A histogram of the total number of steps taken each day after
missing values are imputed**

<br> First I pick out the rows in the raw data that contain NA values
for the number of steps and store them under 'rows\_with\_NA':

    rows_with_NA <- subset(dataset, is.na(steps))
    print(paste("nrow(rows_with_NA) =", nrow(rows_with_NA)))

    ## [1] "nrow(rows_with_NA) = 2304"

<br> Then I pick out the rows in the raw data that do not contain NA
values, storing them under 'rows\_without\_NA':

    rows_without_NA <- subset(dataset, !is.na(steps))
    print(paste("nrow(rows_without_NA) =", nrow(rows_without_NA)))

    ## [1] "nrow(rows_without_NA) = 15264"

<br> Next I store the raw data in a data table called 'DT', before
creating a smaller table 'DT2' that contains the interval and the mean
number of steps in each interval, and then set the column names to
something appropriate:

    DT <- data.table(dataset)
    DT2 <- DT[ , mean(steps, na.rm = TRUE), by = interval]
    colnames(DT2) <- c("interval", "mean steps")
    print(paste("nrow(DT2) =", nrow(DT2)))

    ## [1] "nrow(DT2) = 288"

<br> Then I merge the table containing NA step values with the table
containing mean number of steps in each interval, by the 'interval'
column, so we end up with a table that has 4 columns: interval, steps,
date and mean steps:

    a <- merge(rows_with_NA, DT2, by = "interval") 
    print(paste("nrow(a) =", nrow(a)))

    ## [1] "nrow(a) = 2304"

<br> Then I replicate table a, assigning it the name 'b', and replace
the steps column with the mean steps column, before re-naming this
column:

    b <- a
    b$steps <- b$mean_steps
    colnames(b) <- c("interval", "date", "steps")

So every interval where there was a steps = NA value now has NA replaced
by mean value for that interval.

<br> Then I replicate table b, assigning it the name 'c', but change the
order of the columns to match original dataset:

    c <- b
    colnames(c) <- c("steps", "date", "interval")
    c$steps     <- b$steps
    c$date      <- b$date
    c$interval  <- b$interval

<br> Then I combine the rows that have had NA values replaced (i.e.
table c) with the rows without NA values, and give this object the name
'imputed\_dataset', as well as checking that I have the correct number
of rows afterwards:

    imputed_dataset <- rbind(c, rows_without_NA)
    print(paste("nrow(imputed_dataset) =", nrow(imputed_dataset)))

    ## [1] "nrow(imputed_dataset) = 17568"

<br> Just to check everything is as it should be, I search for any rows
in imputed\_dataset that contain NA values for the number of steps:

    rows_with_NA__imputed <- subset(imputed_dataset, is.na(steps))
    print(paste("nrow(rows_with_NA__imputed) =", nrow(rows_with_NA__imputed)))

    ## [1] "nrow(rows_with_NA__imputed) = 0"

As expected, no rows containing NA exist now.

<br> Using this dataset that has now had missing values imputed, I
create a new table called 'DT\_imputed\_dataset', and from
DT\_imputed\_dataset I create a smaller table called 'DT3' that contains
the date and the sum of the steps taken on each date:

    DT_imputed_dataset <- data.table(imputed_dataset)
    DT3 <- DT_imputed_dataset[ , sum(steps), by = date]
    print(paste("nrow(DT3) =", nrow(DT3)))

    ## [1] "nrow(DT3) = 61"

<br> Now I calculate the mean and median value of daily steps taken:

    mean_total_steps_per_day <- mean(DT3$V1, na.rm = FALSE)
    print(paste("mean_total_steps_per_day =", mean_total_steps_per_day))

    ## [1] "mean_total_steps_per_day = 10766.1886792453"

    median_total_steps_per_day <- median(DT3$V1, na.rm = FALSE)
    print(paste("median_total_steps_per_day =", median_total_steps_per_day))

    ## [1] "median_total_steps_per_day = 10766.1886792453"

So I find that once the missing values have been imputed, the mean value
is no different, but the median value changes slightly, from 10765 to
10766.189.

<br> Finally I plot a histogram of the number of daily steps, with a
vertical line at the mean value:

    hist(x = DT3$V1, col = "green", breaks = 30, xlab = "Sum of steps in a day", main = "Total number of steps taken each day")
    abline(v = mean_total_steps_per_day, col = "black", lwd = 2)

![](Week_2_assignment_files/figure-markdown_strict/unnamed-chunk-23-1.png)

<br>

------------------------------------------------------------------------

**Plot 4: A panel plot comparing the average number of steps taken per
5-minute interval across weekdays and weekends**

<br> I start by assigning the raw data to 'DF', then convert the values
in the 'date' column to objects of the type 'Date':

    DF <- dataset
    DF$date <- as.Date(DF$date)

<br> Then I create a new column in DF called 'day' that is the day of
the week, calculated by the function 'weekdays()' using the entries in
the 'date' column:

    DF$day <- weekdays(DF$date)

<br> Then I loop through the day column, replacing the specific days
with either "weekend" or "weekday":

    for(i in 1:nrow(DF)){
      if(DF$day[i] == "Saturday" | DF$day[i] == "Sunday") DF$day[i] <- "weekend"
      else DF$day[i] <- "weekday"
    }

<br> Then I take two subsets of DF, one for the weekend ('DF\_weekend')
and one for the weekdays ('DF\_weekday').

    DF_weekend <- subset(DF, day == "weekend")
    DF_weekday <- subset(DF, day == "weekday")

<br> Then I create a data table called 'DT\_\_weekend' out of the data
frame 'DF\_weekend'. Using DT\_\_weekend I then create
'DT\_\_weekend\_2' that shows the interval and the average number of
steps in a given interval for the weekend days, and also set appropriate
column names:

    DT__weekend <- data.table(DF_weekend)
    DT__weekend_2 <- DT__weekend[ , mean(steps, na.rm = TRUE), by = interval]
    DT__weekend_2$day <- c(rep("weekend", nrow(DT__weekend_2)))
    colnames(DT__weekend_2) <- c("interval", "mean.steps", "day")
    print(paste("nrow(DT__weekend_2) =", nrow(DT__weekend_2)))

    ## [1] "nrow(DT__weekend_2) = 288"

It has the expected number of rows (288).

<br> Then I create a data table called 'DT\_\_weekday' out of the data
frame 'DF\_weekday'. Using DT\_\_weekday I then create
'DT\_\_weekday\_2' that shows the interval and the average number of
steps in a given interval for the weekdays, and also set appropriate
column names:

    DT__weekday <- data.table(DF_weekday)
    DT__weekday_2 <- DT__weekday[ , mean(steps, na.rm = TRUE), by = interval]
    DT__weekday_2$day <- c(rep("weekday", nrow(DT__weekday_2)))
    colnames(DT__weekday_2) <- c("interval", "mean.steps", "day")
    print(paste("nrow(DT__weekday_2) =", nrow(DT__weekday_2)))

    ## [1] "nrow(DT__weekday_2) = 288"

It has the expected number of rows (288).

<br> I then combine these two data tables, DT\_\_weekend\_2 and
DT\_\_weekday\_2, and make sure the 'interval' and 'mean.steps' columns
are the right type of object (numeric):

    DT__combined <- rbind(DT__weekend_2, DT__weekday_2)
    DT__combined$interval   <- as.numeric(as.character(DT__combined$interval))
    DT__combined$mean.steps <- as.numeric(as.character(DT__combined$mean.steps))

I thus have all the average steps for each interval for both weekends
and weekdays in one table, distinguishable by the 'day' column. This
distinction will be used to generate two comparable plots from one
ggplot() command.

<br> Finally I make the plot, noting the call to facet\_grid() that
causes two plots to be generated, one for each category of day:

    g <- ggplot(DT__combined, aes(x = interval, y = mean.steps, col = "red"))
    g <- g + geom_line()
    g <- g + facet_grid(day ~ .)
    g <- g + xlab("Interval") + ylab("Mean steps")
    g <- g + ggtitle("Comparison of mean steps taken per 5-minute interval for weekdays vs weekends")
    g <- g + theme(legend.position="none")
    print(g)

![](Week_2_assignment_files/figure-markdown_strict/unnamed-chunk-31-1.png)
