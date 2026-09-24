Practical Data Analysis Homework 4
================
Christopher Costello
2026-09-23

- [Question 2](#question-2)
- [Question 3](#question-3)
- [Question 4](#question-4)
  - [Question 4.1](#question-41)
  - [Question 4.2](#question-42)
  - [Question 4.3](#question-43)
  - [Question 4.4](#question-44)
  - [Question 4.5](#question-45)

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

## Question 4.1

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

## Question 4.2

Of all the destinations from Chicago O’Hare (ORD), which were the most
common in 2015?

``` r
dbGetQuery(con,
          "
          SELECT
            dest,
            SUM(1) AS number_of_flights
          FROM
            flights
          WHERE
            origin = 'ORD' AND
            year = 2015
          GROUP BY
            dest
          ORDER BY
            number_of_flights DESC
          LIMIT 10
          ")
```

    ##    dest number_of_flights
    ## 1   LGA             10492
    ## 2   LAX              8720
    ## 3   DFW              8384
    ## 4   SFO              8156
    ## 5   BOS              7240
    ## 6   ATL              7104
    ## 7   MSP              6955
    ## 8   DCA              6495
    ## 9   DEN              6120
    ## 10  MKE              5274

## Question 4.3

Which airport had the highest average arrival delay time in 2015?

``` r
dbGetQuery(con,
          "
          SELECT
            dest,
            avg(arr_delay) AS average_arrival_delay
          FROM flights
          WHERE
            year = 2015
          GROUP BY
            dest
          ORDER BY
            average_arrival_delay DESC
          LIMIT 1
          ")
```

    ##   dest average_arrival_delay
    ## 1  STC                21.622

## Question 4.4

How many domestic flights came into or flew out of Bradley Airport (BDL)
in 2015?

``` r
dbGetQuery(con,
          "
          SELECT
            SUM(1) AS number_of_flights
          FROM
            flights
          WHERE
            year = 2015
            AND (
              origin = 'BDL' OR
              dest = 'BDL'
            )
           ")
```

    ##   number_of_flights
    ## 1             41025

## Question 4.5

List the airline and flight number for all flights between LAX and JFK
on September 26th, 2015.

``` r
# For reference
tbl(con, "carriers")
```

    ## # Source:   table<`carriers`> [?? x 2]
    ## # Database: mysql  [mdsr_public@mdsr.crcbo51tmesf.us-east-2.rds.amazonaws.com:3306/airlines]
    ##    carrier name                                        
    ##    <chr>   <chr>                                       
    ##  1 02Q     Titan Airways                               
    ##  2 04Q     Tradewind Aviation                          
    ##  3 05Q     Comlux Aviation, AG                         
    ##  4 06Q     Master Top Linhas Aereas Ltd.               
    ##  5 07Q     Flair Airlines Ltd.                         
    ##  6 09Q     Swift Air, LLC                              
    ##  7 0BQ     DCA                                         
    ##  8 0CQ     ACM AIR CHARTER GmbH                        
    ##  9 0GQ     Inter Island Airways, d/b/a Inter Island Air
    ## 10 0HQ     Polar Airlines de Mexico d/b/a Nova Air     
    ## # ℹ more rows

``` r
dbGetQuery(con,
          "
          SELECT
            c.name AS airline,
            f.flight AS flight_number
          FROM flights AS f
          JOIN carriers AS c
            ON f.carrier = c.carrier
          WHERE
            year = 2015 AND
            month = 9 AND
            day = 26
            AND (
              (origin = 'LAX' AND dest = 'JFK') OR
              (origin = 'JFK' AND dest = 'LAX')
            )
          ")
```

    ##                   airline flight_number
    ## 1   United Air Lines Inc.           441
    ## 2         JetBlue Airways            24
    ## 3          Virgin America           399
    ## 4    Delta Air Lines Inc.          1908
    ## 5         JetBlue Airways            23
    ## 6  American Airlines Inc.           118
    ## 7    Delta Air Lines Inc.           476
    ## 8          Virgin America           404
    ## 9  American Airlines Inc.            34
    ## 10 American Airlines Inc.            33
    ## 11  United Air Lines Inc.           275
    ## 12   Delta Air Lines Inc.           472
    ## 13  United Air Lines Inc.          1985
    ## 14 American Airlines Inc.             2
    ## 15        JetBlue Airways           123
    ## 16        JetBlue Airways           124
    ## 17   Delta Air Lines Inc.           412
    ## 18         Virgin America           407
    ## 19 American Airlines Inc.           255
    ## 20         Virgin America           406
    ## 21  United Air Lines Inc.           779
    ## 22        JetBlue Airways           223
    ## 23        JetBlue Airways           224
    ## 24  United Air Lines Inc.           703
    ## 25 American Airlines Inc.             4
    ## 26   Delta Air Lines Inc.           423
    ## 27 American Airlines Inc.             3
    ## 28 American Airlines Inc.            19
    ## 29         Virgin America           411
    ## 30        JetBlue Airways           323
    ## 31        JetBlue Airways           324
    ## 32   Delta Air Lines Inc.           920
    ## 33         Virgin America           412
    ## 34   Delta Air Lines Inc.           464
    ## 35 American Airlines Inc.            32
    ## 36  United Air Lines Inc.          1752
    ## 37  United Air Lines Inc.           841
    ## 38 American Airlines Inc.           117
    ## 39        JetBlue Airways           424
    ## 40   Delta Air Lines Inc.           477
    ## 41         Virgin America           416
    ## 42 American Airlines Inc.            22
    ## 43 American Airlines Inc.           180
    ## 44        JetBlue Airways           423
    ## 45         Virgin America           413
    ## 46   Delta Air Lines Inc.           447
    ## 47 American Airlines Inc.            21
    ## 48   Delta Air Lines Inc.           420
    ## 49        JetBlue Airways           523
    ## 50  United Air Lines Inc.           535
    ## 51 American Airlines Inc.           293
    ## 52         Virgin America           415
    ## 53        JetBlue Airways           623
    ## 54        JetBlue Airways           524
    ## 55   Delta Air Lines Inc.          1162
    ## 56  United Air Lines Inc.           912
    ## 57   Delta Air Lines Inc.          1262
    ## 58         Virgin America           420
    ## 59 American Airlines Inc.            30
    ## 60        JetBlue Airways           624
