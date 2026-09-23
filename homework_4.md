Practical Data Analysis Homework 4
================
Christopher Costello
2026-09-23

- [Question 2](#question-2)

# Question 2

How many rows are available in the Measurements table of the Smith
College Wideband Auditory Immittance database?

``` r
library(RMariaDB)
```

    ## Warning: package 'RMariaDB' was built under R version 4.4.3

``` r
con <- dbConnect(
  MariaDB(), host = "scidb.smith.edu",
  user = "waiuser", password = "smith_waiDB", 
  dbname = "wai"
)
Measurements <- tbl(con, "Measurements")

dbGetQuery(con,
           "
           SELECT SUM(1) AS num_rows
           FROM `Measurements`
           ")
```

    ##   num_rows
    ## 1  5052304

From this query, we find that the number of rows in the Measurements
table is 5,052,304.
