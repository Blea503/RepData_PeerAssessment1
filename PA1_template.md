---
title: "PA1_template.md"
author: "Blea503"
date: "September 26, 2017"
output: html_document
---

```{r setup, include=TRUE}
knitr::opts_chunk$set(echo = TRUE)
library("plyr", lib.loc="~/R/win-library/3.4")
library("dplyr", lib.loc="~/R/win-library/3.4")
library("magrittr", lib.loc="~/R/win-library/3.4")
library("lubridate", lib.loc="~/R/win-library/3.4")
library("knitr", lib.loc="~/R/win-library/3.4")

dat <- read.csv("activity.csv", header = TRUE, sep = ",", colClasses = c("numeric", "Date", "numeric"),na.strings = "?")
```

```{r, plot1, echo=TRUE}
png(filename = "plot1.png")
tot_steps <- ddply(dat, .(date), summarise, tot_steps = sum(steps, na.rm = TRUE))
summary(tot_steps)
hist(tot_steps$tot_steps, main = "Total steps per day", xlab = "Mean total number of steps per day")
abline(v = mean(tot_steps$tot_steps, na.rm = TRUE), col = "red", lwd = 4)
abline(v = median(tot_steps$tot_steps, na.rm = TRUE), col = "blue", lwd = 4)
text(20000, 15, paste(c("mean ="), round(mean(tot_steps$tot_steps, na.rm = TRUE), 1)), col="red")
text(20000, 10, paste(c("median ="), round(median(tot_steps$tot_steps, na.rm = TRUE), 1)), col="blue")
```

```{r, plot2, echo=TRUE}
png(filename = "plot2.png")
dpattern <- ddply(dat, .(interval), summarise, meansteps = mean(steps, na.rm = TRUE))
with(dpattern, plot(interval, meansteps, type="l"))
max_step_interval <- dpattern %>% arrange(desc(meansteps))
max_int <- max_step_interval[1,1]
text(1500, 150, paste(c("5-minute interval maximum steps ="), max_int))
```
Replace NAs with the median steps per interval
```{r, plot3, echo=TRUE}
png(filename = "plot3.png")
number_Nas <- sum(is.na(dat$steps))
missing <- ddply(dat, .(interval), transform, steps=ifelse(is.na(steps), median(steps, na.rm=TRUE), steps))
tot_steps2 <- ddply(missing, .(date), summarise, tot_steps = sum(steps, na.rm = TRUE))
summary(tot_steps2)
hist(tot_steps2$tot_steps, main = "Total steps per day", xlab = "Mean total number of steps per day")
abline(v = mean(tot_steps2$tot_steps, na.rm = TRUE), col = "red", lwd = 4)
abline(v = median(tot_steps2$tot_steps, na.rm = TRUE), col = "blue", lwd = 4)
text(20000, 15, paste(c("mean ="), round(mean(tot_steps2$tot_steps, na.rm = TRUE), 1)), col="red")
text(20000, 10, paste(c("median ="), round(median(tot_steps2$tot_steps, na.rm = TRUE), 1)), col="blue")
```
Are there differences in activity patterns between weekdays and weekends?
```{r, plot4, echo=TRUE}
library("ggplot2", lib.loc="~/R/win-library/3.4")
dat$weekday <- weekdays(dat$date)
dat$day <- ifelse(dat$weekday =="Saturday" | dat$weekday=="Sunday", "weekend", "weekday")
df <- aggregate(steps ~ interval + day, data=dat, mean)
g <- ggplot(data = df, aes(interval, steps))+ geom_line() + facet_grid(day ~ .)
ggsave(filename = "plot4.png")
```

