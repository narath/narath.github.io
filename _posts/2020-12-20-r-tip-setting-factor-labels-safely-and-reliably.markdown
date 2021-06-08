---
layout: post
title: "R tip - Setting Factor Labels Safely and Reliably"
date: "2020-12-20 14:14:51 -0500"
---

I was working on a report that uses factor labels to show the days of week that users where pressing buttons. Early on users did not always have data for all days. Overcoming this in R is easy - just specify all the levels you expect when you create your factor. When using days of the week as the factor for dates, the following works:

```r
dow_levels = c("1", "2", "3", "4", "5", "6", "7")
dow_name = c("Sun", "Mon", "Tues", "Wed", "Thurs", "Fri", "Sat")
dates <- as.Date(c("2020-12-20", "2020-12-19"))
f <- factor(wday(dates))
factor(wday(dates), labels = dow_name, levels = dow_levels)
```

If you don't specify levels, then you would get an error because your factor will only have 2 levels (1 for Sunday and 7 for Saturday).



