Practical Data Analysis Homework 4
================
Christopher Costello
2026-09-23

- [Question 2](#question-2)
- [Question 3](#question-3)
- [Question 4](#question-4)
- [Question 4.1](#question-41)

# Question 2

How many rows are available in the Measurements table of the Smith
College Wideband Auditory Immittance database?

``` r
library(tidyverse)
library(RMariaDB)
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

# Question 3

Identify what years of data are available in the flights table of the
airlines database.

``` r
library(tidyverse)
library(mdsr)
library(RMariaDB)
con <- dbConnect_scidb("airlines")

dbListTables(con)
```

    ## [1] "airports"        "planes"          "carriers"        "flights_summary"
    ## [5] "flights"

``` r
tbl(con, "flights")
```

    ## # Source:   table<`flights`> [?? x 21]
    ## # Database: mysql  [mdsr_public@mdsr.crcbo51tmesf.us-east-2.rds.amazonaws.com:3306/airlines]
    ##     year month   day dep_time sched_dep_time dep_delay arr_time sched_arr_time
    ##    <int> <int> <int>    <int>          <int>     <int>    <int>          <int>
    ##  1  2013    10     1        2             10        -8      453            505
    ##  2  2013    10     1        4           2359         5      730            729
    ##  3  2013    10     1       11             15        -4      528            530
    ##  4  2013    10     1       14           2355        19      544            540
    ##  5  2013    10     1       16             17        -1      515            525
    ##  6  2013    10     1       22             20         2      552            554
    ##  7  2013    10     1       29             35        -6      808            816
    ##  8  2013    10     1       29             35        -6      449            458
    ##  9  2013    10     1       31             30         1      519            538
    ## 10  2013    10     1       32             33        -1      557            606
    ## # ℹ more rows
    ## # ℹ 13 more variables: arr_delay <int>, carrier <chr>, tailnum <chr>,
    ## #   flight <int>, origin <chr>, dest <chr>, air_time <int>, distance <int>,
    ## #   cancelled <int>, diverted <int>, hour <int>, minute <int>, time_hour <dttm>

``` r
dbGetQuery(con,
           "
           SELECT DISTINCT year
           FROM flights
           ")
```

    ##   year
    ## 1 2013
    ## 2 2014
    ## 3 2015

# Question 4

Use the dbConnect_scidb function to connect to the airlines database to
answer the following problem.

The `airlines` database is already connected from Question 3.

# Question 4.1

How many domestic flights flew into Dallas-Fort Worth (DFW) on May 14,
2015?

``` r
dbGetQuery(con,
          "
          SELECT SUM(1) AS number_of_flights
          FROM flights
          WHERE
            dest = 'DFW' AND
            year = 2015 AND
            month = 5 AND
            day = 14
          ")
```

    ##   number_of_flights
    ## 1               737
